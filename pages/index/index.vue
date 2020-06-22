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
      
      <view class="operate-icons">
        <u-icon @click="playMode" :name="modes[curModel]" color="#fbf8f8" size="50"></u-icon>
        <u-icon @click="last" name="skip-back-left" color="#fff" size="65"></u-icon>
        <u-icon @click="playOrPause" :name="[pause ? 'play-circle-fill' : 'pause-circle-fill']" color="#fff" size="130"></u-icon>
        <!-- <u-icon name="play-circle-fill"></u-icon> -->
        <u-icon @click="next" name="skip-forward-right" color="#fff" size="65"></u-icon>
        <u-icon @click="toggleList" name="list-dot" color="#fbf8f8" size="50"></u-icon>
      </view>
    </view>
    
    <u-popup v-model="showListFlag" mode="bottom" border-radius="14" length="60%" closeable="true">
    	<view class="music-lists">
        <view class="item content-color u-border-top u-boder-bottom" v-for="(item, index) in lists" :key="index" @click="playItem(index)">
           <view class="name">{{index+1 +'. '+ item.name}}</view>
           <view v-if="item.singer" class="singer">- {{item.singer}}</view>
        </view>
      </view>
    </u-popup>
  </view>
</template>

<script>
  const bgAudioMannager = uni.getBackgroundAudioManager()
  bgAudioMannager.coverImgUrl = ''
  
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
			}
		},
		onLoad() {
      uni.request({
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
        this.pause = false
        
        var item = this.lists[index]
        bgAudioMannager.title = item.title
        bgAudioMannager.singer = item.singer
        bgAudioMannager.src = item.src
        bgAudioMannager.play()
        
        uni.setNavigationBarTitle({
            title: item.title
        });
        
        bgAudioMannager.onEnded(() => {
          this.play(this.getNext())
        })
      },
      playItem(index) {
        this.lastIndex = this.curIndex
        this.curIndex = index
        
        this.play(index)
      },
      last() {
        this.pause = true
        bgAudioMannager.stop()
        this.play(this.lastIndex)
      },
      next() {
        this.pause = true
        bgAudioMannager.stop()
        this.play(this.getNext())
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
        
        this.lastIndex = this.curIndex
        this.curIndex = next
        
        return this.curIndex
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
  width: 540rpx;
  height: 540rpx;
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

.operate-icons{
  margin: 80rpx 0 60rpx 0;
  display: flex;
  u-icon {
    margin: 20rpx;
    padding: 10rpx;
  }
}

.music-lists {
  padding: 40rpx;
  margin-top: 30rpx;
  .item {
    font-size: 32rpx;
    padding: 20rpx 0;
    display: flex;
    align-items: center;
    .singer {
      margin-left: 20rpx;
      font-size: 24rpx;
    }
  }
}
</style>
