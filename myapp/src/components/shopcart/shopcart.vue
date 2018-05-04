<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo"
             :class="{'highlight': totalCount>0}">
              <i class="icon-shopping_cart"
                :class="{'highlight': totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">{{ totalCount }}</div>
          </div>
          <div class="price"
            :class="{'highlight': totalPrice>0}">￥{{ totalPrice }}</div>
          <div class="desc">另需配送费￥{{ deliveryPrice }}元</div>
        </div>
        <!--=================-->
        <div class="content-right" @click.stop.prevent="pay">
          <div class="pay" :class="payClass">
            {{ payDesc }}
          </div>
        </div>
      </div>
      <!--=================-->
      <div class="ball-container">
        <div v-for="(ball,index) in balls"
          :key="index">
          <transition name="drop"
            @before-enter="beforeDrop"
            @enter="dropping"
            @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <!--====================-->
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <!--============-->
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food"
                v-for="(food,index) in selectFoods"
                :key="index">
                <span>{{ food.name }}</span>
                <div class="price">
                  <span>￥{{ food.price * food.count }}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food" @add="addFood"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
          <!--===================-->
        </div>
      </transition>
      <!--====================-->
      <!--以下是背景的HTML：当购物车浮层显示的时候，这个背景也显示-->
      <transition name="fade">
        <div class="list-mask"
         v-show="listShow"
         @click="hideList"></div>
      </transition>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import Vue from 'vue'
import cartcontrol from 'components/cartcontrol/cartcontrol'
import BScorll from 'better-scroll'
export default {
  props: {
    selectFoods: {
      type: Array,
      default () {
        return [
          /* {
            price: 10,
            count: 1
          } */
        ]
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
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
      // 默认是折叠的
      fold: true,
      scroll: false
    }
  },
  computed: {
    payClass () {
      if (this.totalPrice < this.minPrice) {
        return 'not-enough'
      } else {
        return 'enough'
      }
    },
    totalCount () {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    totalPrice () {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    payDesc () {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        return `还差￥${diff}元起送`
      } else {
        return '去结算'
      }
    },
    listShow () {
      if (!this.totalCount) {
        /**
         * 计算属性的getter函数是没有副作用 (side effect) 的，但是setter函数
         * 是有副作用的。
         */
        // this.fold = true // 错误的写法
        Vue.set(this, 'fold', true)
        // this.$set(this, 'fold', true)
        return false
      }
      let show = !this.fold
      if (show) {
        this.$nextTick(() => {
          // better-scroll的初始化
          if (!this.scroll) {
            let betterScroll = new BScorll(this.$refs.listContent, {
              click: true
            })
            if (betterScroll) {
              Vue.set(this, 'scroll', true)
            } else {
              Vue.set(this, 'scroll', false)
            }
          }
        })
      }
      return show
    }
  },
  methods: {
    drop (el) {
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          // console.log(ball.el)
          this.dropBalls.push(ball)
          return
        }
      }
    },
    // 小球抛出之前的定位：把小球放到【+号】处
    beforeDrop (el) {
      // console.log(el)
      let count = this.balls.length
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect()
          let x = rect.left - 22
          let y = -(window.innerHeight - rect.top - 22)
          el.style.display = ''
          el.style.webkitTransform = `translate3d(0, ${y}px, 0)`
          el.style.transform = `translate3d(0, ${y}px, 0)`
          let inner = el.getElementsByClassName('inner-hook')[0]
          inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`
          inner.style.transform = `translate3d(${x}px, 0, 0)`
        }
      }
    },
    // 抛出中
    dropping (el, done) {
      /* eslint-disable no-unused-vars */
      let rf = el.offsetHeight
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0,0,0)'
        el.style.transform = 'translate3d(0,0,0)'
        let inner = el.getElementsByClassName('inner-hook')[0]
        inner.style.webkitTransform = 'translate3d(0,0,0)'
        inner.style.transform = 'translate3d(0,0,0)'
        el.addEventListener('transitionend', done)
      })
    },
    // 抛球完成
    afterDrop (el) {
      let ball = this.dropBalls.shift()
      if (ball) {
        ball.show = false
        el.style.display = 'none'
      }
    },
    // 点击展开购物车详情页
    toggleList () {
      if (!this.totalCount) {
        return
      }
      this.fold = !this.fold
    },
    // 添加食物的时候抛小球
    addFood (target) {
      this.drop(target)
    },
    // 清空
    empty () {
      this.selectFoods.forEach((food) => {
        food.count = 0
      })
    },
    // 点击背景的位置可以隐藏浮层
    hideList () {
      this.fold = true
    },
    pay () {
      if (this.totalPrice < this.minPrice) {
        return
      }
      window.alert(`已经从你的账号中扣除了${this.totalPrice}元`)
    }
  },
  components: {
    cartcontrol
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin.styl"
.shopcart
  position: fixed
  left: 0
  bottom: 0
  width: 100%
  height: 48px
  z-index: 100
  .content
    display: flex
    background: #141d27
    color: rgba(255,255,255,0.4)
    .content-left
      flex: 1
      .logo-wrapper
        display: inline-block
        vertical-align: top
        position: relative
        padding: 6px
        top: -10px
        margin: 0 12px
        border-radius: 50%
        /*background: #141d27*/
        background: blue
        height: 56px
        width: 56px
        margin: 0 16px
        box-sizing: border-box
        .logo
          width: 100%
          height: 100%
          border-radius: 50%
          text-align: center
          background: #2b343c
          &.highlight
            background: rgb(0,160,220)
          .icon-shopping_cart
            line-height: 44px
            font-size: 24px
            color: #80858a
            &.highlight
              color: #fff
        .num
          position: absolute
          top: 0
          right: 0
          width: 24px
          height: 16px
          line-height: 16px
          font-size: 9px
          color: #fff
          font-weight: 700
          text-align: center
          border-radius: 6px
          background: rgb(240,20,20)
          box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4)
      .price
        display: inline-block
        line-height: 24px
        font-size: 16px
        font-weight: 700
        padding-right: 12px
        /*background: red*/
        vertical-align: top
        margin-top: 12px
        box-sizing: border-box
        border-right: 1px solid rgba(255,255,255,0.1)
        &.highlight
          color: #fff
      .desc
        display: inline-block
        vertical-align: top
        margin: 12px 0 0 12px
        line-height: 24px
        font-size: 10px
    .content-right
      flex: 0 0 105px
      width: 105px
      .pay
        height: 48px
        line-height: 48px
        text-align: center
        font-size: 14px
        font-weight: 700
        &.not-enough
          background: #2b333b
        &.enough
          background: #00b43c
          color: #fff
  .ball-container
    .ball
      position: fixed
      left: 32px
      bottom: 22px
      z-index: 200
      transition: all 0.4s cubic-bezier(.49,-0.29,.75,.41)
      .inner
        width: 16px
        height: 16px
        border-radius: 50%
        background: rgb(0,160,220)
        transition: all 0.4s linear
  .shopcart-list
    position: absolute
    left: 0
    top: 0
    z-index: -1
    width: 100%
    transform: translate3d(0,-100%,0)
    &.fold-enter-active,&.fold-leave-active
      transition: all 0.5s
    &.fold-enter,&.fold-leave-active
      transform: translate3d(0,0,0)
    .list-header
      height: 40px
      line-height: 40px
      padding: 0 18px
      background: #f3f5f7
      border-bottom: 1px solid rgba(7,17,27,0.1)
      .title
        float: left
        font-size: 14px
        color: rgb(7,17,27)
      .empty
        float: right
        font-size: 12px
        color: rgb(0,160,220)
    .list-content
      padding: 0 18px
      max-height: 258px
      overflow: hidden
      background: #fff
      .food
        position: relative
        padding: 12px 0
        box-sizing: border-box
        border-1px(rgba(7,17,27,0.1))
        .name
          line-height: 24px
          font-size: 14px
          color: rgb(7,17,27)
        .price
          position: absolute
          right: 90px
          bottom: 12px
          line-height: 24px
          font-size: 14px
          font-weight: 700
          color: rgb(240,20,20)
        .cartcontrol-wrapper
          position: absolute
          right: 0
          bottom: 6px
  .list-mask
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    z-index: -2
    opacity: 1
    backdrop-filter: blur(10px)
    background: rgba(7,17,27,0.6)
    &.fade-enter-active,&.fade-leave-active
      transition: all 0.5s
    &.fade-enter,&.fade-leave-active
      opacity: 0
      background: rgba(7,17,27,0)
</style>
