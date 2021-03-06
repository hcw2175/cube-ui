<template>
  <div class="cube-infinity-scroll">
    <div class="template" v-once>
      <slot name="render"></slot>
      <div ref="tombstone" class="tombstone">
        <slot name="tombstone">
          <div class="tombstone_default"></div>
        </slot>
      </div>
    </div>
    <div class="scroll-wrapper" ref="wrapper">
      <div class="scroll-list-content">
      </div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import scroll from '../../common/mixins/scroll'

  const COMPONENT_NAME = 'cube-infinity-scroll'

  const EVENT_FETCH = 'fetch'

  const DEFAULT_OPTIONS = {
    observeDOM: false,
    click: true,
    scrollY: true
  }

  export default {
    name: COMPONENT_NAME,
    mixins: [scroll],
    props: {
      render: {
        type: Function,
        required: true
      }
    },
    mounted () {
      this._createInfinityScroll()
    },
    activated() {
      /* istanbul ignore next */
      this.enable()
    },
    deactivated() {
      /* istanbul ignore next */
      this.disable()
    },
    beforeDestroy() {
      this.destroy()
    },
    data () {
      return {
        items: []
      }
    },
    methods: {
      _createInfinityScroll () {
        let options = Object.assign({}, DEFAULT_OPTIONS, this.options, {
          infinity: {
            render: this.render,
            createTombstone: () => {
              return this.$refs.tombstone.cloneNode(true)
            },
            fetch: (amount) => {
              return new Promise((resolve, reject) => {
                const unwatch = this.$watch('items', (newItems) => {
                  resolve(newItems)
                  unwatch()
                })
                this.$emit(EVENT_FETCH, amount)
              })
            }
          }
        })
        this.infinityScroll = new BScroll(this.$refs.wrapper, options)
      },
      setItems (items) {
        this.items = items
      },
      disable () {
        this.infinityScroll && this.infinityScroll.disable()
      },
      enable () {
        this.infinityScroll && this.infinityScroll.enable()
      },
      destroy () {
        this.infinityScroll && this.infinityScroll.destroy()
        this.infinityScroll = null
      }
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @require "../../common/stylus/variable.styl"

  .cube-infinity-scroll
    position: relative
    height: 100%
    overflow: hidden
    .template
      display: none
    .tombstone
      width: 100%
      padding: 10px 0
    .tombstone_default
      background-color: $infinity-scroll-tombstone-bgc
      width: 90%
      margin: 0 auto
      height: 20px
    .scroll-wrapper
      position: relative
      height: 100%
      overflow: hidden
      .scroll-list-content
        position: relative
        z-index: 1
</style>
