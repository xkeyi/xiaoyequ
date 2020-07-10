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
      
      <view class="music-title">
        <view class="name">{{curItem.name}}</view>
        <view class="singer">{{curItem.singer}}</view>
      </view>
      
      <view class="music-progress">
        <view class="time">{{nowtime}}</view>
        <view class="slider">
          <u-slider v-model="percent" active-color="#ff8b9f" inactive-color="#c7c2c2" @end="sliderEnd()"></u-slider>
          </view>
        <view class="time">{{alltime}}</view>
      </view>
      
      <view class="operate-icons">
        <u-icon class="operate-icon" @click="playMode" :name="modes[curModel]" color="#fbf8f8" size="55"></u-icon>
        <u-icon class="operate-icon" @click="last" name="skip-back-left" color="#fff" size="65"></u-icon>
        <u-icon class="operate-icon" @click="playOrPause" :name="circleName" color="#fff" size="130"></u-icon>
        <u-icon class="operate-icon" @click="next" name="skip-forward-right" color="#fff" size="65"></u-icon>
        <u-icon class="operate-icon" @click="toggleList" name="list-dot" color="#fbf8f8" size="55"></u-icon>
      </view>
    </view>
    
    <u-popup v-model="showListFlag" mode="bottom" border-radius="14" length="75%" :closeable="popupCloseable" @close="popClose">
      <view class="search">
        <u-search v-model="keyword" shape="round" :clearabled="true" :show-action="false"></u-search>
      </view>
      
    	<scroll-view class="music-lists" scroll-y="true" :scroll-top="scrollTop" @scroll="scroll">
        <view v-if="searchLists.length">
          <view class="item content-color u-border-top u-boder-bottom" v-for="(item, index) in searchLists" :key="index" @click="playItem(item.index)">
             <view class="name" :class="{ active: item.index == curIndex }">{{index+1 +'. '+ item.name}}</view>
             <view v-if="item.singer" class="singer" :class="{ active: item.index == curIndex }">- {{item.singer}}</view>
          </view>
        </view>
        <view v-else>
          <view class="item content-color u-border-top u-boder-bottom" v-for="(item, index) in lists" :key="index" @click="playItem(index)">
             <view class="name" :class="{ active: index == curIndex }">{{index+1 +'. '+ item.name}}</view>
             <view v-if="item.singer" class="singer" :class="{ active: index == curIndex }">- {{item.singer}}</view>
          </view>
        </view>
      </scroll-view>
    </u-popup>
  </view>
</template>

<script>
  const bgAudioMannager = uni.getBackgroundAudioManager()
  
  // var musics = require("xiaoyequ.json")

	export default {
		data() {
			return {
        lists: [{
           "name": "小夜曲",
           "singer": "小夜曲",
           "title": "小夜曲",
           "src": "http://cdn.xkeyi.top/xiaoyequ/%E5%B0%8F%E5%A4%9C%E6%9B%B2.mp3"
         }],
        showListFlag: false,
        pause: true, // 暂停
        modes: ['reload', 'checkmark', 'question'], // 顺序、单曲、随机
        curModel: 0, // 当前模式
        curIndex: 0, // 当前序号
        lastIndex: 0, // 前一首序号
        curTime: 0, // 当前播放时间
        duration: 0, // 音频总时长
        percent: 0, // 当前播放进度
        popupCloseable: true,
        scrollTop: 0,
        oldScrollTop: 0,
        keyword: '',
			}
		},
    computed: {
      circleName() {
        if (this.pause) {
          return 'play-circle-fill'
        }
        
        return 'pause-circle-fill'
      },
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
      	
      	return i + ':' + s
      },
      curItem() {
        return this.lists[this.curIndex]
      },
      searchLists() {
        if (!this.showListFlag || this.keyword.length == 0) {
          return []
        }
        
        var res = [];
        this.lists.forEach((item, index) => {
          if (item.title.indexOf(this.keyword) != -1) {
            var sitem = item
            sitem.index = index
            res.push(sitem)
          }
        })
        
        return res
      }
    },
		async onLoad() {
      // this.lists = musics
      await uni.request({
        url: 'http://cdn.xkeyi.top/xiaoyequ/xiaoyequ.json',
        success: (res) => {
          this.lists = res.data
        },
        fail: (res) => {
          console.log(res)
        }
      })
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
        if (mode > 2) {
          this.curModel = 0
        } else {
          this.curModel = mode
        }
      },
      toggleList() {
        this.showListFlag = !this.showListFlag
        this.goTop()
      },
      scroll(e) {
        this.oldScrollTop = e.detail.scrollTop
      },
      goTop() {
        this.scrollTop = this.oldScrollTop
        this.$nextTick(() => {
          this.scrollTop = this.curIndex * 45
        })
      },  
      popClose() {
        this.keyword = ''
      },
      playItem(index) {
        this.lastIndex = this.curIndex
        this.curIndex = index
        
        this.initPlay()
      },
      last() {
        this.pause = true
        
        this.curIndex = this.lastIndex
        
        this.initPlay()
      },
      next() {
        this.pause = true
        
        this.lastIndex = this.curIndex
        this.curIndex = this.getNext()

        this.initPlay()
      },
      getNext() {
        
        if (this.curModel == 0) { // 顺序播放
         var next = this.curIndex + 1
        } else if (this.curModel == 1) { // 单曲循环
          var next = this.curIndex
        } else { // 随机播放
          var next = parseInt(Math.random()*this.lists.length)
        }
        
        if (next >= this.lists.length) {
          next = this.getNext()
        }
        
        return next
      },
      initPlay() {
        var that = this
        var item = this.curItem

        uni.setNavigationBarTitle({
          title: item.title
        })
        
        bgAudioMannager.title = item.title
        bgAudioMannager.singer = item.singer
        bgAudioMannager.src = item.src
        bgAudioMannager.play()
        
        bgAudioMannager.onPlay(() => {
        	that.pause = false
        })
        bgAudioMannager.onPause(() => {
        	that.pause = true
        })
        bgAudioMannager.onEnded(() => {
          if (!that.pause) {
            that.next()
          }
        })
        bgAudioMannager.onTimeUpdate(() => {
          that.curTime = Math.floor(bgAudioMannager.currentTime)
          that.duration = Math.floor(bgAudioMannager.duration)
        })
        bgAudioMannager.onPrev(() => {
        	that.last()
        })
        bgAudioMannager.onNext(() => {
        	that.next()
        })
        
        bgAudioMannager.onError((error) => {
          //
        })
        bgAudioMannager.onWaiting(function() {
          //
        })
        
        bgAudioMannager.onCanplay(function() {
          // that.duration = Math.floor(bgAudioMannager.duration)
        })
      },
      sliderEnd() {
        bgAudioMannager.pause()
        
        var seek = this.percent / 100 * this.duration
        bgAudioMannager.seek(seek)
        
        bgAudioMannager.play()
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

.music-title {
  margin-top: 60rpx;
  width: 88%;
  text-align: left;
  .name {
    font-size: 38rpx;
    color: #fff;
    font-weight: bold;
  }
  .singer {
    margin-top: 10rpx;
    font-size: 26rpx;
    color: #c7c2c2;
  }
}

.music-progress {
  width: 90%;
  margin-top: 40rpx;
  display: flex;
  .time {
    color: #c7c2c2;
    flex: 0 0 80rpx;
    margin-top: -15rpx;
  }
  .slider {
    flex: 1;
    margin: 0 20rpx;
  }
}

.operate-icons{
  width: 90%;
  margin: 0rpx 0 60rpx 0;
  display: flex;
  justify-content: space-between;
  .operate-icon {
    flex: 1;
    justify-content: center;
  }
}

.search {
  width: 80%;
  margin-top: 20rpx;
  margin-left: 40rpx;
}
.music-lists {
  height: 90%;
  padding: 20rpx 40rpx 40rpx 40rpx;
  .item {
    font-size: 32rpx;
    padding: 20rpx 0;
    display: flex;
    align-items: center;
    .singer {
      margin-left: 20rpx;
      font-size: 26rpx;
    }
    .active {
      color: #49B764;
    }
  }
}
</style>
