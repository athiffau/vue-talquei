
<template>
  <section class="talquei">
    <div
      ref="terminal"
      v-scroll-down
      class="talquei-terminal"
    >
      <slot />
    </div>

    <div
      v-if="inputNode"
      class="talquei-inputbar"
    >
      <VNodes :vnodes="inputNode" />
    </div>
  </section>
</template>

<script>
import TalqueiMessage from './TalqueiMessage.vue'
import VNodes from '../utils/v-nodes'
import scrollDown, { scrollToBottom } from '../utils/scroll-down'

export default {
  name: 'Talquei',

  directives: {
    'scroll-down': scrollDown,
  },

  provide () {
    return {
      next: this.next,
      showInput: this.showInput,
    }
  },

  components: {
    VNodes,
  },

  props: {
    autoRun: {
      type: Boolean,
      required: false,
      default: true,
    },
  },

  data: () => ({
    step: null,
    inputNode: null,
    messages: [],
  }),

  computed: {
    totalSteps () {
      return this.messages.length
    },
  },

  watch: {
    step (val) {
      this.inputNode = null
      this.runMessage(this.messages[val])
    },
  },

  mounted () {
    if (this.autoRun) {
      this.init()
    }
  },

  updated () {
    scrollToBottom(this.$refs.terminal)
  },

  methods: {
    init () {
      this.collectMessages()
      this.step = 0
    },

    next () {
      this.inputNode = null
      this.collectMessages()
      const nextStep = this.step + 1
      if (nextStep < this.totalSteps) {
        this.step = nextStep
      } else {
        this.emitComplete()
      }
    },

    emitComplete () {
      this.$emit('complete')
    },

    showInput (vnode) {
      this.inputNode = vnode
    },

    collectMessages () {
      this.messages = this.$children.filter(node => node.$options.name === TalqueiMessage.name)
    },

    runMessage (message) {
      if (!message) return

      message.run()
    },
  },
}
</script>
