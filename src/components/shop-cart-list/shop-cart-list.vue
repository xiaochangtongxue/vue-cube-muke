<template>
  <transition name="fade" appear>
    <!-- //mask-closable=true 点击蒙层的时候会派发一个事件   @mask-click="maskClick" -->
    <!-- type="shop-cart-list"  => 会添加一个 shop-cart-list的类 -->
    <cube-popup
      :mask-closable="true"
      v-show="visible"
      @mask-click="maskClick"
      position="bottom"
      type="shop-cart-list"
      :z-index="90"
    >
      <transition name="move" appear @after-leave="afterLeave">
        <div v-if="visible">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <cube-scroll class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="(food,index) in selectFoods" :key="index">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price*food.count}}</span>
                </div>
                <div class="cart-control-wrapper">
                  <cart-control @add="onAdd" :food="food"></cart-control>
                </div>
              </li>
            </ul>
          </cube-scroll> 
        </div>
      </transition>
    </cube-popup>
  </transition>
</template>

<script>
import CartControl from "components/cart-control/cart-control";
//   import popupMixin from 'common/mixins/popup'

const EVENT_SHOW = "show";
const EVENT_ADD = "add";
const EVENT_LEAVE = "leave";
const EVENT_HIDE = "hide"
export default {
  name: "shop-cart-list",
  // mixins: [popupMixin],
  data() {
    return {
      visible: false,
    };
  },
  props: {
    selectFoods: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  created() {
    this.$on(EVENT_SHOW, () => {
      this.$nextTick(() => {
        this.$refs.listContent.refresh();
      });
    });
  },
  methods: {
    show() {
      this.visible = true;
      // this.$emit(EVENT_SHOW)
    },
    hide() {
      this.visible = false;
      this.$emit(EVENT_HIDE)
    },

    onAdd(target) {
      // this.$emit(EVENT_ADD, target);
    },
    afterLeave() {
      // this.$emit(EVENT_LEAVE);
    },
    maskClick() {
      console.log(2222222222)
      this.hide();
    },
    empty() {
      this.dialogComp = this.$createDialog({
        type: "confirm",
        content: "清空购物车？",
        $events: {
          confirm: () => {
            this.selectFoods.forEach((food) => {
              food.count = 0;
            });
            this.hide();
          },
        },
      });
      this.dialogComp.show();
    },
  },
  components: {
    CartControl,
  },
};
</script>

<style lang="stylus" scoped>
@import '~common/stylus/variable';

.cube-shop-cart-list {
  bottom: 48px;

  &.fade-enter, &.fade-leave-active {
    opacity: 0;
  }

  &.fade-enter-active, &.fade-leave-active {
    transition: all 0.3s ease-in-out;
  }

  .move-enter, .move-leave-active {
    transform: translate3d(0, 100%, 0);
  }

  .move-enter-active, .move-leave-active {
    transition: all 0.3s ease-in-out;
  }

  .list-header {
    height: 40px;
    line-height: 40px;
    padding: 0 18px;
    background: $color-background-ssss;

    .title {
      float: left;
      font-size: $fontsize-medium;
      color: $color-dark-grey;
    }

    .empty {
      float: right;
      font-size: $fontsize-small;
      color: $color-blue;
    }
  }

  .list-content {
    padding: 0 18px;
    max-height: 217px;
    overflow: hidden;
    background: $color-white;

    .food {
      position: relative;
      padding: 12px 0;
      box-sizing: border-box;

      .name {
        line-height: 24px;
        font-size: $fontsize-medium;
        color: $color-dark-grey;
      }

      .price {
        position: absolute;
        right: 90px;
        bottom: 12px;
        line-height: 24px;
        font-weight: 700;
        font-size: $fontsize-medium;
        color: $color-red;
      }

      .cart-control-wrapper {
        position: absolute;
        right: 0;
        bottom: 6px;
      }
    }
  }
}
</style>
