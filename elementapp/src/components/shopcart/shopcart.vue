<template>
<div>
  <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left" :class="{'highlight': totalCount > 0}">
        <div class="logo-wrapper">
          <div class="logo">
            <i class="icon-shopping_cart"></i>
          </div>
          <div class="num" v-show="totalCount > 0">{{ totalCount }}</div>
        </div>
        <div class="price">￥{{ totalPrice }}</div>
        <div class="desc">另需配送费￥{{ deliveryPrice }}元</div>
      </div>
      <div class="content-right" @click.stop.prevent="pay">
        <div class="pay" :class="payClass">{{ payDesc }}</div>
      </div>
    </div>
    <template v-for='(ball,index) in balls'>
      <transition name='drop' :key='index' @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
        <div class="drop-wrapper" v-show='ball.show'>
          <div class="inner"></div>
        </div>
      </transition>
    </template>
    <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
            <div class="list-header">
                <h1 class="title">购物车</h1>
                <span class="empty" @click="clearTotal">清空</span>
            </div>
            <div class="list-content" ref="listContent">
                <ul>
                    <li class="food border-1px" v-for="(food,index) in selectFoods" :key='index'>
                        <span class="name">{{ food.name }}</span>
                        <div class="price">￥{{ food.price * food.count }}</div>
                        <div class="cartcontrol-wrapper">
                            <cartControl :food="food" v-on:cartAdd='drop'></cartControl>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </transition>
    <transition name="fade">
     <div class="list-mask" v-show="listShow" @click="hideList"></div>
    </transition>
  </div>
</div>
</template>

<script>
import cartControl from '@/components/cartControl/cartControl'
import BScroll from 'better-scroll'
export default {
  data () {
    return {
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      dropBalls: [],
      fold: true
    }
  },
  computed: {
    totalPrice () {
      let total = 0
      this.selectFoods.forEach(foods => {
        total += foods.price * foods.count
      })
      return total
    },
    totalCount () {
      let count = 0
      this.selectFoods.forEach(foods => {
        count += foods.count
      })
      return count
    },
    payDesc () {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        return `还差￥${diff}起送`
      } else {
        return '去支付'
      }
    },
    listShow () {
      if (!this.totalCount) {
        // eslint-disable-next-line
        this.fold = true
        return false
      }
      let show = !this.fold
      if (show) {
        this.$nextTick(() => {
          if (!this.listScroll) {
            // eslint-disable-next-line
            this.listScroll = new BScroll(this.$refs.listContent, {
              click: true
            })
          } else {
            this.listScroll.refresh()
          }
        })
      }
      return show
    },
    payClass () {
      if (this.totalPrice < this.minPrice) {
        return 'not-enough'
      } else {
        return 'enough'
      }
    }
  },
  methods: {
    clearTotal () {
      this.selectFoods.forEach(food => {
        food.count = 0
      })
    },
    toggleList () {
      this.$nextTick(() => {
        this.fold = !this.fold
      })
    },
    drop (el) {
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          this.dropBalls.push(ball)
          return
        }
      }
    },
    beforeEnter (el) {
      let count = this.balls.length
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect()
          let x = rect.left - 20
          let y = -(window.innerHeight - rect.top - 37)
          el.style.display = ''
          el.style.webkitTransform = `translate3d(0,${y}px,0)`
          el.style.transform = `translate3d(0,${y}px,0)`

          let inner = el.getElementsByClassName('inner')[0]
          inner.style.webkitTransform = `translate3d(${x}px,0,0)`
          inner.style.transform = `translate3d(${x}px,0,0)`
        }
      }
    },
    enter (el) {
      // eslint-disable-next-line
      let rf = el.offsetHeight
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0,0,0)'
        el.style.transform = 'translate3d(0,0,0)'
        let inner = el.getElementsByClassName('inner')[0]
        inner.style.webkitTransform = 'translate3d(0,0,0)'
        inner.style.transform = 'translate3d(0,0,0)'
      })
    },
    afterEnter (el) {
      let ball = this.dropBalls.shift()
      if (ball) {
        ball.show = false
        el.style.display = 'none'
      }
    },
    hideList () {
      this.fold = true
    },
    pay () {
      if (this.totalPrice < this.minPrice) {
        return
      }
      window.alert(`支付${this.totalPrice}元`)
    }
  },
  props: {
    minPrice: {
      type: Number,
      default: 0
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    selectFoods: {
      type: Array,
      default () {
        return []
      }
    }
  },
  components: {
    cartControl
  }
}
</script>

<style scoped lang='less'>
@import "../../common/less/mixins";
.shopcart{
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: 50;
  width: 100%;
  height: 48px;
  .content {
    display: flex;
    background: #141d27;
    font-size: 0;
    color: rgba(255, 255, 255, .4);
    .content-left {
      flex: 1;
      .logo-wrapper {
        display: inline-block;
        vertical-align: top;
        position: relative;
        top: -10px;
        margin: 0 18px;
        padding: 6px;
        width: 56px;
        height: 56px;
        box-sizing: border-box;
        border-radius: 50%;
        background: #141d27;
        .logo {
          width: 100%;
          height: 100%;
          border-radius: 50%;
          background: #2b343c;
          text-align: center;
          .icon-shopping_cart {
            font-size: 24px;
            line-height: 44px;
            color: #80858a;
          }
        }
        .num {
          position: absolute;
          top: 0;
          right: 0;
          width: 24px;
          height: 16px;
          line-height: 16px;
          text-align: center;
          border-radius: 16px;
          font-size: 9px;
          font-weight: 700;
          color: #fff;
          background: rgb(240, 20, 20);
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .4);
        }
      }
      .price {
        vertical-align: top;
        display: inline-block;
        line-height: 24px;
        margin-top: 12px;
        padding-right: 12px;
        box-sizing: border-box;
        border-right: 1px solid rgba(255, 255, 255, .1);
        font-size: 16px;
        font-weight: 700;
      }
      .desc {
        display: inline-block;
        vertical-align: top;
        line-height: 24px;
        margin: 12px 0 0 12px;
        font-size: 10px;
      }
      &.highlight {
        .logo-wrapper {
          .logo {
            background: rgb(0, 160, 220);
            .icon-shopping_cart {
              color: #fff;
            }
          }
        }
        .price {
          color: #fff;
        }
      }
    }
    .content-right {
      flex: 0 0 105px;
      width: 105px;
      .pay {
        height: 48px;
        line-height: 48px;
        text-align: center;
        font-size: 12px;
        font-weight: 700;
        &.not-enough {
          background: #2b333b;
        }
        &.enough {
          background: #00b43c;
          color: #fff;
        }
      }
    }
  }
  .drop-wrapper{
    position: fixed;
    left: 1.866667rem;
    bottom: 1.333333rem;
    z-index: 100;
    .inner{
      width:0.533333rem;
      height:0.533333rem;
      border-radius: 50%;
      background-color: rgb(0, 160, 220);
    }
    &.drop-enter-active{
      transition: all .4s cubic-bezier(.59,-0.22,.83,.67);
      .inner{
        transition: all .4s linear;
      }
    }
  }
  .shopcart-list {
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
    width: 100%;
    transform: translate3d(0, -100%, 0);
    &.fold-enter-active, &.fold-leave-active {
        transition: all .5s;
    }
    &.fold-enter, &.fold-leave-active {
        transform: translate3d(0, 0, 0);
    }
    .list-header {
        height: 40px;
        line-height: 40px;
        padding: 0 18px;
        background: #f3f5f7;
        border-bottom: 1px solid rgba(7, 17, 27, .1);
        .title {
            float: left;
            font-size: 14px;
            color: rgb(7, 17, 27);
        }
        .empty {
            float: right;
            font-size: 12px;
            color: rgb(0, 160, 220);
        }
    }
    .list-content {
        padding: 0 18px;
        max-height: 217px;
        background: #fff;
        overflow: hidden;
        .food {
            position: relative;
            padding: 12px 0;
            box-sizing: border-box;
            .border-bottom-1px(rgba(7, 17, 27, .1));
            .name {
                line-height: 24px;
                font-size: 14px;
                color: rgb(7, 17, 27);
            }
            .price {
                position: absolute;
                right: 90px;
                bottom: 12px;
                line-height: 24px;
                font-weight: 700;
                font-size: 14px;
                color: rgb(240, 20, 20);
            }
            .cartcontrol-wrapper {
                position: absolute;
                bottom: 6px;
                right: 0;
            }
        }
    }
  }
  .list-mask {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -2;
    background: rgba(7, 17, 27, .6);
    backdrop-filter: blur(10px);
    &.fade-enter-active{
      opacity:1;
      transition: all .5s;
    }
    &.fade-leave-active{
      opacity: 0;
      transition: all .5s;
    }
    &.fade-enter{
      opacity: 0
    }
    &.fade-leave {
      opacity: 1
    }
  }
}

</style>
