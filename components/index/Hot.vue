<template>
  <view>
    <Commodity :dataList='dataList' itemW='250rpx' bigH='220rpx'></Commodity>
  </view>
</template>

<script>
  import Commodity from '@/components/common/Commodity.vue'
  export default {
    props: {
      dataList: Array
    },
    components: {
      Commodity
    }
  }
</script>

<style scoped>

</style>
