<template>
  <div class="tab">
    <cube-tab-bar
      :showSlider=true
      :useTransition=false 
      v-model="selectedLabel"
      :data="tabs"
      ref="tabBar"
      class="border-bottom-1px"
    >
    </cube-tab-bar>

    <div class="slide-wrapper">
      <cube-slide
        :loop=false
        :auto-play=false
        :show-dots=false
        :initial-index="index"
        ref="slide"
        @change="onChange"
        @scroll="onScroll"
        :options="slideOptions"
      >
        <cube-slide-item v-for = "(item,index) of tabs" :key = "index">
            <!-- component :is 指定具体组件 组件是app.vue传递进来 -->
            <component  :is="item.comment" :data="item.data" ref="component"></component>
        </cube-slide-item>
      </cube-slide>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'tab',
    props: {
      tabs: {
        type: Array,
        default() {
          return []
        }
      }
    },
    data() {
      return {
        index: 0,
        slideOptions: { // scorll配置
          listenScroll: true,
          probeType: 3,
          directionLockThreshold: 0
        }
      }
    },
    mounted () {
      this.onChange(this.index) // 首次加载自动调用
      // console.log(this.tabs)
    },
    methods: {
      onChange(current) {
        this.index = current
        const component = this.$refs.component[current]
        component.fetch && component.fetch() // 如果component下定义了fetch方法才去调用
      },
      onScroll(pos) {
        // console.log(pos.x)  //slide滚动的位置
        // console.log( this.$refs.slide) 
        const tabBarWidth = this.$refs.tabBar.$el.clientWidth //组件的dom 是$el  获取元素宽度需要使用.$el.clientWidth
        const slideWidth = this.$refs.slide.slide.scrollerWidth // slide内部有一个slide对象，slide实际上是batterscroll的实例，scrollerWidth是整个slide的宽度 
        const transfrom = -pos.x / slideWidth * tabBarWidth // ops.x为负值 / 整个slide的宽度算出来的比例 * tabbar的宽度，就是tabbar要移动的距离
        this.$refs.tabBar.setSliderTransform(transfrom) // 改变位置api
      }
    },
    computed: {
      selectedLabel: {
        get() {
          return this.tabs[this.index].label // slide滑动的时候改变index ,tab索引依赖index 所以会重新计算
        },
        // 获取tab索引给slide
        set(newVal) {
          this.index = this.tabs.findIndex((value) => {
            return value.label === newVal
          })
        }
      }
    }
  }
</script>

<style lang="stylus" scoped>
  @import "~common/stylus/variable"

  .tab
    display: flex
    flex-direction: column
    height: 100%
    >>> .cube-tab
      padding: 10px 0
    .slide-wrapper
      flex: 1
      overflow: hidden
</style>
