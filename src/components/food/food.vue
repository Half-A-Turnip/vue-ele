<template>
  <transition name="move">
     <div v-show="showFlag" class="food" ref="food">
       <div class="food-content">
         <div class="image-header">
           <img :src="food.image" alt="">
           <div class="back" @click="back">
            <i class="icon-arrow_lift"></i>
           </div>
         </div>
         <div class="content">
           <h1 class="title">{{food.name}}</h1>
           <div class="detail">
             <span class="sell-count">月售{{food.sellCount}}份</span>
             <span class="rating">好评率{{food.rating}}</span>
           </div>
           <div class="price">
              <span class="now">￥{{food.price}}</span>
              <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food" @cartAdd="addFood"></cartcontrol><!--点击小球动画记得绑定事件-->
          </div>
          <transition name="fade">
            <div class="buy" v-show="!food.count || food.count===0" @click="addFirst($event)">加入购物车</div>
          </transition>
         </div>
         <split v-show="food.info"></split>
         <div class="info" v-show="food.info">
           <h1 class="title">商品信息</h1>
           <p class="text">{{food.info}}</p>
         </div>
         <split v-show="food.ratings"></split>
         <div class="rating">
           <h1 class="title">商品评价</h1>
           <ratingselect 
            :ratings="food.ratings" 
            :select-type="selectType" 
            :only-content="onlyContent" 
            :desc="desc" 
            @select="selectRating"
            @change="changeOnlyContent">
          </ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-for="(rating,index) in food.ratings" :key="index" class="rating-item" v-show="needShow(rating.rateType,rating.text)">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img :src="rating.avatar" alt class="avatar" width="12" height="12">
                </div>
                <div class="time">{{rating.rateTime | formateDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>
                  {{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length"></div>
          </div>
         </div>
       </div>
     </div>
  </transition>
</template>

<script type="text/ecmascript-6">
import Vue from 'vue'
import Bscroll from 'better-scroll'
import cartcontrol from '../cartcontrol/cartcontrol.vue'
import ratingselect from '../ratingselect/ratingselect.vue'
import split from '../split/split.vue'
import {formateDate} from '../../common/js/date.js'

const ALL = 2

export default {
  props: {
    food: {
      type: Object
    }
  },
  components: {
    cartcontrol,
    ratingselect,
    split
  },
  filters: {
    formateDate (time) {
      let date = new Date(time)
      return formateDate(date, 'yyyy-MM-dd hh:mm')
    }
  },
  data () {
    return {
      showFlag: false,
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      }
    }
  },
  methods: {
    show () {
      this.showFlag = true
      this.selectType = ALL
      this.onlyContent = true
      this.$nextTick(() => {
        if (!this.scroll) {
          this.scroll = new Bscroll(this.$refs.food, {
            click: true
          })
        } else {
          this.scroll.refresh()
        }
      })
    },
    back () {
      this.showFlag = false
    },
    addFood (target) {
      // 当前组件必须在父组件 引入处，bangding @add="xxx",继而执行 父组件的 xxx 方法
      this.$emit('cartAdd', target)
    },
    addFirst (event) {
      if (!event._constructed) {
        return false
      }
      this.$emit('cartAdd', event.target)
      Vue.set(this.food, 'count', 1)
    },
    selectRating (type) {
      this.selectType = type
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    changeOnlyContent (onlyContent) {
      this.onlyContent = !onlyContent
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    needShow (type, text) {
      if (this.onlyContent && !text) {
        return false
      }
      if (this.selectType === ALL) {
        return true
      } else {
        return type === this.selectType
      }
    }
  }
}
</script>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  @import "../../common/stylus/mixin.styl"
  .food
    position :fixed
    left:0
    bottom:48px
    top:0
    z-index:30
    width:100%
    background :#fff
    transition all 0.2s linear
    transform :translate3d(0,0,0)
    &.move-enter,&.move-leave-active
      transform :translate3d(100%,0,0)
    .image-header
      position :relative
      width:100%
      height:0
      padding-top:100%
      img
        position :absolute
        top:0
        left:0
        width:100%
        height:100%
      .back
        position :absolute
        top:10px
        left:0
        .icon-arrow_lift
          display :block
          padding:10px
          font-size:20px
          color:#fff
    .content
      padding:18px
      position :relative
      .title
        font-weight :700
        line-height :14px
        margin-bottom :8px
        font-size:14px
        color:rgb(7,17,27)
      .detail
        margin-bottom :18px
        line-height :10px
        height:10px
        font-size:0
        .sell-count,.rating
          font-size:10px
          color:rgb(147,153,159)
        .sell-count
          margin-right :12px
      .price
        font-weight :400px
        line-height :24px
        .now
          font-size:14px
          margin-right :18px
          color:rgb(240,20,20)
        .old
          text-decoration :line-through
          font-size:10px
          color:rgb(147,153,159)
      .cartcontrol-wrapper
        position:absolute
        right:12px
        bottom:12px
      .buy
        position :absolute
        right:18px
        bottom:18px
        z-index:10
        height:24px;
        line-height :24px
        padding:0 12px
        box-sizing :border-box
        font-size:10px
        color:#fff
        border-radius :12px
        background :rgb(0,160,220)
        opacity :1
        &.fade-enter-active, &.fade-leave-active
          transition: all 1s
        &.fade-enter, &.fade-leave-active
          opacity: 0
          z-index: -1
    .info
      background :#fff
      padding:18px
      .title
        line-height :14px
        margin-bottom :6px
        font-size:14px
        color:rgb(7,17,27)
      .text
        font-size:12px
        font-weight :200
        color:rgb(77,85,93)
        line-height :24px
        padding:0 8px
    .rating
      padding-top :18px
      .title
        line-height:14px
        margin-left :18px
        font-size:14px
        color:rgb(7,17,27)
      .rating-wrapper
        padding:0 18px
        .rating-item
          position:relative
          padding:16px 0 
          border-1px(rgba(7,17,27,0.1))
          .user
            position :absolute
            right:0
            top:16px
            font-size:0
            height:12px
            line-height :12px
            .name
              color:rgb(147,153,159)
              font-size:10px
              margin-right :6px
              display :inline-block
              vertical-align :top
            .avatar
              border-radius :50%
          .time
            font-size:10px
            color:rgb(147,153,159)
            line-height :12px
            margin-bottom:6px
          .text
            line-height:16px
            font-size:12px
            color:rgb(7,17,27)
            .icon-thumb_up,.icon-thumb_down
              line-height :16px
              margin-right :4px
              font-size:12px
            .icon-thumb_up
              color:rgb(0,160,220)
            .icon-thumb_down
              color:rgb(147,153,159)
</style>