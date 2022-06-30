<template>
  <view class="content">
    <scroll-view scroll-x="true" class='scroll-content' :scroll-into-view="'top'+topBarIndex">
      <view :id="'top'+index" class='scroll-item' v-for='(item,index) in topBar' :key='index' @tap='changeTab(index)'>
        <text :class='topBarIndex===index? "f-active-color":"f-color"'>{{item.name}}</text>
      </view>
    </scroll-view>
    <swiper class="swiper-wrap" @change="onChangeTab" :current="topBarIndex" :style="`height:${clientHeight}px;`">
      <swiper-item v-for="(item, index) in newTopBar" :key="index">
        <scroll-view scroll-y="true" :style="'height:'+clientHeight+'px;'" @scrolltolower='loadMore(index)'>
          <block v-if="item.data.length > 0">
            <block v-for="(k,i) in item.data" :key="i">

              <!--推荐-->
              <IndexSwiper v-if="k.type==='swiperList'" :dataList="k.data"></IndexSwiper>

              <template v-if='k.type==="recommendList"'>
                <Recommend :dataList='k.data'></Recommend>
                <Card cardTitle='猜你喜欢'></Card>
              </template>

              <!--运动户外....-->
              <Banner v-if='k.type==="bannerList"' :dataList='k.imgUrl'></Banner>

              <template v-if='k.type==="iconsList"'>
                <Icons :dataList='k.data'></Icons>
                <Card cardTitle='热销爆品'></Card>
              </template>

              <template v-if='k.type==="hotList"'>
                <Hot :dataList='k.data'></Hot>
                <Card cardTitle='推荐店铺'></Card>
              </template>

              <template v-if='k.type==="shopList"'>
                <Shop :dataList='k.data'></Shop>
                <Card cardTitle='为您推荐'></Card>
              </template>

              <CommodityList v-if='k.type==="commodityList"' :dataList='k.data'></CommodityList>


            </block>
          </block>
          <view v-else>
            暂无数据....
          </view>

          <view class='load-text f-color'>
            {{item.loadText}}
          </view>
        </scroll-view>

      </swiper-item>
    </swiper>

    <!--    <IndexSwiper></IndexSwiper>
    <Recommend></Recommend>
    <Card cardTitle="猜你喜欢"></Card>
    <CommodityList></CommodityList> -->

    <!-- <Banner></Banner>
    <Icons></Icons>
    <Card cardTitle="热销商品"></Card>
    <Hot></Hot>
    <Card cardTitle="推荐商品"></Card>
    <Shop></Shop>
    <Card cardTitle="为你推荐"></Card>
    <CommodityList></CommodityList> -->
  </view>
</template>

<script>
  import $http from '@/common/api/request.js'
  import IndexSwiper from '@/components/index/IndexSwiper.vue'
  import Recommend from '@/components/index/Recommend.vue'
  import Card from '@/components/common/Card.vue'
  import CommodityList from '@/components/common/CommodityList.vue'
  import Banner from '@/components/index/Banner.vue'
  import Icons from '@/components/index/Icons.vue'
  import Hot from '@/components/index/Hot.vue'
  import Shop from '@/components/index/Shop.vue'
  export default {
    components: {
      IndexSwiper,
      Recommend,
      Card,
      CommodityList,
      Banner,
      Icons,
      Hot,
      Shop
    },
    data() {
      return {
        topBarIndex: 0, // 顶部导航选中的索引值
        clientHeight: 0, // 内容高度
        topBar: [], // 导航条内容
        newTopBar: [] // 承载数据
      }
    },
    onLoad() {
      this.__init() // 请求数据
    },
    onReady() {

      uni.getSystemInfo({
        // 获得页面减去导航的内容高度
        success: (res) => {
          this.clientHeight = res.windowHeight - this.getClientHeight();
        }
      })

      // 自定义组件无法获得高度
      // let el = uni.createSelectorQuery().select('.home-data')
      // el.boundingClientRect(data => {
      //   // 获得.home-data的内容高度
      //   this.clientHeight = 2000
      //   // this.clientHeight = data.height
      // }).exec()
    },
    //导航标题栏 搜索按钮是跳转
    onNavigationBarButtonTap(e) {
      if (e.float == 'left') {
        uni.navigateTo({
          url: '/pages/search/search'
        })
      }
    },
    methods: {
      __init() {
        $http.request({
          url: "/index_list/data"
        }).then((res) => {
          this.topBar = res.topBar;
          this.newTopBar = this.initData(res);
        }).catch(() => {
          uni.showToast({
            title: '请求失败',
            icon: 'none'
          })
        })
      },

      initData(res) { // 初始化请求数据
        let arr = [];
        for (let i = 0; i < this.topBar.length; i++) {
          let obj = {
            data: [],
            load: "first", // 第一次请求
            loadText: "上拉加载更多..."
          }
          //获取首次数据
          if (i == 0) {
            obj.data = res.data;
          }
          arr.push(obj)
        }
        return arr;
      },
      changeTab(index) { // 顶部导航条点击触发
        if (index == this.topBarIndex) {
          return
        }
        this.topBarIndex = index
        //第一次的值为first才请求数据
        if (this.newTopBar[this.topBarIndex].load === 'first') {
          this.addData();
        }
      },

      onChangeTab(e) { // 页面index改变滑动触发
        this.changeTab(e.detail.current)
      },
      //请求不同页面数据
      addData(callback) {
        console.log('add');
        //拿到索引
        let index = this.topBarIndex
        //拿到id===>不同的板块
        let id = this.topBar[index].id

        // 下拉加载更多的索引, 不太理解
        let page = Math.ceil(this.newTopBar[index].data.length / 5) + 1;

        //请求数据
        $http.request({
          url: '/index_list/' + id + '/data/' + page + ''
        }).then((res) => {
          this.newTopBar[index].data = [...this.newTopBar[index].data, ...res];
        }).catch(() => {
          uni.showToast({
            title: '请求失败',
            icon: 'none'
          })
        })



        //当请求结束后，重新赋值
        this.newTopBar[index].load = 'last'
        if (typeof callback === 'function') {
          callback();
        }
      },
      //上拉加载更多
      loadMore(index) {
        this.newTopBar[index].loadText = '加载中...';
        //请求完数据 ，文字提示信息又换成【上拉加载更多...】

        this.addData(() => {
          this.newTopBar[index].loadText = '上拉加载更多...';
        })
      },
      getClientHeight() { //获取可视区域高度【兼容】处理ios和安卓刘海
        const res = uni.getSystemInfoSync()
        const system = res.platform
        if (system === 'ios') {
          return 44 + res.statusBarHeight
        } else if (system === 'android') {
          return 48 + res.statusBarHeight
        } else {
          return 0;
        }
      }

    }
  }
</script>

<style>
  .content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .scroll-content {
    width: 100%;
    height: 80rpx;
    white-space: nowrap;
  }

  .scroll-item {
    display: inline-block;
    padding: 10rpx 20rpx;
    font-size: 32rpx;
  }

  .f-active-color {
    padding: 10rpx 0;
    border-bottom: 6rpx solid #49BDFB;
  }

  .load-text {
    border-top: 2rpx solid #636263;
    line-height: 60rpx;
    text-align: center;
  }

  .swiper-wrap {
    width: 100%;
  }

  .load-text {
    border-top: 2rpx solid #636263;
    line-height: 60rpx;
    text-align: center;
  }
</style>
