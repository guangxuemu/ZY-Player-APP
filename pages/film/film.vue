<template>
  <view class="film">
    <view class="header">
      <u-search
        v-model="search"
        @search="searchEvent"
        @custom="searchEvent"
        :clearabled="true"
        @clear="searchClearEvent"
      ></u-search>
    </view>
    <view class="btns">
      <u-button @click="openSiteSelect" size="mini"
        >网站: {{ site.name }}</u-button
      >
      <u-button @click="openTypeSelect" size="mini"
        >分类: {{ type.name }}</u-button
      >
      <text>共: {{recordcount}} 资源</text>
    </view>
    <view class="body">
      <u-waterfall v-model="flowList" ref="uWaterfall">
        <template v-slot:left="{ leftList }">
          <view
            class="box-warter left-box-warter"
            v-for="(item, index) in leftList"
            :key="index"
            @click="openDetail(item)"
          >
            <u-lazy-load
              border-radius="4"
              :image="item.pic"
              :index="index"
            ></u-lazy-load>
            <view class="box-name">{{ item.name }}</view>
            <view class="box-info">
              <view class="box-class">{{ item.type }}</view>
			  <view class="box-class">{{ item.note }}</view>
              <view class="box-year">{{ item.year }}</view>
            </view>
          </view>
        </template>
        <template v-slot:right="{ rightList }">
          <view
            class="box-warter right-box-warter"
            v-for="(item, index) in rightList"
            :key="index"
            @click="openDetail(item)"
          >
            <u-lazy-load
              threshold="-450"
              border-radius="4"
              :image="item.pic"
              :index="index"
            ></u-lazy-load>
            <view class="box-name">{{ item.name }}</view>
            <view class="box-info">
              <view class="box-class">{{ item.type }}</view>
			  <view class="box-class">{{ item.note }}</view>
              <view class="box-year">{{ item.year }}</view>
            </view>
          </view>
        </template>
      </u-waterfall>
      <u-loadmore
        bg-color="#f8f8f8"
        :status="loadStatus"
        @loadmore="addData"
      ></u-loadmore>
    </view>
    <u-back-top :scroll-top="scrollTop" icon="search"></u-back-top>
    <u-select
      v-model="siteShow"
      :list="siteList"
      value-name="key"
      label-name="name"
      @confirm="siteConfirm"
    ></u-select>
    <u-select
      v-model="typeShow"
      :list="typeList"
      value-name="tid"
      label-name="name"
      @confirm="typeConfirm"
    ></u-select>
    <u-top-tips ref="uTips"></u-top-tips>
    <u-mask :show="mask" @tap.stop>
      <view class="mask">
        <u-loading mode="flower" size="80"></u-loading>
      </view>
    </u-mask>
    <u-toast ref="uToast" />
  </view>
</template>

<script>
import db from "../../utils/database.js";
import http from "../../utils/request.js";
export default {
  data() {
    return {
      search: "",
      site: {},
      siteShow: false,
      siteList: [],
      type: {},
      typeShow: false,
      typeList: [],
      flowList: [],
      loadStatus: "loadmore",
      scrollTop: 0,
      pageCount: 0,
      recordcount: 0,
      mask: false,
      r18KeyWords: ['伦理', '论理', '倫理', '福利', '激情', '理论', '写真', '情色', '美女', '街拍', '赤足', '性感', '里番']
    };
  },
  methods: {
    async searchEvent() {
      if (this.search === '') {
        return false
      }
      this.mask = true
      this.flowList = []
      this.$refs.uWaterfall.clear();
      const res = await http.search(this.site.key, this.search)
      if (res.length >= 1) {
        for (const i of res) {
          const data = await http.detail(this.site.key, i.id)
          this.flowList.push(data);
        }
      }
      this.mask = false
    },
    async searchClearEvent() {
      this.search = ''
      this.mask = true
      this.flowList = []
      this.$refs.uWaterfall.clear();
      await this.getPage(this.type.tid)
      await this.addData(this.site.key, this.pageCount, this.type.tid)
      this.pageCount--
      await this.addData(this.site.key, this.pageCount, this.type.tid)
      this.mask = false
    },
    async openSiteSelect() {
      this.siteShow = true;
      const site = await db.get("site", this.site.key);
    },
    async siteConfirm(e) {
      this.mask = true
      this.flowList = []
      this.$refs.uWaterfall.clear();
      const site = await db.get("site", e[0].value);
      this.site = site.data;
      await this.getPage()
      await this.getClass(this.site.key)
      await this.addData(this.site.key, this.pageCount)
      this.pageCount--
      await this.addData(this.site.key, this.pageCount)
      this.mask = false
    },
    openTypeSelect () {
      this.typeShow = true
    },
    async typeConfirm(e) {
      this.type = {
        name: e[0].label,
        tid: e[0].value
      }
      this.mask = true
      this.flowList = []
      this.$refs.uWaterfall.clear();
      await this.getPage(this.type.tid)
      await this.addData(this.site.key, this.pageCount, this.type.tid)
      this.pageCount--
      await this.addData(this.site.key, this.pageCount, this.type.tid)
      this.mask = false
    },
    openDetail(item) {
      const url = `/pages/detail/detail?site=${this.site.key}&id=${item.id}`;
      this.$u.route({ url: url });
    },
    async getSite() {
      const res = await db.getAll("site");
      if (res.flag) {
        const arr = []
        for (const i of res.data) {
          if (i.isActive) {
            arr.push(i)
          }
        }
        this.siteList = arr
      } else {
        this.$refs.uToast.show({ title: '读取视频源出错', type: 'warning', duration: '2300' })
        return false
      }
      this.site = this.siteList[0]
      await this.getPage()
      await this.getClass(this.site.key)
      await this.addData(this.site.key, this.pageCount)
      this.pageCount--
      await this.addData(this.site.key, this.pageCount)
    },
    async getPage (type) {
      const res = await http.page(this.site.key, type)
      this.pageCount = res.pagecount
      this.recordcount = res.recordcount
    },
    async getClass(key) {
      this.typeList = [{ name: '最新', tid: 0 }]
      this.type = { name: '最新', tid: 0 }
      http.class(key).then(res => {
        // 确保res是数组
        const classData = Array.isArray(res) ? res : [];
        // 屏蔽主分类
        const classToHide = ['电影', '电影片', '电视剧', '连续剧', '综艺', '动漫']
        classData.forEach(ele => {
          if (!classToHide.includes(ele.name)) {
            this.typeList.push(ele)
          }
        })
      }).catch(error => {
        console.error('获取分类失败:', error)
      })
    },
    async addData(key, page, t) {
      console.log(key, page, t, 'lalala')
      const res = await http.list(key, page, t);
      const config = await db.get('setting', 'config')
      for (let i = 0; i < res.length; i++) {
        let item = res[i];
        if (config.data.R18) {
          const j = this.r18KeyWords.some(v => item.name.includes(v))
          const k = this.r18KeyWords.some(v => item.type.includes(v))
          if (j !== true && k !== true) {
            this.flowList.push(item);
          }
        } else {
          this.flowList.push(item);
        }
      }
      if (this.flowList.length <= 6 && page) {
        this.pageCount--;
        this.addData(key, this.pageCount, t);
      }
    }
  },
  onPageScroll(e) {
    this.scrollTop = e.scrollTop;
  },
  onLoad() {
    this.getSite();
  },
  async onReachBottom() {
    if (this.search !== '') {
      this.loadStatus = "nomore";
      return false
    }
    this.loadStatus = "loading";
    this.pageCount--
    await this.addData(this.site.key, this.pageCount, this.type.tid);
    this.loadStatus = "loadmore";
  },
  onTabItemTap() {
    this.getSite();
  }
};
</script>

<style lang="scss" scoped>
.film {
  padding: 20rpx 20rpx;
  background-color: #f8f8f8;
  .btns {
    margin-top: 10rpx;
    display: flex;
    justify-content: space-around;
    align-items: center;
  }
  .body {
    .box-warter {
      border-radius: 8rpx;
      padding: 14rpx;
      background-color: #fff;
      .box-info {
        margin-top: 10rpx;
        display: flex;
        justify-content: space-between;
        font-size: 12px;
      }
    }
    .left-box-warter {
      margin: 10rpx 10rpx 10rpx 0;
    }
    .right-box-warter {
      margin: 10rpx 0rpx 10rpx 10rpx;
    }
  }
  .mask{
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
</style>
