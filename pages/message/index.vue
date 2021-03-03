<template>
  <div>
      <div :style="{ height: `${navBarHeight}px`, padding: navBarPadding }" style="background: rgb(239, 196, 128);position:sticky;top:0;z-index:10;" >
			</div>
      <div style="height:300rpx;background: linear-gradient(180deg, rgba(239, 196, 128, 1) 0%, rgba(252, 233, 202, 1) 100%)" class="flex">
        <div class="flex-sub">
          <img :src="userInfo.avatar" style="width:200rpx;height:200rpx;border-radius:50%;">
        </div>
        <div class="flex-sub text-center">
          <div>20</div>
          <div>抽盒</div>
        </div>
        <div class="flex-sub text-center">
          <div>0</div>
          <div>消息</div>
        </div>
      </div>
    <div style="margin-top:-36rpx; border-radius:20rpx;overflow:hidden;">
      <div
        v-for="(i,inx) in list"
        :key="inx"
        class="message-item flex align-center"
      >
        <img :src="i.user.avatar" style="width:100rpx;height:100rpx;margin-right:20rpx;border-radius:50%;" alt="">
        <div class="flex-sub">
          <div class="text-bold">{{i.user.nickname}}</div>
          <div style="color:#929292;font-size:12px;margin-top:10rpx;">{{i.paytime_text}}</div>
        </div>
        <img :src="i.goods_image" style="width:120rpx;height:176rpx;" alt="">
      </div>
    </div>
    <shopro-login-modal />
    <shopro-tabbar />
  </div>
</template>

<script>
import { mapState } from 'vuex'
import api from '@/common/request/index'

export default {
  onLoad() {
    this.getData()
  },
  data() {
    return {
      page: 1,
      list: [],
      isLoading: false,
      isEnd: false,
    }
  },
  methods: {
    getData() {
      let d = {
        page: this.page
      }
      api('box.message', d)
        .then(r => {
          this.list.push(...r.data.data)
          if (this.page >= r.data.last_page) {
            this.isEnd = true
          }
        })
    }
  },
  computed: {
    ...mapState('device', ['navBarHeight', 'navBarPadding']),
    ...mapState('user', ['userInfo']),
	},
}
</script>

<style lang="scss">
.header{
  // background: linear-gradient(180deg, rgba(239, 196, 128, 1) 0%, rgba(248, 220, 165, 1) 100%);
}

.message-item{
  background: #fff;
  width: 700rpx;
  margin: 0 auto;
  padding: 12rpx;
  border-bottom: 1rpx solid #ccc;
}
</style>