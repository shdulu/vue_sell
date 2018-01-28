<template>
  <div class="goods">
    <div class="menu-wrapper">
      <ul>
        <li :key="index" @click="selectMenu(index)" class="menu-item" :class="{'current':currentIndex===index}" v-for="(item,index) in goods">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper">
      <ul>
        <li :key="index" v-for="(item,index) in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
              <div class="icon">
                <img :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <v-cartcontrol :food="food"></v-cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <v-shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></v-shopcart>
    <v-food @add="addFood" :food="selectedFood" ref="food"></v-food>
    <!-- ref="food" 调用子组件中的方法 -->
  
  </div>
    
</template>
<script type="text/ecmascript-6">
    import BScroll from 'better-scroll';
    import shopcart from '../shopcart/shopcart.vue';
    import cartcontrol from '../cartcontrol/cartcontrol.vue';
    import food from '../food/food.vue';
    const ERR_OK = 0;
    export default {
      props: {
        seller: {
          type: Object
        }
      },
      data() {
        return {
          goods: [],
          listHeight: [],
          scrollY :0,
          selectedFood:{}
        }
      },
      computed: {
        currentIndex() {
          for(let i=0; i< this.listHeight.length; i++){
            let height1 = this.listHeight[i];
            let height2 =this.listHeight[i+1];
            if(!height2 || (this.scrollY>=height1 && this.scrollY<height2)){
              return i;
            }
          }
          return 0;
        },
        selectFoods() {
          let foods =[];
          this.goods.forEach((good) => {
            good.foods.forEach((food) => {
              if(food.count){
                foods.push(food);
              }
            })
          })
          return foods;
        }
      },
      created() {
        this.classMap = ['decrease','discount','special','invoice','guarantee'];
        this.$http.get('./api/goods').then((response) => {
          response = response.body;
          if(response.errno === ERR_OK) {
            this.goods = response.data;
            this.$nextTick(()=>{   //更新数据到页面渲染异步过程 初始化要在DOM重新渲染之后 使用到 this.$nextTick()
              this._initScroll();
              this._calculateHeight();
            })
          }
        })
      },
      methods: {
        selectMenu(index) {
          // console.log(index);
          let foodList = document.querySelectorAll(".food-list-hook");
          let el = foodList[index];
          this.foodsScroll.scrollToElement(el,300);
        },
        selectFood(food,event) {
          this.selectedFood=food;
          this.$refs.food.show();  //点击调用子组件food中的方法
        },
        addFood(target){
          this._drop(target);
        },
        _drop(target) {
          this.$nextTick(() => {
            this.$refs.shopcart.drop(target);
          })
        },
        _initScroll() {
          this.menuScroll = new BScroll('.menu-wrapper',{
            click: true  //better-scroll 调用的时候默认不会调用 click 事件
          });
          this.foodsScroll = new BScroll('.foods-wrapper',{
            click: true,
            probeType: 3  //实时监听滚动位置
          });
          this.foodsScroll.on('scroll',(pos)=>{
            this.scrollY = Math.abs(Math.round(pos.y));
            // pos.y 实时滚动位置获取
          })
        },
        _calculateHeight() {
          let foodList = document.querySelectorAll(".food-list-hook");
          let height = 0;
          this.listHeight.push(height);
          for(let i=0; i<foodList.length; i++){
            let item = foodList[i];
            height +=item.clientHeight;
            this.listHeight.push(height);
          }
          // console.log(this.listHeight);
        }
      },
      components: {
        'v-shopcart':shopcart,
        'v-cartcontrol':cartcontrol,
        'v-food':food
      }
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl";
  .goods
    display flex
    position :absolute
    top :174px
    bottom :46px
    width :100%
    overflow :hidden
    .menu-wrapper
      flex :0 0 80px
      width :80px
      background :#f3f5f7
      .menu-item
        display :table
        height :54px
        width :56px
        padding :0 12px
        line-height :14px
        &.current
          position :relative
          z-index :10
          margin-top :-1px
          background :#fff
          font-weight :700
          .text
            border-none()
        .icon
          display :inline-block
          vertical-align :top
          width :12px
          height :12px
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
          display : table-cell
          width :56px
          vertical-align :middle
          border-1px(rgba(7,17,27,.1))
          font-size :12px
    .foods-wrapper
      flex :1
      .title
        padding-left:14px
        height :26px
        line-height :26px
        border-left :2px solid #d9dde1
        font-size :12px
        color :rgb(147,153,159)
        background :#f3f5f7
      .food-item
        display :flex
        margin :18px
        padding-bottom :18px
        border-1px(rgba(7,17,27,.1))
        &:last-child
          border-none()
          margin-bottom :0
        .icon
          flex :0 0 57px
          margin-right :10px
          img
            display :block
            width :57px
        .content
          position :relative
          flex :1
          .name
            margin :2px 0 8px 0
            height :14px
            line-height :14px
            font-size :14px
            color :rgb(7,17,27)
          .desc, .extra
            line-height :10px
            font-size :10px
            color :rgb(147,153,159)
          .desc 
            margin-bottom :8px
            line-height :12px
          .extra
           .count
            margin-right :12px
          .price
            font-weight :700
            line-height :24px
            .now
              margin-right :8px
              font-size :14px
              font-weight :700
              color :rgb(240,20,20)
            .old
              text-decoration :line-through
              font-size :10px
              color :rgb(147,153,159)
          .cartcontrol-wrapper
            position :absolute
            right :0
            bottom :-10px
          
</style>

