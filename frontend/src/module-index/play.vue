<template>
  <div class="learingIndexPlay">
    <div class="vidPlayBox" @click="playVid">
      <video id="courseVid" :src="videoUrl"  width="100%" autoplay="autoplay" x5-playsinline="" playsinline="" webkit-playsinline="" poster="" preload="auto">your browser does not support the video tag</video>
      <!--<i v-if="backSub" @click.stop="goBack()" class="icon-plab-back"></i>-->
    </div>
    <div class="title"><span>章节</span></div>
    <!--课程大纲-->
    <div class="courseItem">
      <div class="tit">课程大纲</div>
      <ul v-for="(item, index) in courseItem" :key="index">{{item.name}}
        <li v-for="(it, ind) in item.lessions" :key="ind" :class="{act: it.id == select}" @click="checkPlayVid(it.id, it.free_trial)"><span v-show="it.is_video">视频</span>{{it.name}} <a v-if="it.free_trial" v-show="it" >试学</a></li>
      </ul>
    </div>
  </div>
</template>
<script>
  import learingFooter from './../components/footer.vue'
  import IndexApi from '../api/learingInd.js'
  import cfg from './../utils/config'
  import { MessageBox } from 'mint-ui';

  export default {
    name: 'learingIndexPlay',
    data () {
      return {
        select: 1,       // 选择要播放的视频
        videoUrl: 'https://stream7.iqilu.com/10339/upload_transcode/202002/18/20200218114723HDu3hhxqIT.mp4',    // 视频连接
        courseItem: [
          {
            id: '1',
            name: 'aaa',
            lessions: 'alsdkjf'
          }
        ],  // 课程大纲
        imgBaseUrl: cfg.imgBaseUrl,
        isBuy: 0,       // 是否购买课程
        backSub:true,
        playParam:{
          course_id:'',   //课程Id
          chapter_id:'',  //播放章Id
          lession_id: 1,  // 播放小节Id
          time:'0'        // 当前播放时间
        },
      }
    },
    methods:{
      init: function(){
        let _this = this
        this.playParam.course_id = this.$route.params.classId
        this.getCourseItem(this.playParam.course_id)
      },
      // 课程大纲
      getCourseItem: function(obj){
        // 从课程详情获取是否购买信息
        IndexApi.courses({id:obj},(ret, err) => {
          if (err) {
            console.log(err)
          }else{
            this.isBuy = ret.data.is_buy
          }
        })
        // 获取课程大纲
        IndexApi.coursesutline({id:obj},(ret, err) => {
          if (err) {
            console.log(err)
          }else{
            let _this = this
            this.courseItem = ret.data.chapters
            this.playParam = ret.data.progress
            // 播放的初始设置
            let myVideo = document.getElementById('courseVid')
            myVideo.currentTime = this.playParam.time;
            myVideo.addEventListener("timeupdate", function(){
              _this.playParam.time = myVideo.currentTime
              _this.playStart()
            });

            this.checkPlayVid(this.playParam.lession_id, true)
          }
        })
      },
      // 选择播放课程
      checkPlayVid: function(obj, play){
        let _this = this
        this.select = obj
        if(this.isBuy == 1 || play){
          this.courseItem.map(function(n, i){
            n.lessions.map(function(h, j){
              if (h.id == obj){
                _this.playParam.chapter_id = n.id
                _this.playParam.lession_id = h.id
                document.getElementById("courseVid").src = _this.imgBaseUrl + h.video_address;
                return
              }
            })
          })
        }else{
          // 弹框 去支付
          MessageBox({
            title: '您还没有购买本课程',
            message: '前往购买本课程？',
            showCancelButton: true,
          }).then(action => {
            if(action == 'confirm'){
              _this.$router.push('/pay/'+obj)
            }
          });
        }
      },
      // 视频播放
      playVid: function(){
        let myVideo = document.getElementById('courseVid')
        if (myVideo.paused){
          myVideo.play();
          this.backSub = false
        }else{
          myVideo.pause();
          this.backSub = true
        }
      },
      // 播放状态提交
      playStart: function(){
        IndexApi.PlayStart(this.playParam, (ret, err) => {
          if (err) {
            console.log(err)
          }else{

          }
        })
      },
      //
      goBack: function(){
        this.$router.go(-1)
      }
    },
    mounted: function(){
      this.init()
    },
    components: {
      learingFooter
    }
  }
</script>

<style lang="scss">
  @import "../assets/baseScss";
  .learingIndexPlay{
    text-align: center;
    .vidPlayBox{
      position: relative;
      i{
        position: absolute;
        left: 20px;
        top:20px;
        font-size: 30px;
        color:rgba(255, 255, 255, 0.6)
      };
    }
    .title{
      box-shadow: 0px 3px 5px $cl5;
      background: $cl1;
      text-align: left;
      height: 40px;
      line-height: 40px;
      padding-left: 30px;
      font-size: 18px;
      position: relative;
      top:-7px;
      span{
        display: inline-block;
        height: 38px;
        padding: 0 10px;
        color:$cl0;
        border-bottom: solid 2px $cl0;
      }
    }
    .courseItem{
      padding:10px;
      margin: 10px 0;
      font-size: 12px;
      line-height: 28px;
      color:$cl6;
      text-align: left;
      .tit{
        font-size: 14px;
        color: $cl3;
      }
      .act{
        color:$cl0;
        span{
          border-color: $cl0;
        }
      }
      ul{
        color:$cl3;
        padding-left: 10px;
        li{
          color:$cl6;
          font-size: 12px;
          padding-left: 20px;
          span{
            font-size: 10px;
            padding:0px 2px;
            margin-right: 3px;
            border:solid 1px $cl6
          }
          a{
            position: absolute;
            right: 20px;
            color:$cl0;
          }
        }
      }
      ul:before{
        content: ' ● ';
        font-size: 20px;
        color:$cl4;
        position: relative;
        top:2px;
      }
      li:before{
        content: ' ';
        display: inline-block;
        position: absolute;
        left: 25px;
        color:$cl7;
        width: 18px;
        height: 30px;
        border-left: solid 1px $cl4;
      }
    }
  }
</style>
