<template>
    <div class='detail'>
    <!-- 头部导航 -->
     <div class='top'>
       <span @click='back' class="iconfont icon--fanhui"></span>
       <span>{{title}}</span>
     </div>
     <!-- 头像 -->
     <div class='avator'
     :style="{'background-image':`url(${avator})`}"
     ref='img'
     >
       <div class='random' ref="random" @click="randomPlay()">
         <span class="iconfont icon-play-circle"></span>
         <span>随机播放全部</span>
       </div>
     </div>
     <!-- 歌曲列表 -->
     <div class='songlist' ref='wrapper'>
        <div class='content'>
          <ul>
            <li v-for='(item,index) in list'
              :key='index'
              @click="openPlay(index)"
              >
               <h2>{{item.data.songname}}</h2>
               <p>{{item.data.singer[0].name}}.{{item.data.albumname}}</p>
            </li>
          </ul>
          <div style="height:50px"></div>
        </div>
     </div>
  </div>
</template>

<script>
import axios from 'axios'
import BS from 'better-scroll'
import { mapMutations } from 'vuex'
import { Indicator } from 'mint-ui'
export default {
  data () {
    return {
      list: [],
      title: '',
      avator: '',
      songlist: []
    }
  },
  methods: {
    ...mapMutations(['addSongList', 'changeCurrendIndex', 'changeScreen', 'assignLoop']),
    back () {
      this.$router.go(-1)
    },
    randomPlay () {
      const random = parseInt(Math.random() * this.songlist.length)
      this.openPlay(random)
      this.assignLoop(2)
    },
    openPlay (index) {
      // 点击歌的li 显示播放器
      this.addSongList(this.songlist)
      // 确定点击的是那首歌
      this.changeCurrendIndex(index)
      // 点击屏幕变大
      this.changeScreen(true)
    }
  },
  mounted () {
    const img = this.$refs.img
    const random = this.$refs.random
    // console.log(this.$refs.random)
    const imgH = img.clientHeight
    const wrapper = this.$refs.wrapper
    this.bs = new BS(wrapper, { probeType: 3, click: true })
    this.bs.on('scroll', ({ y }) => {
      // 获取图片的元素和高度
      if (y >= 0) {
        // 下拉放大
        const precent = 1 + (y / imgH)
        img.style.transform = `scale(${precent})`
        img.style.zIndex = 1
      } else {
        // 向上滚动
        // 没有到顶部 滚动层的层级高
        // 到达顶部额时候 图片的层级高
        if (Math.abs(y) > imgH - 40) {
          img.style.zIndex = 1
          img.style.paddingTop = '0%'
          img.style.height = '40px'
          random.style.display = 'none'
        } else {
          img.style.zIndex = -1
          img.style.paddingTop = '70%'
          img.style.height = 0
          random.style.display = 'flex'
        }
      }
    })
  },
  created () {
    Indicator.open({
      text: '加载中...',
      spinnerType: 'fading-circle'
    })
    // console.log(this.$route.params)
    const { topid } = this.$route.params
    const url = '/aaa/fcj/music/rankSongList'
    axios.post(url, { topid })
      .then(res => {
        // console.log(res.data.songlist)
        // console.log(res.data)
        this.list = res.data.songlist
        this.title = res.data.topinfo.ListName
        this.avator = `https://y.gtimg.cn/music/photo_new/T002R300x300M000${res.data.songlist[0].data.albummid}.jpg?max_age=2592000`

        // 处理数据
        const mids = []
        res.data.songlist.map(item => {
          const obj = {}
          obj.albummid = item.data.albummid
          obj.albumname = item.data.albumname
          obj.singer = item.data.singer
          obj.songmid = item.data.songmid
          obj.songname = item.data.songname
          obj.albumUrl = `https://y.gtimg.cn/music/photo_new/T002R300x300M000${item.data.albummid}.jpg?max_age=2592000`
          this.songlist.push(obj)
          mids.push(item.data.songmid)
        })
        // console.log(this.songlist)
        // console.log(mids)
        let urls = []
        axios.post('/aaa/fcj/music/songurl', { mids })
          .then(res => {
            Indicator.close()
            // console.log(res)
            urls = res.data.urls
            // console.log(urls)
            const finalData = []
            for (let index = 0; index < this.songlist.length; index++) {
              this.songlist[index].audioUrl = urls[index]
              if (urls[index]) {
                // 将不能播放的歌曲去除
                finalData.push(this.songlist[index])
              }
            }
            // console.log(finalData)
            this.songlist = finalData
          })
      })
  }
}
</script>

<style lang="less" scoped>
@import '../../style/index.less';
.detail{
  position: fixed;
  top: 0px;
  bottom: 0px;
  background: @white;
  left:0px;
  right: 0px;
  // 头部
  .top{
    height: 40px;
    .w(375);
    // color: @green;
    background: @green;
    position: absolute;
    top:0;
    z-index: 3;
    display: flex;
    justify-content: center;
    .icon--fanhui{
      font-size: 22px;
      color: @white;
      position: absolute;
      left: 20px;
      top: 6px;
    }
    span:last-of-type{
      font-size: 20px;
      display: inline-block;
      margin-top: 5px;
      color: @white;
    }
  }
  // 头像
  .avator{
    .w(375);
    padding-top: 70%;
    background-size:cover;
    position: absolute;
    top: 0px;
    z-index: -1;
    .random{
      position: absolute;
      top: 206px;
      left: 122px;
      border-radius: 30px;
      background-color: @green;
      .w(135);
      height: 32px;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #fff;
      .icon-play-circle{
        font-size: 18px;
      }
      span:nth-of-type(2){
        font-size: 14px;
      }
    }
  }
  // 歌曲列表
  .songlist{
   position: fixed;
   background: @white;
  //  overflow: hidden;
   top:270px;
   bottom: 0px;
   left:0;
   right: 0;
   .content{
     padding: 10px 30px;
     background: @white;
     li{
       font-size: @fs-s;
       color: #000;
       height: 61px;
       h2{
        height: 20px;
        font-size: 18px;
        line-height: 20px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
       }
       p{
        margin-top:5px;
        height:20px ;
        color: @gray;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
       }
     }
   }
  }
}
</style>
