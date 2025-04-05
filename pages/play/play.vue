<template>
  <view class="play">
    <view class="play-box">
      <video 
        class="player" 
        :autoplay="true" 
        :src="url" 
        :initial-time="initialtime" 
        @timeupdate="videoTimeUpdateEvent"
<<<<<<< HEAD
        @fullscreenchange="handleFullScreenChange"
=======
>>>>>>> b5aee18bd3b0a27c522816e7cb1810ddcdd1a06e
        :controls="true"
        :show-center-play-btn="true"
        :show-fullscreen-btn="true"
        :show-play-btn="true"
        :show-progress="true"
        :enable-progress-gesture="true"
        :object-fit="'contain'"
        :direction="0"
        :show-loading="true"
        :enable-play-gesture="true"
        :auto-pause-if-navigate="true"
        :auto-pause-if-open-native="true"
        :vslide-gesture="true"
        :vslide-gesture-in-fullscreen="true"
        :danmu-list="[]"
        :enable-danmu="false"
        :page-gesture="true"
        :show-screen-lock-button="true"
        :show-snapshot-button="true"
        :show-background-playback-button="true"
        :background-poster="detail.pic"
        :poster="detail.pic"
        :title="name"
        :play-btn-position="'center'"
        :custom-cache="true"
        :enable-auto-rotation="true"
        :show-mute-btn="true"
        :show-volume-slider="true"
        :show-casting-button="true"
<<<<<<< HEAD
        :playback-rate="playbackRate"
=======
>>>>>>> b5aee18bd3b0a27c522816e7cb1810ddcdd1a06e
      ></video>
    </view>
    <view class="icon-box">
      <view v-if="playList.length > 1" class="episode-controls">
        <u-button @click="prevEpisode()" style="margin-right: 20rpx; background-color: #6dd143; color: white; height: 60rpx; line-height: 60rpx; font-size: 32rpx">上一集</u-button>
        <u-button @click="nextEpisode()" style="background-color: #6dd143; color: white; height: 60rpx; line-height: 60rpx; font-size: 32rpx">下一集</u-button>
      </view>
      <u-dropdown>
        <u-icon name="speed" size="60" color="#6dd143"></u-icon>
        <u-dropdown-item slot="content">
          <u-radio-group v-model="playbackRate" :wrap="true">
            <u-radio 
              v-for="(rate, index) in playbackRates" 
              :key="index" 
              :name="rate" 
              :label="rate + 'x'"
              @change="changePlaybackRate"
            ></u-radio>
          </u-radio-group>
        </u-dropdown-item>
      </u-dropdown>
      <u-icon v-if="!starShow" name="star" size="60" @click="addStar()" style="margin-left: 30rpx"></u-icon>
      <u-icon v-if="starShow" name="star-fill" color="#ff4445" size="60" @click="removeStar()" style="margin-left: 30rpx"></u-icon>
      <u-icon name="play-circle" size="70" color="#6dd143" @click="selectPlay()" style="margin-left: 30rpx" v-if="playList.length > 0"></u-icon>
    </view>
    <view class="box-info">
      <view class="name-box">
        <text class="name">{{ detail.name }}</text>
      </view>
      <view class="info-box">
        <text>{{ detail.area }}</text>
        <text class="gap">|</text>
        <text>{{ detail.lang }}</text>
        <text class="gap">|</text>
        <text>{{ detail.type }}</text>
        <text class="gap">|</text>
        <text>{{ detail.year }}</text>
        <text class="gap">|</text>
        <text>{{ detail.note }}</text>
      </view>
      <view class="info-box">
        <text>导演: {{ detail.director }}</text>
      </view>
      <view class="info-box">
        <text>演员: {{ detail.actor }}</text>
      </view>
      <view class="info-box">
        <text>简介: {{ detail.des }}</text>
      </view>
    </view>
    <u-select
      v-model="playShow"
      :list="playList"
      confirm-text="播放"
      @confirm="playConfirm"
    ></u-select>
    <u-toast ref="uToast" />
  </view>
</template>

<script>
import db from "../../utils/database.js";
import http from "../../utils/request.js";
export default {
  data() {
    return {
      siteKey: "",
      id: "",
      name: "",
      url: "",
      initialtime: 0,
      playShow: false,
      playList: [],
      detail: {},
      starShow: true,
      videoUpdateCounter: 0,
      isFullScreen: false,
      playbackRate: 1,
      playbackRates: [0.5, 1.0, 1.5, 2.0],
      currentEpisodeIndex: 0
    };
  },
  methods: {
    async videoTimeUpdateEvent (e) {
      // 播放进度变化时触发timeupdate，event.detail = {currentTime, duration} 。触发频率 250ms 一次
      this.videoUpdateCounter += 1
      if (this.videoUpdateCounter % 40 === 0) {
        // 每10秒更新一下历史记录的时间进度
        this.videoUpdateCounter = 0
        await this.updateHistory(e.detail)
      }
    },
    async updateHistory (detail) {
      const res = await db.get('history', `${this.siteKey}-${this.id}`)
      if (res.flag) {
        res.data.currentTime = detail.currentTime
        res.data.duration = detail.duration
        await db.update('history', res.data)
      }
    },
    selectPlay() {
      this.playShow = !this.playShow;
    },
    playConfirm(e) {
      console.log('播放确认:', e);
      const d = e[0];
      if (!d || !d.value) {
        console.error('无效的播放项:', d);
        this.$refs.uToast.show({ title: '获取播放地址失败', type: 'error', duration: 2300 });
        return;
      }
      
      // 确保value是有效的m3u8地址
      const m3u8Url = d.value.includes('$') ? d.value.split('$').find(url => url.includes('.m3u8')) : d.value;
      if (!m3u8Url || !m3u8Url.startsWith('http')) {
        console.error('无效的m3u8地址:', m3u8Url);
        this.$refs.uToast.show({ title: '无效的播放地址', type: 'error', duration: 2300 });
        return;
      }
      
      this.url = m3u8Url;
      uni.setNavigationBarTitle({ title: d.label });
      // 更新当前集数索引
      this.currentEpisodeIndex = this.playList.findIndex(item => item.value === d.value);
      console.log('当前播放集索引:', this.currentEpisodeIndex, 'URL:', this.url);
    },
    async getDetail(key, id) {
      const res = await http.detail(key, id);
      this.detail = res;
      const arr = [];
      let num = 1
      for (const i of res.m3u8List) {
        const j = i.split('$')
        let label = res.m3u8List.length > 1 ? `第${num}集`: this.detail.name
        if (j.length > 1) {
          for (let m = 0; m < j.length; m++) {
            if (j[m].indexOf('.m3u8') >= 0 && j[m].startsWith('http')) {
              let d = {
                index: i,
                value: j[m],
                label: label,
                extra: {
                  site: key,
                  id: id,
                },
              };
              arr.push(d);
              break
            }
          }
        } else {
          let d = {
            index: i,
            value: j[0],
            label: label,
            extra: {
              site: key,
              id: id,
            },
          };
          arr.push(d);
        }
        num++
      }
      this.playList = arr;
    },
    async checkStar () {
      const res = await db.get('star', `${this.siteKey}-${this.id}`)
      this.starShow = res.flag
    },
    async removeStar () {
      const res = await db.remove('star', `${this.siteKey}-${this.id}`)
      if (res.flag) {
        this.$refs.uToast.show({ title: '移除收藏成功', type: 'success', duration: '2300' })
      } else {
        this.$refs.uToast.show({ title: '移除收藏失败', type: 'warning', duration: '2300' })
      }
      this.checkStar()
    },
    async addStar () {
      let s = {...this.detail}
      s.key = `${this.siteKey}-${this.id}`
      const res = await db.add('star', s)
      if (res.flag) {
        this.$refs.uToast.show({ title: '添加收藏成功', type: 'success', duration: '2300' })
      } else {
        this.$refs.uToast.show({ title: '添加收藏失败', type: 'warning', duration: '2300' })
      }
      this.checkStar()
    },
    handleFullScreenChange(e) {
      this.isFullScreen = e.detail.fullScreen
      if (this.isFullScreen) {
        // 全屏时锁定横屏
        plus.screen.lockOrientation('landscape-primary')
      } else {
        // 退出全屏时恢复竖屏
        plus.screen.lockOrientation('portrait-primary')
      }
    },
    changePlaybackRate() {
      // 倍速切换无需额外处理，video组件会自动响应playbackRate变化
    },
    prevEpisode() {
      if (this.currentEpisodeIndex > 0) {
        this.currentEpisodeIndex--
        this.playConfirm([this.playList[this.currentEpisodeIndex]])
      }
    },
    nextEpisode() {
      if (this.currentEpisodeIndex < this.playList.length - 1) {
        this.currentEpisodeIndex++
        this.playConfirm([this.playList[this.currentEpisodeIndex]])
      }
    }
  },
  onLoad(opt) {
    this.siteKey = opt.site;
    this.id = opt.id;
    this.name = opt.name;
    this.url = opt.url;
    this.initialtime = opt.initialtime;
    this.getDetail(this.siteKey, this.id)
    this.checkStar()
    uni.setNavigationBarTitle({ title: opt.name });
  }
};
</script>

<style lang="scss" scoped>
.play {
  .play-box {
    .player {
      width: 100vw;
      height: 56.25vw; // 16:9 比例
    }
  }
  .icon-box {
    padding: 20px 10%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    .episode-controls {
      display: flex;
      align-items: center;
    }
  }
  .box-info {
    padding: 0 10% 10px;
    .name-box {
      width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      .name {
        font-size: 40rpx;
      }
    }
    .info-box {
      margin-top: 20rpx;
      .gap {
        margin: 0 10rpx;
      }
    }
  }
}
</style>
