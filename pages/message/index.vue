<template>
  <div>
      <div :style="{ height: `${navBarHeight}px`, padding: navBarPadding }" style="background: rgb(239, 196, 128);position:sticky;top:0;z-index:10;" >
			</div>
      <div style="height:300rpx;background: linear-gradient(180deg, rgba(239, 196, 128, 1) 0%, rgba(252, 233, 202, 1) 100%)" class="flex">
        <div class="flex-sub flex justify-around">
          <img :src="userInfo.avatar" style="width:180rpx;height:180rpx;border-radius:50%;">
        </div>
        <div class="flex-sub text-center">
          <div class="text-bold" style="font-size: 60rpx;margin:20rpx 0;">{{bNum}}</div>
          <div style="font-size: 18px;color: #888888;">抽盒</div>
        </div>
        <div class="flex-sub text-center">
          <div class="text-bold" style="font-size: 60rpx;margin:20rpx 0;">{{total}}</div>
          <div style="font-size: 18px;color: #888888;">消息</div>
        </div>
      </div>
    <div style="margin-top:-36rpx; border-radius:20rpx;overflow:hidden;">
      <div
        v-for="(i,inx) in list"
        :key="inx"
        class="message-item flex align-center"
				@click="onDetail(i)"
      >
        <img :src="i.user.avatar" style="width:100rpx;height:100rpx;margin-right:20rpx;border-radius:50%;" alt="">
        <div class="flex-sub">
          <div class="text-bold">{{i.user.nickname}}</div>
          <div style="color:#929292;font-size:12px;margin-top:10rpx;">{{i.paytime_text}}</div>
        </div>
        <img :src="i.goods_image" style="width:120rpx;height:176rpx;" alt="">
      </div>
    </div>
		<shopro-empty v-if="!list.length && isLoaded" :emptyData="emptyData" />
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
    api('user.info').then(r => {
      this.bNum = r.data.box_num
    })
  },
  data() {
    return {
      page: 1,
      list: [],
      isLoading: false,
      isEnd: false,
			isLoaded: false,
			total: 0,
      bNum: 0,
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
      api('box.message', d)
        .then(r => {
          this.list.push(...r.data.data)
					this.total = r.data.total
          if (this.page >= r.data.last_page) {
            this.isEnd = true
          }
        })
				.finally(_ => {
					this.isLoading = false
					this.isLoaded = true
				})
    },
		onDetail(i) {
			uni.navigateTo({
				url: `/pages/message/list?id=${i.id}`
			})
		}
  },
  computed: {
    ...mapState('device', ['navBarHeight', 'navBarPadding']),
    ...mapState({
				userInfo: state => state.user.userInfo,
			}),
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
	&:last-child{
		border-width: 0;
	}
}
</style>