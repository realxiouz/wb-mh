<template>
	<div>
		<div style="height: 12rpx;"></div>
		<div class="flex" style="background: #fff;padding: 20rpx;">
			<img :src="goods.goods_image" style="width: 200rpx;height: 200rpx;margin-right: 20rpx;" alt="">
			<div class="flex-sub">
				<div class="text-bold" style="margin-bottom: 20rpx;">{{goods.goods_title}}</div>
				<div class="flex align-center" style="color:#929292;font-size:12px;margin-bottom: 20rpx;">
					数量: {{goods.goods_num}}
					<div style="margin-left: 16rpx;" class="cu-tag bg-olive radius">{{goods.goods_sku_text}}</div>
				</div>
				<div>¥{{goods.goods_price}}</div>
			</div>
		</div>

		<div style="height: 12rpx;"></div>
		<div v-for="(comment,inx) in list" :key="inx">
			<view class="comment-box">
				<view class="head x-bc">
					<view class="x-f">
						<image class="avatar" :src="comment.user ? comment.user.avatar : $IMG_URL+'/imgs/base_avatar.png'" mode="aspectFill"></image>
						<view class="user-name one-t">{{ comment.user ? comment.user.nickname : '***' }}</view>
					</view>
					<text class="time">{{ comment.createtime_text }}</text>
				</view>
				<view class="detail">{{ comment.content }}</view>
				<view class="img-box">
					<view class="mask"></view>
					<scroll-view class="nav" enable-back-to-top scroll-x scroll-with-animation :scroll-left="scrollLeft">
						<image
							v-for="(img, index) in comment.images"
							:key="index"
							@tap="imgSelect(index)"
							@tap.stop="tools.previewImage(comment.images, index)"
							class="comment-img"
							:src="img"
							mode="aspectFill"
						></image>
					</scroll-view>
				</view>
				<div v-if="comment.reply.length" style="background:#e4e4e4;padding: 20rpx;border-radius: 10rpx;position:relative;">
					<div v-for="(i, index) in comment.reply" :key="index" class="flex align-center">
						<img :src="i.user.avatar" style="width:40rpx;height:40rpx;border-radius:50%;margin-right:8rpx;" alt="">
						<div>{{i.content}}</div>
					</div>
				</div>
				<div v-if="userInfo.id==goods.user_id" @click="onComment(comment.id)" class="flex align-center">
					<div class="flex-sub"></div>
					<div class="cuIcon-comment" style="font-size:20px;"></div>
					<div style="font-size:14px;">回复Ta</div>
				</div>
			</view>
		</div>
		<shopro-empty v-if="!list.length && isLoaded" :emptyData="emptyData" />
		<div style="height:120rpx;"></div>
		<view class="foot_box x-c">
			<button class="cu-btn post-btn" :disabled="isFormEnd" @tap="onComment">
				<text v-if="isFormEnd" class="cuIcon-loading2 cuIconfont-spin"></text>
				发表评论
			</button>
		</view>
	</div>
</template>

<script>
	import api from '@/common/request/index'
	import { mapState } from 'vuex'
	export default {
		onLoad(opt) {
			this.opt = opt
			// this.getData()
			console.log(this.userInfo)
		},
		onShow() {
			this.getData(true)
		},
		data() {
			return {
				page: 1,
				list: [],
				isLoading: false,
				isEnd: false,
				isLoaded: false,
				
				goods: {},

				emptyData: {
			  img: this.$IMG_URL + '/imgs/empty/empty_goods.png',
				tip: '暂无数据，去首页看看吧~',
				path: '/pages/index/index',
				pathText: '去首页逛逛'
			}
			}
		},
		methods: {
			getData(reset=false) {
				if (reset) {
					this.list = []
				}
				this.isLoading = true
				api('box.list', {order_item_id: this.opt.id})
					.then(r => {
						this.goods = r.data.order_goods
						this.list.push(...r.data.list)
					})
					.finally(_ => {
						this.isLoaded = true
						this.isLoading = false
					})
			},
			onComment(replayId) {
				let url = `/pages/message/comment?orderItemId=${this.opt.id}`
				if (replayId) {
					url = `${url}&rId=${replayId}`
				}
				uni.navigateTo({
					url
				})
			}
		},
		computed: {
			...mapState({
				userInfo: state => state.user.userInfo,
			}),
		}
	}
</script>

<style lang='scss'>
.comment-box {
	padding: 30rpx 20rpx;
	border-bottom: 1rpx solid #eee;
	background: #fff;

	.head {
		margin-bottom: 20rpx;

		.avatar {
			width: 52rpx;
			height: 52rpx;
			border-radius: 50%;
			background: #ccc;
		}

		.user-name {
			font-size: 26rpx;
			color: #999;
			width: 140rpx;
			margin: 0 20rpx;
		}

		.star-box {
			.cuIcon-favorfill {
				font-size: 30rpx;
				color: #dedede;
			}

			.star-active {
				color: #ffba00;
			}
		}

		.time {
			font-size: 24rpx;
			color: #c4c4c4;
		}
	}

	.detail {
		font-size: 26rpx;
		font-family: PingFang SC;
		font-weight: 500;
		color: rgba(102, 102, 102, 1);
		line-height: 42rpx;
	}

	.img-box {
		margin-top: 30rpx;
		position: relative;

		.mask {
			background: linear-gradient(270deg, rgba(255, 255, 255, 0.8) 0%, rgba(255, 255, 255, 0) 100%);
			width: 100rpx;
			height: 100%;
			position: absolute;
			z-index: 10;
			right: 0;
		}

		.comment-img {
			width: 166rpx;
			height: 166rpx;
			background: #ccc;
			margin-right: 19rpx;
			margin-bottom: 19rpx;
		}
	}
}

.foot_box{
	position: fixed;
	left: 0;
	bottom: 0rpx;
	right: 0;
	z-index: 10;
}

.post-btn {
	width: 690rpx;
	height: 80rpx;
	background: linear-gradient(90deg, rgba(240, 199, 133, 1), rgba(246, 214, 157, 1));
	border-radius: 40rpx;
	color: rgba(#fff, 0.9);
	margin-bottom: 30rpx;
}
</style>
