<template>
  <div class="ratings" ref="ratings">
    <div class="rating-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <start :size="36" :score="seller.serviceScore"></start>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <start :size="36" :score="seller.foodScore"></start>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper">
            <span class="title">送达时间</span>
            <span class="time">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <ratingselect :ratings="ratings" :select-type="selectType" :only-content="onlyContent" :desc="desc" @select="selectRating" @change="changeOnlyContent">
      </ratingselect>
      <div class="rating-wrapper">
        <ul>
          <li v-for="(rating,index) in ratings" class="rating-item" :key="index" v-show="needShow(rating.rateType,rating.text)">
            <div class="avatar">
              <img :src="rating.avatar" width="28" height="28">
            </div>
            <div class="content">
              <h1 class="name">{{rating.username}}</h1>
              <div class="star-wrapper">
                <start :size="24" :score="rating.score"></start>
                <span class="delivery" v-show="rating.deliveryTime">{{rating.deliveryTime}}分钟送达</span>
              </div>
              <div class="text">{{rating.text}}</div>
              <div class="recommend" v-show="rating.recommend && rating.recommend.length">
                <span class="icon-thumb_up"></span>
                <span class="item" v-for="(item,index) in rating.recommend" :key="index">{{item}}</span>
              </div>
              <div class="time">{{rating.rateTime | formateDate}}</div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
 <script type="text/ecmascript-6">
import Bscroll from 'better-scroll'
import start from '../start/start.vue'
import split from '../split/split.vue'
import ratingselect from '../ratingselect/ratingselect.vue'
import { formateDate } from '../../common/js/date.js'
const ALL = 2
const ERR_OK = 0
export default {
  props: {
    seller: {
      type: Object
    }
  },
  filters: {
    formateDate(time) {
      let date = new Date(time)
      return formateDate(date, 'yyyy-MM-dd hh:mm')
    }
  },
  components: {
    start,
    split,
    ratingselect
  },
  created() {
    this.$http.get('/api/ratings').then((response) => {
      response = response.body
      if (response.errno === ERR_OK) {
        this.ratings = response.data
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new Bscroll(this.$refs.ratings, {
              click: true
            })
          } else {
            this.scroll.refresh()
          }
        })
      }
    })
  },
  data() {
    return {
      ratings: [],
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '满意',
        negative: '不满意'
      }
    }
  },
  methods: {
    selectRating(type) {
      this.selectType = type
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    changeOnlyContent(onlyContent) {
      this.onlyContent = !onlyContent
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    needShow(type, text) {
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
  .ratings
    position :absolute
    top:174px
    bottom:0
    width:100%
    left:0
    overflow :hidden
    .overview
      display :flex
      padding:18px 0
      .overview-left
        flex:0 0 137px
        width:137px
        text-align :center
        padding:6px 0
        border-right:1px solid rgba(7,17,27,0.1)
        @media only screen and (max-width:320px)
          flex:0 0 120px
          width:120px
        .score
          line-height:28px
          font-size:24px
          color:rgb(255,153,0)
          margin-bottom :6px
        .title
          line-height:12px
          font-size:12px
          color:rgb(7,17,27)
          margin-bottom :6px
        .rank
          line-height:10px
          font-size:12px
          color:rgb(147,153,159)
      .overview-right
        flex:1
        padding:6px 0 0 24px
        @media only screen and (max-width:320px) 
         padding:6px 0 0 6px
        .score-wrapper
          margin-bottom :8px
          font-size:0
          .title
            font-size:12px
            color:rgb(7,17,27)
            line-height:18px
          .star
            display :inline-block
            vertical-align :top
            margin :0 8px
          .score
            font-size:12px
            color:rgb(255,153,0)
            line-height:18px
            display :inline-block
            vertical-align :top
        .delivery-wrapper
          margin-bottom :8px
          font-size:0
          .title444444
            font-size:12px
            color:rgb(7,17,27)
            line-height:18px
          .time
            color:rgb(147,153,159)
            line-height :18px
            font-size:12px
            margin :0 8px
    .rating-wrapper
      padding:0 18px
      .rating-item
        display :flex
        padding:18px 0
        border-1px(rgba(7,17,27,0.1))
        .avatar
          flex: 0 0 28px
          width:28px
          margin-right:12px
          img
            border-radius :50%
        .content
          flex:1
          position :relative
          .name
            font-size:10px
            color:rgb(7,17,27)
            line-height :12px
            margin-bottom :4px
          .star-wrapper
            font-size:0
            margin-bottom :6px
            .star
              vertical-align :top
              display :inline-block
              margin-right :6px
            .delivery
              vertical-align :top
              display :inline-block
              line-height :12px
              font-size:12px
              font-weight :200
              color:rgb(147,153,159)
          .text
            line-height :18px
            color:rgb(7,17,27)
            font-size:12px
            margin-bottom :18px
          .recommend
            font-size:0
            line-height :16px
            .icon-thumb_up,.item
              display :inline-block
              margin :0 8px 4px 0
              font-size:9px
            .icon-thumb_up
              color:rgb(0,160,220)
            .item
              padding:0 6px
              border:1px solid rgba(7,17,27,0.1)
              border-radius :1px
              color:rgb(147,153,159)
              background :#fff
              max-width:60px
          .time
            position :absolute
            right:0
            top:0
            font-size:10px
            line-height :12px
            color:rgb(147,153,159)
</style>