<template>
	<!-- 为你推荐 -->
	<view class="hot-goods mx20 mb10" v-if="goodsList.length">
		<div style="height:100rpx;padding-left: 10rpx;" class="flex align-center">
      <img src="/static/imgs/qbsp.jpg" style="height:60rpx;" mode='heightFix' alt="">
    </div>
		<view class="goods-list x-f">
			<view class="goods-item" v-if="goods.id" v-for="goods in goodsList" :key="goods.id">
				<!-- <shopro-goods-card :detail="goods" :isTag="true"></shopro-goods-card> -->
				<div style="background:#fff;" @click="jump('/pages/goods/mh/home', { id: goods.id })">
					<img :src="goods.image" style="width:345rpx;height:345rpx" />
					<div class="flex align-center" style="padding:0 20rpx;">
						<div style="flex:1;">
							<div style="font-size:28rpx;font-weight:bold;">{{goods.title}}</div>
							<div style="font-size:26rpx;color:#444;">{{goods.subtitle}}</div>
						</div>
						<div style="font-size: 30rpx;font-weight: bold;color: #F97309;">￥{{goods.price}}</div>
					</div>
				</div>
			</view>
		</view>
		<button v-if="total > perPage" class="cu-btn refresh-btn my20 x-f" @tap.stop="loadMore">
			<text class="cuIcon-refresh" :class="{ 'refresh-active': isRefresh }"></text>
			{{ listParams.page >= lastPage ? '收起' : '加载更多' }}
		</button>
	</view>
</template>

<script>
/**
 * 自定义之为你推荐
 * @property {Object} detail - 推荐商品信息
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
		if (this.detail.id) {
			this.listParams.category_id = this.detail.id;
			this.getGoodsList();
		}
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
// 为你推荐
.hot-goods {
	background: linear-gradient(#fff 200rpx, #f6f6f6 500rpx, #f6f6f6);
	border-radius: 20rpx;
	.goods-list {
		flex-wrap: wrap;
		width: 710rpx;
		.goods-item {
			margin-right: 20rpx;
			margin-bottom: 20rpx;
			width: 345rpx;
			box-shadow: 0px 0px 10rpx 4rpx rgba(199, 199, 199, 0.22);
			border-radius: 20rpx;
			&:nth-child(2n) {
				margin-right: 0;
			}
		}
	}
	.refresh-btn {
		margin-left: 50%;
		transform: translateX(-50%);
		width: 156rpx;
		height: 50rpx;
		background: #ffffff;
		border-radius: 25rpx;
		font-size: 22rpx;
		font-weight: 500;
		color: #999999;
		white-space: nowrap;
		.cuIcon-refresh {
			color: #dbdbdb;
			margin-right: 12rpx;
			font-size: 32rpx;
		}
	}
	.refresh-active {
		transform: rotate(360deg);
		transition: all linear 0.5s;
	}
}
</style>
