<template>
  <div>
    <div v-for="(i,inx) in list" :key="inx" class="item">
      <img style="width:200rpx;height:200rpx;margin-right:16rpx;" :src="i.goods_image" alt="">
      <div class="flex-sub flex flex-direction justify-between">
        <div class="text-grey">{{i.goods_title}}</div>
        <div>
          <div class="cu-tag line-blue radius">{{i.goods_sku_text}}</div>
          <div>{{}}</div>
        </div>
        <div class="flex align-center">
          <img :src="i.user.avatar" style="width:72rpx;height:72rpx;border-radius:50%;margin-right:8rpx;" alt="">
          <div class="text-gray text-sm">{{i.user.nickname}}赠送给您</div>
          <div class="flex-sub"></div>
          <div class="cu-tag bg-olive radius">{{i.paytime_text.split(' ')[0]}}</div>
        </div>
      </div>
    </div>

    <shopro-empty v-if="!list.length && isLoaded" :emptyData="emptyData" />
  </div>
</template>

<script>
import api from '@/common/request/index'

export default {
  onLoad(opt) {
    this.getData()
  },
  data() {
    return {
      list: [],
      isLoading: false,
      isEnd: false,
      isLoaded: false,
      page: 1,
      emptyData: {
        img: this.$IMG_URL + '/imgs/empty/empty_goods.png',
				tip: '暂无数据，去首页看看吧~',
				path: '/pages/index/index',
				pathText: '去首页逛逛'
      }
    }
  },
  methods: {
    getData() {
      let d = {
        page: this.page
      }
      this.isLoading = true
      api('box.give', d).then(r => {
        if (this.page >= r.data.last_page) {
          this.isEnd = true
        }
        this.list.push(...r.data.data)
      }).finally(_ => {
        this.isLoaded = true
        this.isLoading = false
      })
    }
  },
  onReachBottom() {
    if (this.isLoading || this.isEnd) {
      return
    }
    this.page++
    this.getData()
  }
}
</script>

<style lang="scss" scoped>
.item{
  margin-top: 16rpx;
  background: #fff;
  padding: 12rpx;
  display: flex;
}
</style>