<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item,index) in goods" :key="item.id" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
            <span class="text border-1px" >
              <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
              {{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodWrapper">
        <ul>
          <li v-for="item in goods" :key="item.id" class="food-list food-list-hook">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li  v-for="food in item.foods" :key="food.id" @click="selectFood(food,$event)" class="food-item border-1px">
                <div class="icon">
                  <img :src="food.icon" width="57" height="57">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span>
                    <span>好评率{{food.sellCount}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food" @cartAdd="addFood"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" :select-foods="selectFoods" ref="shopcart"></shopcart>
    </div>
    <food :food="selectedFood" ref="food"  @cartAdd="addFood"></food>
  </div>
</template>
 <script type="text/ecmascript-6">
  import Bscroll from 'better-scroll'
  import shopcart from '../shopcart/shopcart.vue'
  import cartcontrol from '../cartcontrol/cartcontrol.vue'
  import food from '../food/food.vue'

  const ERR_OK = 0
  export default {
    props: {
      seller: {
        type: Object
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food
    },
    data () {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      }
    },
    created () {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
      this.$http.get('/api/goods').then((response) => {
        response = response.body
        if (response.errno === ERR_OK) {
          this.goods = response.data
          this.$nextTick(() => {
            this._initScroll()
            this._calculateHeight()
          })
        }
      })
    },
    computed: {
      currentIndex () {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i]
          let height2 = this.listHeight[i + 1]
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i
          }
        }
        return 0
      },
      selectFoods () {
        let foods = []
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
        return foods
      }
    },
    methods: {
      _initScroll () {
        this.menuScroll = new Bscroll(this.$refs.menuWrapper, {
          click: true
        })
        this.foodScroll = new Bscroll(this.$refs.foodWrapper, {
          click: true,
          probeType: 3 // 实时告诉位置
        })
        this.foodScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y))
        })
      },
      _calculateHeight () {
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectMenu (index, event) {
        if (!event._constructed) {
          return false
        }
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook')
        let el = foodList[index]
        this.foodScroll.scrollToElement(el, 300)
      },
      addFood (target) {
        this._drop(target)
      },
      _drop (target) {
        // 优化动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target)// chuanru
        })
      },
      selectFood (food, event) {
        if (!event._constructed) {
          return false
        }
        this.selectedFood = food
        this.$refs.food.show()
      }
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  @import "../../common/stylus/mixin.styl"
  .goods
    position:absolute
    top:174px
    bottom:46px
    width:100%
    display :flex
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width:80px
      background :#f3f5f7
      .menu-item
        display:table
        width :56px
        height:54px
        padding:0 12px
        line-height :14px
        &.current
          font-weight :700px
          position :relative
          margin-top:1px
          background :#fff
          .text
            border-none()
        .icon
          display :inline-block
          width:12px
          height:12px
          vertical-align :top
          margin-right :2px
          background-size :12px 12px
          background-repeat :no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          font-size:12px
          display :table-cell
          width:56px
          vertical-align :middle
          border-1px(rgba(7,17,27,0.1))
    .foods-wrapper
      flex:1
      .title
        padding-left:14px
        height:26px
        line-height :26px
        border-left:2px solid #d9dde1
        font-size:12px
        color:rgb(147,153,159)
        background :#f3f5f7
      .food-item
        display :flex
        margin :18px
        padding-bottom :18px
        border-1px(rgba(7,17,27,0.1))
        &.last-child
          border-none()
          margin-bottom :0
        .icon
          flex: 0 0 57px
          margin-right :10px
        .content
          flex:1
          .name
            font-size:14px
            margin:2px 0 8px 0
            line-height :14px
            height:14px
            color:rgb(7,17,27)
          .desc,.extra
            line-height :10px
            font-size:10px
            color:rgb(147,153,159)
          .desc
            line-height :12px
            margin-bottom :8px
          .extra
            .count
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
            position :absolute
            right :0
            bottom:12px
</style>
