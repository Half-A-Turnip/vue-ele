<template>
  <div class="ratingselect">
    <div class="rating-type border-1px">
      <span @click="select(2,$event)" class="block positive" :class="{'active':selectType===2}">
        {{desc.all}}
        <b class="count">{{ratings.length}}</b>
      </span>
      <span @click="select(0,$event)" class="block positive" :class="{'active':selectType===0}">
        {{desc.positive}}
        <b class="count">{{positive.length}}</b>
      </span>
      <span @click="select(1,$event)" class="block negative" :class="{'active':selectType===1}">
        {{desc.negative}}
        <b class="count">{{negative.length}}</b>
      </span>
    </div>
    <div class="switch" :class="{'on':onlyContent}" @click="toggleContent">
      <span class="icon-check_circle"></span>
      <span class="text">只看内容的评价</span>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
const POSITIVE = 0
const NEGATIVE = 1
const ALL = 2

export default {
  props: {
    ratings: {
      type: Array,
      default () {
        return []
      }
    },
    selectType: {
      type: Number,
      default: ALL
    },
    onlyContent: {
      type: Boolean,
      default: false
    },
    desc: {
      type: Object,
      default () {
        return {
          all: '全部',
          positive: '满意',
          negative: '不满意'
        }
      }
    }
  },
  computed: {
    positive () {
      return this.ratings.filter((rating) => {
        return rating.rateType === POSITIVE
      })
    },
    negative () {
      return this.ratings.filter((rating) => {
        return rating.rateType === NEGATIVE
      })
    }
  },
  methods: {
    select (type, event) {
      if (!event._constructed) {
        return false
      } else {
        // this.selectType = type
        // 不可以在子组件里面随意更改父组件传来的值，可以通过$emit将需要改的值传给父组件再通过props传过来
        this.$emit('select', type)
      }
    },
    // 同select
    toggleContent (event) {
      if (!event._constructed) {
        return false
      } else {
        this.$emit('change', this.onlyContent)
      }
    }
  }
}
</script>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  @import "../../common/stylus/mixin.styl"
  .ratingselect
    .rating-type
      padding:18px 0
      margin :0 18px
      border-1px(rgba(7,17,27,0.1))
      font-size:0
      .block
        display :inline-block
        padding:8px 12px
        border-radius :2px
        margin-right :8px
        line-height :16px
        color:rgb(77,85,93)
        font-size:12px
        &.active
          color:#fff
        .count
          font-size:8px
          margin-left :2px
        &.positive
          background :rgba(0,160,220,0.2)
          &.active
            background :rgb(0,160,220)
        &.negative
          background :rgba(77,85,93,0.2)
          &.active
            background :rgb(77,85,93)
    .switch
      padding:12px 18px
      line-height :24px
      font-size:0
      border-bottom:1px solid rgba(7,17,27,0.1)
      color:rgb(147,153,159)
      &.on
        .icon-check_circle
          color:#00c850
      .icon-check_circle
        display :inline-block
        vertical-align :top
        font-size:24px
        margin-right :4px
      .text
        font-size:12px
</style>