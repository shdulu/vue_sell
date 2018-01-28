<template>
    <div class="cartcontrol">
        <transition name="move">
            <div @click.stop.prevent="decreaseCart" v-show="food.count>0" class="cart-decrease">
                <div class="inner icon-remove_circle_outline"></div>
            </div>
        </transition>
        <div v-show="food.count>0" class="cart-count">{{food.count}}</div>
        <div @click.stop.prevent="addCart" class="cart-add icon-add_circle"></div>
    </div>
</template>

<script>
    import Vue from 'vue';
    export default {
        props: {
            food:{
                type: Object
            }
        },
        methods: {
            addCart() {
                // 此处默认点击事件失效，需要在goods组件 better-scroll 中设置click:true
                // console.log("测试点击是否生效")
                if(!this.food.count){
                    Vue.set(this.food,'count',1)  //用于避开 Vue 不能检测属性被添加的限制。
                }else{
                    this.food.count++;
                }
            },
            decreaseCart() {
                if(this.food.count){
                    this.food.count--;
                }
            }
        }
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
    .cartcontrol
        font-size :0
        .cart-decrease
            display :inline-block
            padding :6px
            transition :all 0.4s linear
            .inner
                line-height :24px
                font-size :24px
                color :rgb(0,160,220)
            &.move-enter-active,&.move-leave-active
                opacity :0
                transform :translateX(24px)
        .cart-count
            display :inline-block
            vertical-align :top
            width :12px
            padding-top :6px
            line-height :24px
            text-align :center
            font-size :10px
            color :rgb(147,153,159)
        .cart-add
            display :inline-block
            padding :6px
            line-height :24px
            font-size :24px
            color :rgb(0,160,220)
            
</style>
