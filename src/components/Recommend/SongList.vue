<template>
  <div>
    <div class="title">热门歌单推荐</div>
    <ul class="list">
      <li v-for="(item,index) in songList" :key="index" @click="goDateil(item.dissid)">
        <div class="left">
          <!-- 图片懒加载 -->
          <img v-lazy="item.imgurl" alt />
        </div>
        <div class="right">
          <p class="name">{{item.creator.name}}</p>
          <p class="album">{{item.dissname}}</p>
        </div>
      </li>
    </ul>
    <div style="height:50px"></div>
  </div>
</template>

<script>
import axios from 'axios'
import { Indicator } from 'mint-ui'

export default {
  data () {
    return {
      songList: []
    }
  },
  mounted () {
    Indicator.open({
      text: '加载中...',
      spinnerType: 'fading-circle'
    })
    axios.get('/api/music/api/getDiscList?g_tk=1928093487&inCharset=utf-8&outCharset=utf-8&notice=0&format=json&platform=yqq&hostUin=0&sin=0&ein=29&sortId=5&needNewCode=0&categoryId=10000000&rnd=0.3983624931217431')
      .then(res => {
        Indicator.close()
        // console.log(res.data.data.list)
        this.songList = res.data.data.list
      })
  },
  methods: {
    goDateil (dissid) {
      this.$router.push(`/recommend/${dissid}`)
    }
  }
}
</script>

<style lang="less" scoped>
@import "../../style/index.less";
.title {
  .w(375);
  height: 65px;
  text-align: center;
  line-height: 65px;
  font-size: @fs-s;
  color: @green;
}
.list {
  li {
    .w(375);
    height: 82px;
    box-sizing: border-box;
    padding: 0px 20px 20px 20px;
    display: flex;
    .left {
      width: 82px;
      height: 60px;
      img {
        width: 60px;
        height: 60px;
      }
    }
    .right {
      color: #000;
      font-size: @fs-s;
      .name{
        padding-top: 5px;
      }
      .album{
        color: @gray;
        margin-top: 10px;
      }
    }
  }
}
</style>
