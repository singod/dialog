<template>
  <div :aria-hidden="`${ariaHidden}`">
    <div
      class="c-dialog-overlay"
      :style="{backgroundColor: overlayBackground}"
      @click.stop="close"
      v-if="isOpen"
      transition="wv-dialog-overlay"
      tabindex="-1"
    ></div>
    <div
      class="c-dialog"
      @click.stop="close"
      v-show="isOpen"
      transition="wv-dialog-overlay"
      :aria-labelledby="ariaLabelledby"
      :aria-describedby="ariaDescribedby"
      role="dialog"
    >
      <div
        class="c-dialog__content"
        :class="[class]"
        :style="{maxWidth: dialogMaxWidth}"
        v-el:dialog-content
        v-show="isOpen"
        @animationend="onTransitionEnd"
        transition="wv-dialog"
        role="document"
      >
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
  import {eventBus} from './helpers'

  export default {
    props: [
      {
        name: 'overlayBackground',
        type: String,
        default: 'rgba(0, 0, 0, .5)'
      }, {
        name: 'class',
        type: String,
        default: false
      }, {
        name: 'width',
        type: [Number, String],
        default: false
      }, {
        name: 'ref',
        type: String,
        required: true
      }, {
        name: 'closeOnEsc',
        type: Boolean,
        default: true
      }, {
        name: 'closeOnOutsideClick',
        type: Boolean,
        default: true
      }, {
        name: 'ariaLabelledby',
        type: String,
        default: false
      }, {
        name: 'ariaDescribedby',
        type: String,
        default: false
      }
    ],
    data () {
      return {
        isOpen: false,
        ariaHidden: true
      }
    },
    computed: {
      dialogMaxWidth () {
        if (!this.width) {
          return false
        }

        if (typeof this.width === 'number') {
          return `${this.width}px`
        }

        if (typeof this.width === 'string') {
          return this.width
        }
      }
    },
    ready () {
      eventBus.on('toggle:dialog', this.onToggle)
      document.addEventListener('keydown', this.onKeyDown)
    },
    beforeDestroy () {
      eventBus.removeListener('toggle:dialog', this.onToggle)
      document.removeEventListener('keydown', this.onKeyDown)
    },
    methods: {
      onKeyDown (event) {
        // check if keydown is 'esc'
        if (this.isOpen && event.which === 27 && this.closeOnEsc) {
          event.stopPropagation()
          this.close()
        }
      },
      onToggle (dialogRef, actionType) {
        if (this.ref !== dialogRef) return

        switch (actionType) {
          case 'toggle': {
            if (this.isOpen) {
              this.close()
            } else {
              this.open()
            }
            break
          }
          case 'close': {
            this.close()
            break
          }
          case 'open': {
            this.open()
            break
          }
          default: {}
        }
      },
      close (event) {
        if (event) {
          if (!this.closeOnOutsideClick) return

          // check if click inside
          if (event.target === this.$els.dialogContent || this.$els.dialogContent.contains(event.target)) {
            return
          }

          // return if event.target doesn't exist
          if (!window.document.contains(event.target)) return
        }

        this.isOpen = false
      },
      open () {
        this.isOpen = true
        this.ariaHidden = false
        this.$emit('opened')
        // TODO: añadir aria-hidden true al rootEl
        // TODO: añadir overflow hidden al html
      },
      onTransitionEnd () {
        if (!this.isOpen) {
          this.ariaHidden = true
          this.$emit('closed')
          // TODO: añadir aria-hidden false al rootEl
          // TODO: remover overflow hidden al html
        }
      }
    }
  }
</script>
