<template>
  <div id="app">
    

    <view-box>
      <!-- 头部 -->
      <x-header
        slot="header"
        title="网易"
        style="background-color:#ee1a1a;"
        :left-options="{backText: ''}"
        :right-options="{showMore: false}"        
      >
        <div slot="right">搜索</div>
      </x-header>

      <!-- tab -->
      <tab class="tab">
        <tab-item selected>推荐</tab-item>
        <tab-item>要闻</tab-item>
        <tab-item>体育</tab-item>
        <tab-item>社会</tab-item>
        <tab-item>新时代</tab-item>
        <tab-item>娱乐</tab-item>
        <tab-item>房产</tab-item>
        <tab-item>养生</tab-item>
      </tab>

      <!-- 
        第三方组件： vue-scroller
        自定义滚动条刷新加载组件
        https://www.npmjs.com/package/vue-scroller

        安装： npm i vue-scroller -S 
      -->
      <scroller 
        class="scroller"
        :onRefresh="refresh"
        refreshText="下拉刷新"
        :onInfinite="infinite"
        ref="scrollRef"
      >

        <!-- swiper -->
        <swiper :auto="true" :list="swiperList" v-model="index"></swiper>      

        <!-- Marquee -->
        <marquee direction="up" :interval="3000">
          <marquee-item class="align-middle" v-for="(item,index) in marqueeList" :key="index">
            {{item.title}}
          </marquee-item>        
        </marquee>

        <!-- panel -->
        <panel :list="datalist"></panel>

        <panel :list="moreDatalist"></panel>

      </scroller>

      <!-- 底部 -->
      <tabbar slot="bottom" class="tabbar">
        <tabbar-item link="http://www.163.com">
          <img slot="icon" src="https://cms-bucket.nosdn.127.net/2018/08/24/2b4ea76a09be4034a7554b506df2b854.png?imageView&thumbnail=234y146&quality=45&interlace=1&enlarge=1&type=webp" alt="">
          <span slot="label">首页</span>
        </tabbar-item>
        <tabbar-item>
          <img slot="icon" src="https://cms-bucket.nosdn.127.net/2018/08/24/2b4ea76a09be4034a7554b506df2b854.png?imageView&thumbnail=234y146&quality=45&interlace=1&enlarge=1&type=webp" alt="">
          <span slot="label">视频</span>
        </tabbar-item>
        <tabbar-item>
          <img slot="icon" src="https://cms-bucket.nosdn.127.net/2018/08/24/2b4ea76a09be4034a7554b506df2b854.png?imageView&thumbnail=234y146&quality=45&interlace=1&enlarge=1&type=webp" alt="">
          <span slot="label">讲讲</span>
        </tabbar-item>
        <tabbar-item>
          <img slot="icon" src="https://cms-bucket.nosdn.127.net/2018/08/24/2b4ea76a09be4034a7554b506df2b854.png?imageView&thumbnail=234y146&quality=45&interlace=1&enlarge=1&type=webp" alt="">
          <span slot="label">我的</span>
        </tabbar-item>
      </tabbar>
    </view-box>
    <!-- <router-view/> -->
  </div>
</template>

<script>
import { ViewBox, XHeader, Tabbar, TabbarItem, Tab, TabItem, Marquee, MarqueeItem, Swiper, Panel, Icon } from "vux";

var refreshNum = ['A','B01','B02','B03','B04','B05','B06','B07','B08','B09','B010'];
var num = 0;
function getNum(){
  var key = refreshNum[num];
  num++;
  if(num == refreshNum.length){
    num = 0;
  }
  return key;
}

var start = 0;
var end = start + 10;

export default {
  name: "App",
  data() {
    // panel
    /*
    var datalist = [];
    for(var i=0; i<10; i++){
      datalist.push({
          src: 'http://somedomain.somdomain/x.jpg',
          fallbackSrc: 'http://placeholder.qiniudn.com/60x60/3cc51f/ffffff',
          title: '标题一',
          desc: '由各种物质组成的巨型球状天体，叫做星球。星球有一定的形状，有自己的运行轨道。',
          url: '/component/cell'
      })
    }
    */
    return {
      marqueeList: [],
      index: 0,
      swiperList: [],
      datalist: [],
      moreDatalist: []
    }
  },
  created() {
    this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html')
    .then(
      data => {
        // console.log(data)
        // marquee列表
        this.marqueeList = data.live.filter(item => {
          return item.addata === null
        }).map(item => {
          return {
            title: item.title
          }
        })

        // swiper列表
        this.swiperList = data.focus.filter(item => {
          return item.addata === null
        }).map(item => {
          return {
            url: item.link,
            img: item.picInfo[0].url,
            title: item.title
          }
        })

        // 首屏列表
        this.datalist = data.list.filter(item => {
          return item.picInfo.length != 0
        }).map(item => {
            return {
              src: item.picInfo[0].url,
              fallbackSrc: 'http://placeholder.qiniudn.com/60x60/ee1a1a/ffffff',
              title: item.title,
              desc: item.digest,
              url: item.link
            }

        })

        
      }
    )
  },
  components: {
    ViewBox,
    XHeader,
    Tabbar,
    TabbarItem,
    Tab, 
    TabItem,
    Marquee,
    MarqueeItem,
    Swiper,
    Panel,
    Icon
  },
  methods: {
    // 下拉刷新
    refresh(){
      // console.log(1)
      console.log(getNum())
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html', {
        miss: '00',
        refresh: getNum()
      }).then( data => {
        console.log(data)
          // 首屏列表
        this.datalist = data.list.filter(item => {
          return item.picInfo.length != 0
        }).map(item => {
          return {
            src: item.picInfo[0].url,
              title: item.title,
              desc: item.digest,
              url: item.link
            }
        })
          
        
        console.log(this.$refs.scrollRef)
        this.$refs.scrollRef.finishPullToRefresh();
        this.$vux.toast.show({
          text: `已刷新${this.datalist.length}条内容`
        })

      })
    },
    // 上拉加载
    infinite(){
      // console.log(2)

      

      // 上拉加载更多新闻
      // http://3g.163.com/touch/jsonp/sy/recommend/'+start+'-'+end+'.html'
      this.$jsonp(`http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html`, {
        miss: '00',
        refresh: getNum()
      })
      .then( data => {
        console.log(data)

        // console.log(22)

        // 首屏列表
        var Datalist = data.list.filter(item => {
          return item.picInfo.length != 0
        }).map(item => {
            return {
              src: item.picInfo[0].url,
              fallbackSrc: 'http://placeholder.qiniudn.com/60x60/ee1a1a/ffffff',
              title: item.title,
              desc: item.digest,
              url: item.link
            }

        })

        start += 10
        end = start + 10

        this.moreDatalist = this.moreDatalist.concat(Datalist)
        this.$refs.scrollRef.finishInfinite();


      })


    }
  }
};
</script>

<style lang="less">
@import "~vux/src/styles/reset.less";
#app {
  height: 100%;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  .header {
    position: fixed;
    left: 0;
    width: 100%;
    top: 0;
    z-index: 9;
  }
  .scroller {
    top: 90px;
  }
  .loading-layer {
    padding-bottom: 90px;
  }
  .tabbar {
    position: fixed;
    left: 0;
    width: 100%;
    bottom: 0;
  }
  .weui-media-box_appmsg .weui-media-box__hd {
    width: 105px;
  }
  .weui-media-box_appmsg .weui-media-box__thumb {
    width: 105px;
  }
}
.tab .vux-tab-item.vux-tab-selected {
    color: #ee1a1a;
    border-bottom: 3px solid #ee1a1a
}
html,
body {
  margin: 0;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
}
</style>
