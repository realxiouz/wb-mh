<template>
	<!-- 新品推荐 -->
	<view class="hot-goods mx20 mb10" v-if="goodsList.length">
		<div style="height:100rpx;padding-left: 10rpx;" class="flex align-center">
      <img src="/static/imgs/xptj.jpg" style="height:60rpx;" mode='heightFix' alt="">
    </div>
    <scroll-view scroll-x>
      <view class="goods-item" v-for="(item,index) in goodsList" :key="index" @click="jump('/pages/goods/mh/home', { id: item.id })">
        <image :src="item.image" mode="aspectFill"></image>
        <view class="text">
          <view class="left">
            <text class="t1">{{item.title}}</text>
            <text class="t2">{{item.subtitle}}</text>
          </view>
          <view class="right">
            {{item.price}}
          </view>
        </view>
      </view>
    </scroll-view>
	</view>
</template>

<script>
/**
 * 自定义之为你推荐
 * @property {Object} detail - 推荐商品信息
 * ids image name
 */
export default {
	components: {},
	data() {
		return {
			listParams: {
				page: 1
			},
			lastPage: 1, //分页总数
			total: 0, //总商品数
			perPage: 0, //单页商品数
			goodsList: [],
			isRefresh: false
		};
	},
	props: {
		detail: {
			type: Object,
			default: null
		}
	},
	created() {
		if (this.detail.ids) {
			this.listParams.goods_ids = this.detail.ids;
			this.getGoodsList();
		}
	},
	computed: {},
	methods: {
		// 商品列表
		getGoodsList() {
			let that = this;
			that.$api('goods.lists', this.listParams).then(res => {
				if (res.code === 1) {
					this.lastPage = res.data.last_page;
					this.total = res.data.total;
					this.perPage = res.data.per_page;
					this.isRefresh = false;
					that.goodsList = [...that.goodsList, ...res.data.data];
				}
			});
		},

		// 加载更多
		loadMore() {
			if (!this.isRefresh) {
				// 加载更多
				if (this.listParams.page < this.lastPage) {
					this.isRefresh = true;
					this.listParams.page += 1;
					this.getGoodsList();
				} else {
					// 重置为1页数据
					this.listParams.page = 1;
					this.lastPage = 1;
					this.goodsList = [];
					this.getGoodsList();
				}
			}
		},

		jump(path, parmas) {
			this.$Router.push({
				path: path,
				query: parmas
			});
		}
	}
};
</script>

<style lang="scss">
.goods-item{
  border-radius: 20rpx;
  overflow: hidden;
  background-color: white;
  margin-right: 16rpx;
  box-shadow:  10rpx 20rpx 30rpx rgba(0,0,0,0.07);
  width: 300rpx;
  height: 420rpx;
  image{
    width: 300rpx;
    height: 300rpx;
  }
  .text{
    height: 100rpx;
    padding: 10rpx;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .left{
    display: flex;
    flex-direction: column;
  }
  .t1{
    font-size: 28rpx;
    font-weight: bold;
  }
  .t2{
    font-size: 26rpx;
    color: #444;
  }
  .right{
    font-size: 30rpx;
    font-weight: bold;
    color: #F97309;
    &::before{
      content: "￥";
    }
  }
}
</style>
