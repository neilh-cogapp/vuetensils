
<script>
import { safeSlot } from "../../utils"
/**
 * A renderless component for awaiting promises to resolve; great for making HTTP requests. Supports showing pending, resolved, or rejected promises.
 */
export default {
  name: "VAsync",
  props: {
    /**
     * A promise or function that returns a promise.
     */
    await: {
      type: [Promise, Function],
      default: () => Promise.resolve(),
    },
    /**
     * The default value to provide for the `results`. Useful if the promise resolve value is undefined.
     */
    default: {
      type: undefined,
      default: undefined,
    },
  },

  data() {
    return {
      pending: false,
      results: this.default,
      error: null,
      done: false,
    }
  },

  watch: {
    await: {
      handler: "awaitOn",
      immediate: true,
    },

    pending: {
      handler(pending) {
        /**
         * Fired whenever the pending status changes.
         * @event pending
         * @type { boolean }
         */
        this.$emit("pending", pending)
      },
      immediate: true,
    },
  },

  methods: {
    awaitOn(promise) {
      if (!promise) return

      promise = typeof promise === "function" ? promise() : promise

      if (!promise.then) return

      this.pending = true
      this.results = this.default
      this.error = null

      return promise
        .then(results => {
          this.results = typeof results === "undefined" ? this.default : results
          /**
           * Fired after promise has resolved with the resolved value.
           * @event resolve
           * @type { unknown }
           */
          this.$emit("resolve", results)
        })
        .catch(error => {
          if (error instanceof Error) {
            error = {
              name: error.name,
              message: error.message,
            }
          }
          this.error = error
          /**
           * Fired after promise has rejected with the rejected error.
           * @event reject
           * @type { error }
           */
          this.$emit("reject", error)
        })
        .finally(() => {
          this.pending = false
          this.done = true
          /**
           * Fired after promise has fulfilled, regardless of success or failure.
           * @event finally
           * @type { undefined }
           */
          this.$emit("finally")
        })
    },
  },

  render(h) {
    const { pending, error, results, done } = this

    /** @slot Rendered while the promise is in a pending state */
    const pendingSlot = this.$scopedSlots.pending
    /** @slot Rendered when the promise has rejected. Provides the caught error. */
    const rejectedSlot = this.$scopedSlots.rejected
    /** @slot Rendered when the promise has resolved. Provides the results. */
    const resolvedSlot = this.$scopedSlots.resolved
    /** @slot Provides the status of the component for pending state, error, or results. */
    const defaultSlot = this.$scopedSlots.default

    if (pending && pendingSlot) {
      return safeSlot(h, pendingSlot())
    }

    if (done && error && rejectedSlot) {
      return safeSlot(h, rejectedSlot(error))
    }

    if (done && !error && resolvedSlot) {
      return safeSlot(h, resolvedSlot(results))
    }

    if (!defaultSlot) return

    return safeSlot(
      h,
      defaultSlot({
        pending,
        results,
        error,
      })
    )
  },
}
</script>
