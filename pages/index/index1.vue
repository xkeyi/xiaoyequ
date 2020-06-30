<template>
  <view class="app-bg">
    <view class="content">
      <view class="music-logo">
        <view class="logo-border">
          <view class="logo-bg">
            <view class="animation" :class="{pause: pause}">
              <image src="../../static/music.png" ></image>
            </view>
          </view>
        </view>
      </view>
      
      <view class="music-progress">
        <view class="time">{{nowtime}}</view>
        <view class="slider">
          <u-slider v-model="percent" active-color="#ff8b9f" inactive-color="#c7c2c2" @end="sliderEnd()"></u-slider>
          </view>
        <view class="time">{{alltime}}</view>
      </view>
      
      <!-- <view class="music-time">
        <view class="time">{{nowtime}}</view>
        <view class="time">{{alltime}}</view>
      </view> -->
      <view class="operate-icons">
        <u-icon class="operate-icon" @click="playMode" :name="modes[curModel]" color="#fbf8f8" size="60"></u-icon>
        <u-icon class="operate-icon" @click="last" name="skip-back-left" color="#fff" size="65"></u-icon>
        <u-icon class="operate-icon" @click="playOrPause" :name="[pause ? 'play-circle-fill' : 'pause-circle-fill']" color="#fff" size="130"></u-icon>
        <!-- <u-icon name="play-circle-fill"></u-icon> -->
        <u-icon class="operate-icon" @click="next" name="skip-forward-right" color="#fff" size="65"></u-icon>
        <u-icon class="operate-icon" @click="toggleList" name="list-dot" color="#fbf8f8" size="60"></u-icon>
      </view>
    </view>
    
    <u-popup v-model="showListFlag" mode="bottom" border-radius="14" length="75%" closeable="true">
    	<scroll-view class="music-lists" scroll-y="true">
        <view class="item content-color u-border-top u-boder-bottom" v-for="(item, index) in lists" :key="index" @click="play(index)">
           <view class="name">{{index+1 +'. '+ item.name}}</view>
           <view v-if="item.singer" class="singer">- {{item.singer}}</view>
        </view>
      </scroll-view>
    </u-popup>
  </view>
</template>

<script>
  const bgAudioMannager = uni.getBackgroundAudioManager()
  
  var musics = require("xiaoyequ.json")

	export default {
		data() {
			return {
        lists: [],
        showListFlag: false,
        pause: true, // 暂停
        modes: ['reload', 'checkbox-mark'],
        curModel: 0, // 当前模式
        curIndex: 0, // 当前序号
        lastIndex: 0, // 前一首序号
        curTime: 0, // 当前播放时间
        duration: 0, // 音频总时长
        percent: 0, // 当前播放进度
			}
		},
    computed: {
      nowtime() {
      	//计算分钟：将秒数除以60，然后下舍入，既得到分钟数
      	var i = Math.floor(this.curTime / 60)
      	//计算秒：取得秒%60的余数，既得到秒数
      	var s = this.curTime % 60
      	//将变量转换为字符串
      	i += ''
      	s += ''
        
      	//如果只有一位数，前面增加一个0
      	i = (i.length == 1) ? '0' + i : i
      	s = (s.length == 1) ? '0' + s : s
        
        // 计算当前播放进度
        var that = this
        if (this.duration == 0) {
          that.percent = 0
        } else {
          that.percent = Math.floor(this.curTime / this.duration * 100)
        }

      	return i + ':' + s
      },
      alltime() {
      	//计算分钟：将秒数除以60，然后下舍入，既得到分钟数
      	var i = Math.floor(this.duration / 60)
      	//计算秒：取得秒%60的余数，既得到秒数
      	var s = this.duration % 60
      	//将变量转换为字符串
      	i += ''
      	s += ''
      	
      	//如果只有一位数，前面增加一个0
      	i = (i.length == 1) ? '0' + i : i
      	s = (s.length == 1) ? '0' + s : s
      	console.log('alltime:'+i)
      	return i + ':' + s
      },
      // percent() {
      //   if (this.duration == 0) {
      //     return 10
      //   }
              
      //   return Math.floor(this.curTime / this.duration * 100)
      // }
      // percent: {
      //   get: function () {
      //     if (this.duration == 0) {
      //       return 10
      //     }
              
      //     return Math.floor(this.curTime / this.duration * 100)
      //   },
      //   set: function (newValue) {
      //     console.log('set:'+newValue)
      //   }
      // }
    },
		onLoad() {
      this.lists = musics
      // uni.request({
      //     url: 'http://cdn.xkeyi.top/xiaoyequ/xiaoyequ.json',
      //     success: (res) => {
      //       this.lists = res.data
      //     },
      //     fail: (res) => {
      //       console.log(res)
      //     }
      // })
		},
		methods: {
      playOrPause() {        
        if (this.pause) {
          bgAudioMannager.play()
        } else {
          bgAudioMannager.pause()
        }
        this.pause = !this.pause
      },
      playMode() {
        var mode = this.curModel + 1
        if (mode > 1) {
          this.curModel = 0
        } else {
          this.curModel = mode
        }
      },
      toggleList() {
        this.showListFlag = !this.showListFlag
      },
      play(index) {
        this.lastIndex = this.curIndex
        this.curIndex = index
        
        this.initPlay()
      },
      last() {
        this.pause = true
        
        this.duration = 0
        this.curTime = 0
        
        this.curIndex = this.lastIndex
        
        this.initPlay()
      },
      next() {
        this.pause = true
        
        this.duration = 0
        this.curTime = 0
        
        this.lastIndex = this.curIndex
        this.curIndex = this.getNext()

        this.initPlay()
      },
      getNext() {
        // 单曲循环
        if (this.curModel == 1) { 
         return this.curIndex
        }
        
        // parseInt(Math.random()*(上限-下限+1)+下限); 
        var next = parseInt(Math.random()*this.lists.length)
        if (next >= this.lists.length) {
          next = this.getNext()
        }
        
        return next
      },
      initPlay() {
        var that = this
        
        var item = that.lists[that.curIndex]

        uni.setNavigationBarTitle({
          title: item.title
        })
        
        bgAudioMannager.title = item.title
        bgAudioMannager.singer = item.singer
        bgAudioMannager.src = item.src
        // bgAudioMannager.play()
        
        bgAudioMannager.onPlay(() => {
        	that.pause = false
        })
        bgAudioMannager.onPause(() => {
        	that.pause = true
        })
        bgAudioMannager.onEnded(() => {
        	that.next()
        })
        bgAudioMannager.onTimeUpdate(() => {
          that.curTime = Math.floor(bgAudioMannager.currentTime)
        })
        bgAudioMannager.onPrev(() => {
        	that.last()
        })
        bgAudioMannager.onNext(() => {
        	that.next()
        })
        
        bgAudioMannager.onError((error) => {
          that.pause = true
        	console.log(error)
        })
        bgAudioMannager.onWaiting(function() {
        	that.pause = true
        })
        
        bgAudioMannager.onCanplay(function() {
          that.duration = Math.floor(bgAudioMannager.duration)
        })
      },
      sliderEnd() {
        // this.pause = true
        
        var seek = this.percent / 100 * this.duration
        bgAudioMannager.seek(seek)
        
        // this.pause = false
      }
		}
	}
</script>

<style lang="scss">
 .animation {
  width: 100%;
  height: 100%;
	animation:my-rotate 5s linear infinite;
 }
.pause{
  animation-play-state:paused;
}
@keyframes my-rotate{
  0% { transform: rotate(0) }
  25%{transform:rotate(90deg)}
  50%{transform:rotate(180deg)}
  75%{transform:rotate(270deg)}
  100% {transform: rotate(360deg)}
}

.music-logo {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 480rpx;
  height: 480rpx;
  border-radius: 50%;
  // border: #606266 30rpx solid;
  padding: 30rpx;
  background: rgba(255,255,255,0.4);
  .logo-border {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    border-radius: 50%;
    // border: #000 75rpx solid;
    padding: 75rpx;
    background: rgba(0,0,0,0.6);
    .logo-bg {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 100%;
      background: url('http://cdn.xkeyi.top/xiaoyequ/bg2.jpg') no-repeat;
      background-size: 100% 100%;
      // background: ;
      opacity: 1;
      border-radius: 50%;
      image {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            opacity: 1;
            transform:rotate(360deg);
            transition: all 2s;

            // padding: 40rpx;
            // background: rgba(255,255,255,0.5);
            // background-color: #fff;
          }
      
    }
  }
}

.music-progress {
  width: 90%;
  margin-top: 80rpx;
  display: flex;
  .time {
    color: #c7c2c2;
    flex: 0 0 80rpx;
    margin-top: -20rpx;
  }
  .slider {
    flex: 1;
    margin: 0 15rpx;
  }
}

.music-time {
  width: 80%;
  margin-top: 10rpx;
  display: flex;
  justify-content: space-between;
  color: #c7c2c2;
}

.operate-icons{
  margin: -40rpx 0 60rpx 0;
  display: flex;
  .operate-icon {
    flex: 1;
    margin: 20rpx;
    padding: 10rpx;
  }
}

.music-lists {
  height: 90%;
  padding: 40rpx;
  margin-top: 30rpx;
  .item {
    font-size: 32rpx;
    padding: 20rpx 0;
    display: flex;
    align-items: center;
    .singer {
      margin-left: 20rpx;
      font-size: 26rpx;
    }
  }
}
</style>
