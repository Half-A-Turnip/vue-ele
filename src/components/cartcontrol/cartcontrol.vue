<template>
  <div class="cartcontrol">
    <transition name="move">
      <div class="cart-decrease" v-show="food.count>0" @click="decreaseCart">
        <span class="inner icon-remove_circle_outline"></span>
      </div>
    </transition>
    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>
<script type="text/ecmascript-6">
import Vue from 'vue'
export default {
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    addCart (event) {
      if (!event._constructed) {
        return false
      }
      if (!this.food.count) {
        Vue.set(this.food, 'count', 1)
        // this.food.count = 1
      } else {
        this.food.count++
      }
      this.$emit('cartAdd', event.target)
    },
    decreaseCart (event) {
      if (!event._constructed) {
        return false
      }
      if (this.food.count) {
        this.food.count--
      }
    }
  }
}
</script>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  .cartcontrol
    font-size :0
    .cart-decrease
      display :inline-block
      padding:6px
      transition: all .4s linear
      transform :translate3d(0,0,0)
      &.move-enter,&.move-leave-active
        transition: all .4s linear
        opacity :0
        transform :translate3d(24px,0,0)
        .inner
          transform :rotate(180deg)
      .inner
        font-size:24px
        line-height :24px
        color:rgb(0,160,220)
        display :inline-block
        transition: all .4s linear
        transform:rotate(0deg)
    .cart-count
      display :inline-block
      color:rgb(147,153,159)
      font-size:10px
      line-height :24px
      width:12px
      padding-top:6px
      text-align :center
      vertical-align :top
    .cart-add
      display :inline-block
      padding:6px
      font-size:24px
      line-height :24px
      color:rgb(0,160,220)
</style>