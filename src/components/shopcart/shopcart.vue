<template>
<div>
    <div class="shopcart">
        <div @click="toggleList" class="content">
            <div class="content-left">
                <div class="logo-wrapper">
                    <div class="logo" :class="{'highLight':totalCount>0}">
                        <i class="icon-shopping_cart" :class="{'highLight':totalCount>0}"></i>
                    </div>
                    <div class="num" v-show="totalCount>0">{{totalCount}}</div>
                </div>
                <div class="price" :class="{'highLight':totalPrice>0}">￥{{totalPrice}}元</div>
                <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
            </div>
            <div class="content-right">
                <div @click.stop.prevent="pay" class="pay" :class="payClass">
                    {{payDesc}}
                </div>
            </div>
        </div>
        <transition name="fold">
            <div class="shopcart-list" v-show="listShow">
                <div class="list-header">
                        <div class="title">购物车</div>
                        <span @click="empty" class="empty">清空</span>
                </div>
                <div class="list-content">
                    <ul>
                        <li class="food" v-for="food in selectFoods">
                            <span class="name">{{food.name}}</span>
                            <div class="price">
                                <span>￥{{food.price*food.count}}</span>
                            </div>
                            <div class="cartcontrol-wrapper">
                                <v-cartcontrol :food="food"></v-cartcontrol>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </transition>
    </div>
    <transition name="mask">
        <div @click="hideList" class="list-mask" v-show="listShow"></div>
    </transition>
</div>
</template>

<script>
    import BScroll from 'better-scroll';
    import cartcontrol from '../cartcontrol/cartcontrol.vue';
    export default {
        props: {
            selectFoods: {
                type: Array,
                default() {
                    return [];
                }
            },
            deliveryPrice:{
                type: Number,
                default: 0
            },
            minPrice: {
                type: Number,
                default: 0
            }
        },
        data (){
            return {
                fold: true
            }
        },
        computed: {
            totalPrice() {
                let total = 0;
                this.selectFoods.forEach((food) => {
                    total += food.price*food.count;
                });
                return total;
            },
            totalCount() {
                let count = 0;
                this.selectFoods.forEach((food) => {
                    count +=food.count;
                });
                return count;
            },
            payDesc() {
                if(this.totalPrice === 0){
                    return `￥${this.minPrice}元起送`;
                }else if(this.totalPrice<this.minPrice){
                    let diff = this.minPrice - this.totalPrice;
                    return `还差￥${diff}元起送`;
                }else{
                    return "去结算";
                }
            },
            payClass() {
                if(this.totalPrice<this.minPrice){
                    return "not-enough"
                }else{
                    return "enough"
                }
            },
            listShow() {
                if(!this.totalCount){
                    this.fold = true;
                    return false;
                }
                let show = !this.fold;
                if(show){
                    this.$nextTick(() => {
                        if(!this.scroll){
                            this.scroll = new BScroll('.list-content',{
                                click: true
                            });
                        }else{
                            this.scroll.refresh();
                        }
                    })
                }
                return show;
            }
        },
        methods: {
            toggleList() {
                if(!this.totalCount){
                    return;
                }
                this.fold = !this.fold;
            },
            empty() {
                this.selectFoods.forEach((food)=>{
                    food.count = 0;
                })
            },
            hideList() {
                this.fold = true;
            },
            pay() {
                if(this.totalPrice<this.minPrice){
                    return;
                }
                window.alert(`支付${this.totalPrice}元`);
            }
        },
        components: {
            'v-cartcontrol':cartcontrol
        }
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
    @import "../../common/stylus/mixin.styl";
    .shopcart
        position :fixed
        left :0
        bottom :0
        z-index :50
        height :48px
        width :100%
        background :#000
        .content
            display :flex
            background :#141d27
            font-size :0
            color :rgba(255,255,255,.4)
            .content-left
                flex :1
                .logo-wrapper
                    display :inline-block
                    position :relative
                    top :-10px
                    margin :0 12px
                    padding :6px
                    width :56px
                    height :56px
                    box-sizing :border-box
                    vertical-align :top
                    border-radius :50%
                    background :#141d27
                    .logo
                        width :100%
                        height :100%
                        border-radius :50%
                        background :#2b343c
                        text-align :center
                        &.highLight
                            background :rgb(0,160,220)
                        .icon-shopping_cart
                            font-size :24px
                            line-height :44px
                            color :#80858a
                            &.highLight
                                color :#fff
                    .num
                        position :absolute
                        top :0
                        right :0
                        width :24px
                        height :16px
                        line-height :16px
                        text-align :center
                        border-radius :16px
                        font-size :9px
                        font-weight :700
                        background :rgb(240,20,20)
                        box-shadow :0 4px 8px 0 rgba(0,0,0,.4)
                .price
                    display :inline-block
                    vertical-align :top
                    line-height :24px
                    margin-top :12px
                    padding-right :12px
                    box-sizing :border-box
                    border-right :1px solid rgba(255,255,255,.1)
                    font-size :16px
                    font-weight :700
                    &.highLight
                        color:#fff
                .desc
                    display :inline-block
                    vertical-align :top
                    line-height :24px
                    margin :12px 0 0 12px
                    font-size :10px   
            .content-right
                flex :0 0 105px
                width :105px
                .pay
                    height :48px
                    line-height :48px
                    text-align :center
                    font-size :12px
                    font-weight :700
                    &.not-enough
                        background :#2b333b
                    &.enough
                        background :#00b43c
                        color :#fff
        .shopcart-list
            position :absolute
            top :0
            left :0
            z-index :-1
            width :100%
            transition :all .4s
            transform :translate3d(0,-100%,0)
            &.fold-enter-active,&.fold-leave-active
                transform :translate3d(0,0,0)
            .list-header
                height :40px
                line-height :40px
                padding : 0 18px
                background :#f3f5f7
                border-bottom :1px solid rgba(7,17,27,.1)
                .title
                    float :left
                    font-size :14px
                    color :rgb(7,17,27)
                .empty
                    float :right
                    font-size :12px
                    color :rgb(0,160,220)
            .list-content
                padding :0 18px
                max-height :217px
                background :#fff
                overflow :hidden
                .food
                    position :relative
                    padding :12px 0
                    box-sizing :border-box
                    border-1px(rgba(7,17,27,.1))
                    .name
                        line-height :24px
                        font-size :14px
                        color :rgb(7,17,27)
                    .price
                        position absolute
                        right :90px
                        bottom :12px
                        line-height :24px
                        font-size :14px
                        color :rgb(240,20,20)
                        font-weight :700
                    .cartcontrol-wrapper
                        position :absolute
                        bottom :6px
                        right :0


    .list-mask
        position :fixed
        top :0
        left :0
        width :100%
        height :100%
        z-index :40
        background-filter :blur(10px)
        background :rgba(7,17,27,.6)
        transition :all .4s
        &.mask-enter-active,&.mask-leave-active
            opacity :0
            transform :translate3d(0,100%,0)

</style>

