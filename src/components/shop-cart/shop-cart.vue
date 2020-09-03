<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">
              <bubble :num="totalCount"></bubble>
            </div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
          <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right">
          <div class="pay" :class="payClass">{{payDesc}}</div>
        </div>
      </div>

      <div class="ball-container">
        <div v-for="(ball,index) in balls" :key="index">
          <transition @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Bubble from "components/bubble/bubble";

const BALL_LEN = 10;

function createBalls() {
  let ret = [];
  for (let i = 0; i < BALL_LEN; i++) {
    ret.push({ show: false });
  }
  return ret;
}
export default {
  name: "shop-cart",
  props: {
    selectFoods: {
      type: Array,
      default() {
        return [];
      },
    },
    deliveryPrice: {
      type: Number,
      default: 0,
    },
    minPrice: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      balls: createBalls(),
    };
  },
  computed: {
    totalPrice() {
      let total = 0;
      this.selectFoods.forEach((food) => {
        total += food.price * food.count;
      });
      return total;
    },
    totalCount() {
      let count = 0;
      this.selectFoods.forEach((food) => {
        count += food.count;
      });
      return count;
    },
    payDesc() {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`;
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice;
        return `还差￥${diff}元起送`;
      } else {
        return "去结算";
      }
    },
    payClass() {
      if (!this.totalCount || this.totalPrice < this.minPrice) {
        return "not-enough";
      } else {
        return "enough";
      }
    },
  },
  created() {
    // 下落小球
    this.dropBalls = [];
    this.listFold = true;
  },
  methods: {
    drop(el) {
      // console.log(this.balls)
      for (let i = 0; i < this.balls.length; i++) {
        const ball = this.balls[i];
        // console.log(ball, i)
        if (!ball.show) {
          //如果上一个动画还没有完，就取下一个
          ball.show = true;
          ball.el = el;
          // console.log(ball.show)
          this.dropBalls.push(ball);
          // console.log(this.dropBalls)
          return;
        }
      }
    },
    beforeDrop(el) {
      //动画的钩子
      const ball = this.dropBalls[this.dropBalls.length - 1];

      // 获取添加按钮的位置
      const rect = ball.el.getBoundingClientRect(); //方法返回元素的大小及其相对于视口的位置。
      // console.log(rect)
      const x = rect.left - 32; //购物车位置与添加之间的x 轴距离
      const y = -(window.innerHeight - rect.top - 22); //购物车位置与添加之间的y 轴距离 因为是向上 所以是负值
      el.style.display = "";
      // 开始动画前把小球放到初始位置
      el.style.transform = el.style.webkitTransform = `translate3d(0,${y}px,0)`; //兼容安卓浏览器webkitTransform
      const inner = el.getElementsByClassName("inner-hook")[0];
      inner.style.transform = inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
    },
    dropping(el, done) {
      // 计算body高度出发重绘 赋值给不用的值就行  在将一个元素移动到一个位置在进行动画是需要触发重绘的
      this._reflow = document.body.offsetHeight;
      el.style.transform = el.style.webkitTransform = `translate3d(0,0,0)`;
      const inner = el.getElementsByClassName("inner-hook")[0];
      inner.style.transform = inner.style.webkitTransform = `translate3d(0,0,0)`;
      // 监听transitionend事件 告诉完成了才能执行afterDrop
      el.addEventListener("transitionend", done);
    },
    afterDrop(el) {
      const ball = this.dropBalls.shift();
      if (ball) {
        ball.show = false;
        el.style.display = "none";
      }
    },
    toggleList() {
      if (this.listFold) {
        if (!this.totalCount) {
          return;
        }
        this.listFold = false;
        this._showShopCartList();
        // this._showShopCartSticky();
      } else {
        this.listFold = true;
        this._hideShopCartList();
      }
    },
   
    _showShopCartList() {
      this.shopCartListComp =
        this.shopCartListComp ||
        this.$createShopCartList({
          $props: {
            selectFoods: "selectFoods",
          },
          $events:{
            hide:()=>{
              this.listFold = true
            }
          }

        });
      this.shopCartListComp.show();
    },
    _hideShopCartList() {
      this.shopCartListComp.hide();
    },
  },
  components: {
    Bubble,
  },
};
</script>

<style lang="stylus" scoped>
@import '~common/stylus/mixin';
@import '~common/stylus/variable';

.shopcart {
  height: 100%;

  .content {
    display: flex;
    background: $color-background;
    font-size: 0;
    color: $color-light-grey;

    .content-left {
      flex: 1;

      .logo-wrapper {
        display: inline-block;
        vertical-align: top;
        position: relative;
        top: -10px;
        margin: 0 12px;
        padding: 6px;
        width: 56px;
        height: 56px;
        box-sizing: border-box;
        border-radius: 50%;
        background: $color-background;

        .logo {
          width: 100%;
          height: 100%;
          border-radius: 50%;
          text-align: center;
          background: $color-dark-grey;

          &.highlight {
            background: $color-blue;
          }

          .icon-shopping_cart {
            line-height: 44px;
            font-size: $fontsize-large-xxx;
            color: $color-light-grey;

            &.highlight {
              color: $color-white;
            }
          }
        }

        .num {
          position: absolute;
          top: 0;
          right: 0;
        }
      }

      .price {
        display: inline-block;
        vertical-align: top;
        margin-top: 12px;
        line-height: 24px;
        padding-right: 12px;
        box-sizing: border-box;
        border-right: 1px solid rgba(255, 255, 255, 0.1);
        font-weight: 700;
        font-size: $fontsize-large;

        &.highlight {
          color: $color-white;
        }
      }

      .desc {
        display: inline-block;
        vertical-align: top;
        margin: 12px 0 0 12px;
        line-height: 24px;
        font-size: $fontsize-small-s;
      }
    }

    .content-right {
      flex: 0 0 105px;
      width: 105px;

      .pay {
        height: 48px;
        line-height: 48px;
        text-align: center;
        font-weight: 700;
        font-size: $fontsize-small;

        &.not-enough {
          background: $color-dark-grey;
        }

        &.enough {
          background: $color-green;
          color: $color-white;
        }
      }
    }
  }

  .ball-container {
    .ball {
      position: fixed;
      left: 32px;
      bottom: 22px;
      z-index: 200;
      transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41);

      .inner {
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background: $color-blue;
        transition: all 0.4s linear;
      }
    }
  }
}
</style>
