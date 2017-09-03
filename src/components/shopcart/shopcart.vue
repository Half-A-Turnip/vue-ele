<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">
            ￥{{totalPrice}}元
          </div>
          <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right">
          <div class="pay" :class="payClass" @click.stop.prevent="pay">{{payDesc}}</div>
        </div>
      </div>
      <div class="ball-content">
        <div v-for="(ball,index) in balls" :key="index">
            <transition v-on:before-enter="beforeDrop" v-on:enter="dropping" v-on:after-enter="afterDrop">
              <div class="ball" v-show="ball.show">
                <div class="inner inner-hook"></div>
              </div>
            </transition>
          </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="name">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="(food,index) in selectFoods" :key="index">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price * food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food" @cartAdd="addFood"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
        <div class="list-mask" v-show="listShow" @click="hideList"></div>
    </transition>
  </div>
</template>
<script type="text/ecmascript-6">
  import cartcontrol from '../cartcontrol/cartcontrol.vue'
  import Bscroll from 'better-scroll'
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
        dropBall: [],
        fold: true
      }
    },
    components: {
      cartcontrol
    },
    props: {
      selectFoods: {
        type: Array,
        default () {
          return []
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
    computed: {
      totalPrice () {
        let total = 0
        this.selectFoods.forEach((food) => {
          total += food.price * food.count
        })
        return total
      },
      totalCount () {
        let count = 0
        this.selectFoods.forEach((food) => {
          count += food.count
        })
        return count
      },
       // 计算结算的state
      payDesc () {
        if (this.totalPrice === 0) {
          return `￥${this.minPrice}起送`
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice
          return `还差￥${diff}元起送`
        } else {
          return '去结算'
        }
      },
      // 计算结算的class
      payClass () {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough'
        } else {
          return 'enough'
        }
      },
      listShow () {
        if (!this.totalCount) {
          this.fold = true
        }
        let show = !this.fold
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new Bscroll(this.$refs.listContent, {
                click: true
              })
            } else {
              this.scroll.refresh()
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
            this.dropBall.push(ball)
            return false
          }
        }
      },
      beforeDrop (el) {
        let count = this.balls.length
        while (count--) {
          let ball = this.balls[count]
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect()
            let x = rect.left - 32
            let y = -(window.innerHeight - rect.top - 22)
            el.style.display = ''
            el.style.webkitTransform = `translate3d(0,${y}px,0)`
            el.style.transform = `translate3d(0,${y}px,0)`
            let inner = el.getElementsByClassName('inner-hook')[0]
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`
            inner.style.transform = `translate3d(${x}px,0,0)`
          }
        }
      },
      dropping (el, done) {
        /* eslint no-unused-vars: "error" */
        let rf = el.offsetHeight
        console.log(rf)
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0,0,0)'
          el.style.transform = 'translate3d(0,0,0)'
          let inner = el.getElementsByClassName('inner-hook')[0]
          inner.style.webkitTransform = 'translate3d(0,0,0)'
          inner.style.transform = 'translate3d(0,0,0)'
          el.addEventListener('transitionend', done)
        })
      },
      afterDrop (el) {
        let ball = this.dropBall.shift()
        if (ball) {
          ball.show = false
          el.style.display = 'none'
        }
      },
      toggleList () {
        if (!this.totalCount) {
          return false
        }
        this.fold = !this.fold
      },
      empty () {
        this.selectFoods.forEach((food) => {
          food.count = 0
        })
      },
      hideList () {
        this.fold = true
      },
      pay () {
        if (this.totalPrice < this.minPrice) {
          return false
        }
        window.alert(`需要支付${this.totalPrice}元`)
      },
      addFood (target) {
        this.drop(target)
      }
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  @import "../../common/stylus/mixin.styl"
  .shopcart
    position :fixed
    left:0
    bottom:0
    z-index :50
    width:100%
    height:48px
    .content
      display :flex
      background :#141d27
      font-size:0
      color:rgba(255,255,255,.4)
      .content-left
        flex:1
        .logo-wrapper
          display :inline-block
          position :relative
          top:-10px
          margin :0 12px
          padding:6px
          width:56px
          height:56px
          box-sizing :border-box
          border-radius :50%
          vertical-align :top
          background :#141d27
          .logo
            width:100%
            height:100%
            border-radius :50%
            text-align :center
            background :#2b343c
            &.highlight
              background :rgb(0,160,220)
            .icon-shopping_cart
              font-size:24px
              color:#80858a
              line-height :44px
              &.highlight
                color:rgb(255,255,255)
          .num
            position :absolute
            top:0 
            right:0
            width:24px
            height:16px
            line-height :16px
            text-align :center
            border-radius :16px
            font-size:9px
            font-weight :400
            color:#fff
            background :rgb(240,20,20)
            box-shadow :0 4px 8px 0 rgba(0,0,0,.4)
        .price
          display :inline-block
          vertical-align :top
          line-height :24px
          margin-top:12px
          box-sizing :border-box
          padding-right :12px
          border-right :1px solid rgba(255,255,255,.1)
          font-size:16px
          font-weight :700
          &.highlight
            color:#fff
        .desc
          display :inline-block
          vertical-align :top
          margin:12px 0 0 12px
          line-height :24px
          font-size:10px
      .content-right
        flex:0 0 105px
        width:105px
        .pay  
          font-size: 12px
          line-height :48px
          text-align :center
          font-weight :700
          &.not-enough
            background :#2b333b
          &.enough
            background :#00b43c
            color:#fff
    .ball-content
      .ball
        position:fixed
        left:32px
        bottom:22px
        z-index:200
        transition :all 0.5s cubic-bezier(0.49,-0.29,0.75,0.41)
        .inner
          height:16px
          width:16px
          border-radius :50%
          background :rgb(0,160,220)
          transition :all 0.5s linear
    .shopcart-list
      position :absolute
      top:0
      z-index :-1
      left :0
      width:100%
      transition :all .5s
      transform :translate3d(0,-100%,0)
      &.fold-enter-active,&.fold-leave-active
        transition :all 0.4s linear
      &.fold-enter,&.fold-leave-active
        transform :translate3d(0,0,0)
      .list-header
        height:40px
        line-height :40px
        padding:0 18px
        background :#f3f5f7
        border-bottom:1px solid rgba(7,17,27,0.1)
        .name
          float: left
          font-size:10px
          color:rgb(7,17,27)
        .empty
          float :right
          font-size:12px
          color:rgb(0,160,120)
      .list-content
        padding:0 18px
        max-height:217px
        background :#fff
        overflow :hidden
        .food
          position :relative
          padding:12px 0
          box-sizing : border-box
          border-1px(rgba(7,17,27,0.1))
          .name
            line-height :24px
            font-size:14px
            color:rgb(7,17,27)
            font-weight :700
          .price
            position :absolute
            right:90px
            bottom:12px
            line-height :24px
            font-size:14px
            color:rgb(240,20,20)
            font-weight :700
          .cartcontrol-wrapper
            position :absolute
            right:0
            bottom:6px
  .list-mask
    position :fixed
    top:0
    left:0
    width:100%
    height:100%
    z-index:40
    -webkit-backdrop-filter :blur(10)
    background-color :rgba(7,17,27,0.6)
    &.fade-enter-active,&.fade-leave
      transition :all 0.5s
    &.fade-enter,&.fade-leave-active
      opacity :0
      background :rgba(7,17,27,0)
</style>