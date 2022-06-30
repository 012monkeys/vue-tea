<template>
  <swiper :indicator-dots="true" :autoplay="false" :interval="3000" :duration="1000">
    <swiper-item v-for='(item,index) in dataList' :key='index'>
      <view class="swiper-item">
        <image class='swiper-img' :src="item.imgUrl" mode=""></image>
      </view>
    </swiper-item>
  </swiper>
</template>

<script>
  export default {
    props: {
      dataList: Array
    },
    data() {
      return {

      }
    },
    methods: {

    }
  }
</script>

<style scoped>
  swiper {
    width: 100%;
    height: 300rpx;
  }

  .swiper-img {
    width: 100%;
    height: 300rpx;
  }
</style>
