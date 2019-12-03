<template>
  <div id="home">
    <nav-bar class="nav-bar"><div slot="center">购物街</div></nav-bar>
    <tab-control v-show="isTabFixed" class="fixed" @itemClick="tabClick"
                 :titles="['流行', '新款', '精选']"></tab-control>
    <scroll class="content"
            ref="scroll"
            @scroll="contentScroll"
         
            :data="showGoodsList"
            :pull-up-load="true"
            :probe-type="3">
      <div>
        <home-swiper :banners="banners"
                     ref="hSwiper"></home-swiper>
        <feature-view :features="recommends"></feature-view>
        <recommend-view></recommend-view>
        <tab-control @tabClick="tabClick"
                     :titles="['流行', '新款', '精选']"
                     ref="tabControl"></tab-control>
        <goods-list :goods-list="showGoodsList"></goods-list>
      </div>
    </scroll>
    <back-top @click.native="backClick" class="back-top" v-show="showBackTop">
      <img src="~assets/img/common/top.png" alt="">
    </back-top>
  </div>
</template>

<script>
  import NavBar from 'components/common/navbar/NavBar'
  import Scroll from 'components/common/scroll/Scroll'
  import TabControl from 'components/content/tabControl/TabControl'
  import BackTop from 'components/content/backTop/BackTop'
 
  import HomeSwiper from './childComps/HomeSwiper'
  import FeatureView from './childComps/FeatureView'
  import RecommendView from './childComps/RecommendView'
  import GoodsList from '.components/content/goods/GoodsList'

  import {getHomeMultidata, getHomeData, RECOMMEND, BANNER} from "network/home";
  import {NEW, POP, SELL, BACKTOP_DISTANCE} from "@/common/const";

  


  export default {
		name: "Home",
    components: {
		  NavBar,
      Scroll,
      TabControl,
      BackTop,
      HomeSwiper,
      FeatureView,
      RecommendView,
      GoodsList,
    },
    data() {
		  return {
		    banners: [],
        recommends: [],
        goodsList: {
          'pop': {page: 1, list: []},
          'new': {page: 1, list: []},
          'sell': {page: 1, list: []}
        },
        currentType: POP,
        isTabFixed: false,
        tabOffsetTop: 0,
        showBackTop: false,
        tabOffsetTop:0
      }
    },
    computed: {
		  showGoodsList() {
		    return this.goodsList[this.currentType].list
      }
    },
    created() {
      console.log('创建Home');
      // 1.请求多个数据
      this.getMultiData()

      // 2.请求商品数据
      this.getHomeProducts(POP)
      this.getHomeProducts(NEW)
      this.getHomeProducts(SELL)


    
    },
     mounted() {
       this.debounce(this.$refs.scroll.refrech,500)
         this.$bus.$on('itemImageLoad',()=>{
          this.$refs.scroll.refrech()
      })
     },
    activated: function () {
      this.$refs.hSwiper.startTimer()
    },
    deactivated: function () {
      this.$refs.hSwiper.stopTimer()
    },
    updated() {
      // this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop
      // console.log(this.tabOffsetTop);
    },
    methods: {
 
       debounce(func,delay){
         let timer=null;
         return function (...args){
           if(timer)clearTimeout(timer)
           timer = setTimeout(()=>{
             func.apply(this,args)
           },delay)
         }
       },








		  tabClick(index) {
		    switch (index) {
          case 0:
            this.currentType = POP
            break
          case 1:
            this.currentType = NEW
            break
          case 2:
            this.currentType = SELL
            break
        }
      },
      contentScroll(position) {
		    // 1.决定tabFixed是否显示
        this.isTabFixed = position.y < -this.tabOffsetTop

        // 2.决定backTop是否显示
        this.showBackTop = position.y < -BACKTOP_DISTANCE
      },
     
      backClick() {
        this.$refs.scroll.scrollTo(0,0)
      },
      /**
       * 网络请求相关方法
       */ 
      getMultiData() {
        getHomeMultidata().then(res => {
          this.banners = res.data[BANNER].list
          this.recommends = res.data[RECOMMEND].list
          // 下次更新DOM时,获取新的tabOffsetTop值(不保险,可以在updated钩子中获取)
          this.$nextTick(() => {
            this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop
          })
        })
      },
      getHomeProducts(type) {
        getHomeData(type, this.goodsList[type].page).then(res => {
          const goodsList = res.data.list;
          this.goodsList[type].list.push(...goodsList)
          this.goodsList[type].page += 1

          
        })
      }
    }
	}
</script>

<style scoped>
  #home {
    /*position: relative;*/
    
    height: 100vh;
  }

  .nav-bar {
    background-color: var(--color-tint);
    font-weight: 700;
    color: #fff;
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
  }
  
  .content {
    
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;


  }

  .fixed {
    position: fixed;
    top: 44px;
    left: 0;
    right: 0;
  }

  .back-top {
    position: fixed;
    right: 10px;
    bottom: 60px;
  }

</style>
