<template>
	<view>
		<view class="content">
			<div :style="{ height: `${navBarHeight}px`, padding: navBarPadding }" class="flex align-center" @click="back">
        <img src="/static/imgs/back.png" style="height:36rpx;margin-right:10rpx;" mode="heightFix" />
        <div style="font-weight:bold;color:#fff;font-size:16px;">{{title}}</div>
      </div>
			<view class="count-down">
				<image src="/static/imgs/timer.png" mode="aspectFit"></image>
				<text>剩余抽盒时间：</text>
				<text class="num">123s</text>
			</view>
			<view class="box-content">
				<view class="top">
					<image src="/static/imgs/shake.png" mode="aspectFit"></image>
					<text>摇一摇盲盒</text>
				</view>
				<view class="box">
					<image :src="image" mode="aspectFit"></image>
				</view>
				<text class="box-id">NO.123457·5</text>
			</view>
			<view class="scroll">
				<scroll-view scroll-x="true" >
					<view class="box">
						<view class="item" v-for="(i,inx) in skus" :key="inx">
							<image :src="i.image" mode="aspectFit"></image>
							<view class="text">{{i.goods_sku_text}}</view>
						</view>
					</view>
				</scroll-view>
			</view>
		</view>
		<view class="tab">
			<view class="item">
				<image src="/static/imgs/tsk.png" mode="aspectFit"></image>
				<text>提示卡</text>
			</view>
			<view class="item">
				<image src="/static/imgs/xsk.png" mode="aspectFit"></image>
				<text>显示卡</text>
			</view>
		</view>
		<view class="btn-group">
			<view class="btn btn1" @click="showShade(1)">
				虚拟盲盒
			</view>
			<view class="btn btn2" @click="showShade(2)">
				实体盲盒
			</view>
		</view>
		<view class="shade" v-if="showShade1" @click="hideShade()">
			<view class="con shade1" v-if="showShade1" @click.stop="clickStop">
				<view class="title">
					{{curSku.goods_sku_text}}
				</view>
				<view class="box">
					<view class="item">
						<view class="lable">
							单价：
						</view>
						<view class="value">
							<view class="text">
								{{curSku.price}}元
							</view>
						</view>
					</view>
					<view class="item" style="align-items:center;">
						<view class="lable">
							数量：
						</view>
						<view class="value">
							<uni-number-box :max="99" :min="0" v-model="count" @change="onChange" />
						</view>
					</view>
					<view class="item">
						<view class="lable">
							合计：
						</view>
						<view class="value">
							<view class="text">
								{{total}}元
							</view>
						</view>
					</view>
					<div class="item" v-if="type==1">
						<view class="lable">
							秘钥：
						</view>
						<div class="value">
							<input type="text" placeholder="输入秘钥" v-model="addressMy">
						</div>
					</div>
					<view class="address" @click="chooseAddress">
						<view class="left" v-if="address.id">
							<view class="t1">
								{{`${address.consignee} ${address.phone}`}}
							</view>
							<view class="t2">
								{{`${address.province_name}${address.city_name}${address.area_name}${address.address}`}}
							</view>
						</view>
						<view v-else class="left" style="font-size: 26rpx;color: #666;">
							请选择收货地址
						</view>
						<view class="right">
							<image src="/static/imgs/edit.png" mode="aspectFit"></image>
						</view>
					</view>
					<view class="item">
						<view class="lable">
							提示：
						</view>
						<view class="value">
							<view class="text1">
								1、盲盒类商品付款后即完成在线拆盒，不适用七日无理由退货；2、购买商品不足3个将收取8元运费，慢3个包邮
							</view>
						</view>
					</view>

					
					
				</view>
				<view class="btn-group">
					<button @click="payByOther" class="btn1" style="margin:0;" v-if="type==2">他人付款</button>
					<view class="btn2" @click.stop="hideShade1(type)">确认上述信息并支付</view>
				</view>
			</view>
		</view>
		
		<shopro-modal v-model="show">
			<div slot="modalContent">
				<div style="background:#fff;padding: 0 24rpx;">
					<div style="padding:24rpx 0;font-weight:bold;font-size:16px;">温馨提示</div>
					<div style="font-size:14px;color:#666;padding: 0 0 24rpx 0;">订单已生成,点击按钮,让Ta帮你付款吧~~~</div>
					<button
						style="height:80rpx;width:260rpx;border-radius:40rpx;background:#FDC204;font-size:12px;color:#fff;margin:0 auto;border:none;"
					 	open-type="share" class="reset flex align-center justify-around" @click="show=false">发给Ta</button>
					<div style="height:36rpx;"></div>
				</div>
			</div>
		</shopro-modal>

		<shopro-modal v-model="showOtherBuy" modalType='bottom-modal'>
			<div slot="modalContent">
				<div style="background:#fff;padding: 0 24rpx;">
					<div style="padding:24rpx 0;font-weight:bold;font-size:16px;">{{otherPayInfo.consignee}}请你送Ta一个玩偶</div>

					<view class="address" style="text-align:left;">
						<view class="left">
							<view class="t1">
								{{`${otherPayInfo.consignee} ${otherPayInfo.phone}`}}
							</view>
							<view class="t2">
								{{`${otherPayInfo.province_name}${otherPayInfo.city_name}${otherPayInfo.area_name}${otherPayInfo.address}`}}
							</view>
						</view>
					</view>
					<div style="font-size:14px;color:#666;margin-bottom:24rpx;"><span style="font-weight:bold;color:#333;">总价:</span>{{otherPayInfo.total_fee}}</div>
					<div @click="doOtherPay" class="flex align-center justify-around"
						style="height:80rpx;width:260rpx;border-radius:40rpx;background:#FDC204;font-size:12px;color:#fff;margin:0 auto;">
						确认赠送给Ta
					</div>
					<div style="height:32px"></div>
				</div>
			</div>
		</shopro-modal>
	</view>
</template>

<script>
	import uniNumberBox from "@/components/uni-number-box/uni-number-box.vue"
  import { mapState } from 'vuex';
	import Wechat from '@/common/wechat/wechat' 


	export default {
		onLoad(opt) {
      this.getGoodsDetail()
			this.getDefaultAddress()

			if (this.$Route.query.orderSn) {
				this.getPayInfo()
			}
    },
		components:{
			uniNumberBox,
		},
		data() {
			return {
				type:1,
				showShade1:false,

				address: {
					is_default: 0
				},
				curSku: {},
				title: '',
				image: '',
				skus: [],
				total: '0.00',
				count: 1,

				show: false,
				showOtherBuy: false,
				otherPayInfo: {},
				addressMy: ''
			};
		},
		methods:{
			back(){
				uni.navigateBack()
			},
			showShade(type){
				if(type==1){
					this.type=1
					this.showShade1=true
				}else if(type==2){
					this.type=2
					this.showShade1=true
				}
			},
			chooseAddress(){
				uni.navigateTo({
					url:"/pages/user/address/list"
				})
				
			},
			hideShade(){
				this.showShade1=false;
			},
			hideShade1(type){
				if(type==1){
					if (!this.addressMy.trim()) {
						uni.showToast({
							title: '必须输入虚拟秘钥',
							icon: 'none'
						})
						return
					}
					this.showShade1=false
					this.useVirtual()
				}else if (type==2){
					this.showShade1=false
					this.createOrder()
				}
			},
			clickStop(){},
			getGoodsDetail() {
        let that = this;
        this.$api('goods.detail', {
          id: that.$Route.query.id
        }).then(res => {
          if (res.code === 1) {
            let { title, sku_price, image, } = res.data
            this.title = title
            this.skus = sku_price
            this.image = image

						this.curSku = {...this.skus.find(i => i.id == this.$Route.query.sId), count: 1}
          }
          if (res.code == 0) {
            that.$tools.toast(res.msg);
          }
        });
      },
			getDefaultAddress() {
				this.$api('address.defaults').then(res => {
					if (res.code === 1) {
						if (res.data) {
							this.address = res.data || {};
						}
					}
				});
			},
			createOrder() {
				if (!this.address.id) {
					uni.showToast({
						title: '还没有选择配送地址呢',
						icon: 'none'
					})
					return
				}
				this.$api('order.createOrder', {
					goods_list: [{
						dispatch_type: "express",
						goods_id: this.curSku.goods_id,
						goods_num: this.count,
						goods_price: this.curSku.price,
						// sku_price_id: this.curSku.id,
					}],
					from: 'alone', // that.from,
					address_id: this.address.id,
					coupons_id: 0, //that.couponId,
					remark: '',
					order_type: 'goods', // that.orderType,
					buy_type: 'buy_type', //that.grouponBuyType,
					// groupon_id: that.grouponId
				}).then(r => {
					let params = {
						order_sn: r.data.order_sn,
						payment: 'wechat'
					}
					if (uni.getStorageSync('openid')) {
						params.openid = uni.getStorageSync('openid');
					}
					this.$api('pay.prepay', params).then(res => {
						if (res.code === 1) {
							if (res.data === 'no_openid') {
								uni.hideLoading();
								uni.showModal({
									title: '微信支付',
									content: '点击确定后请再次使用微信支付',
									success: function(res) {
										if (res.confirm) {
											//静默获取openid
											let wechat = new Wechat();
											wechat.wxOfficialAccountBaseLogin();
										} else if (res.cancel) {
											console.log('用户点击取消');
										}
									},
									fail: (err) => {
										uni.hideLoading();
									}
								});

							} else {
								uni.hideLoading();
								let payData = res.data.pay_data
								uni.requestPayment({
									provider: 'wxpay',
									timeStamp: payData.timeStamp,
									nonceStr: payData.nonceStr,
									package: payData.package,
									signType: payData.signType,
									paySign: payData.paySign,
									success: _ => {
										this.$Router.replace({
											path: '/pages/order/payment/result',
											query: {
												orderSn: r.data.order_sn,
												type: 'wechat',
												pay: 1
											}
										});
										// uni.navigateBack()
									},
									fail: err => {
										console.log('支付取消或者失败:', err);
										if (err.errMsg !== "requestPayment:fail cancel") {
											this.$Router.replace({
												path: '/pages/order/payment/result',
												query: {
													orderSn: r.data.order_sn,
													type: 'wechat',
													pay: 0
												}
											});
										}
									}
								});
							}
						} else {
							uni.hideLoading();
							uni.showToast({
								title: res.msg,
								icon: 'none'
							})
						}
					})
						})
			},
			getTotal() {
				let t = this.count * this.curSku.price
				this.total = t.toFixed(2)
			},
			onChange(e) {
				this.count = e
			},
			useVirtual() {
				if (!this.address.id) {
					uni.showToast({
						title: '还没有选择配送地址呢',
						icon: 'none'
					})
					return
				}
				if (!this.addressMy.trim()) {
					uni.showToast({
						title: '必须输入虚拟秘钥',
						icon: 'none'
					})
					return
				}
				this.$api('order.virtual', {
					goods_list: [{
						dispatch_type: "express",
						goods_id: this.curSku.goods_id,
						goods_num: this.count,
						goods_price: this.curSku.price,
						// sku_price_id: this.curSku.id,
					}],
					from: 'alone', // that.from,
					address_id: this.address.id,
					coupons_id: 0, //that.couponId,
					remark: '',
					order_type: 'goods', // that.orderType,
					buy_type: 'buy_type', //that.grouponBuyType,
					address: this.addressMy
				})
					.then(r => {
						let params = {
						order_sn: r.data.order_sn,
						payment: 'wechat'
					}
					if (uni.getStorageSync('openid')) {
						params.openid = uni.getStorageSync('openid');
					}
					this.$api('pay.prepay', params).then(res => {
						if (res.code === 1) {
							if (res.data === 'no_openid') {
								uni.hideLoading();
								uni.showModal({
									title: '微信支付',
									content: '点击确定后请再次使用微信支付',
									success: function(res) {
										if (res.confirm) {
											//静默获取openid
											let wechat = new Wechat();
											wechat.wxOfficialAccountBaseLogin();
										} else if (res.cancel) {
											console.log('用户点击取消');
										}
									},
									fail: (err) => {
										uni.hideLoading();
									}
								});

							} else {
								uni.hideLoading();
								let payData = res.data.pay_data
								uni.requestPayment({
									provider: 'wxpay',
									timeStamp: payData.timeStamp,
									nonceStr: payData.nonceStr,
									package: payData.package,
									signType: payData.signType,
									paySign: payData.paySign,
									success: _ => {
										this.$Router.replace({
											path: '/pages/order/payment/result',
											query: {
												orderSn: r.data.order_sn,
												type: 'wechat',
												pay: 1
											}
										});
										// uni.navigateBack()
									},
									fail: err => {
										console.log('支付取消或者失败:', err);
										if (err.errMsg !== "requestPayment:fail cancel") {
											this.$Router.replace({
												path: '/pages/order/payment/result',
												query: {
													orderSn: r.data.order_sn,
													type: 'wechat',
													pay: 0
												}
											});
										}
									}
								});
							}
						} else {
							uni.hideLoading();
							uni.showToast({
								title: res.msg,
								icon: 'none'
							})
						}
					})
					})
			},
			payByOther() {
				this.$api('order.createOrder', {
					goods_list: [{
						dispatch_type: "express",
						goods_id: this.curSku.goods_id,
						goods_num: this.count,
						goods_price: this.curSku.price,
						// sku_price_id: this.curSku.id,
					}],
					from: 'alone', // that.from,
					address_id: this.address.id,
					coupons_id: 0, //that.couponId,
					remark: '',
					order_type: 'goods', // that.orderType,
					buy_type: 'buy_type', //that.grouponBuyType,
				}).then(r => {
					this.show = true
					this.shareObj = {
						title: '请你送Ta一个玩偶',
						imageUrl: this.image,
						path: `/pages/goods/mh/sku?id=${this.$Route.query.id}&orderSn=${r.data.order_sn}&sId=${this.$Route.query.sId}`
					}
				})
			},
			doOtherPay() {
				let params = {
						order_sn: this.$Route.query.orderSn,
						payment: 'wechat'
					}
					if (uni.getStorageSync('openid')) {
						params.openid = uni.getStorageSync('openid');
					}
					this.$api('pay.prepay', params)
						.then(res => {
							let payData = res.data.pay_data
								uni.requestPayment({
									provider: 'wxpay',
									timeStamp: payData.timeStamp,
									nonceStr: payData.nonceStr,
									package: payData.package,
									signType: payData.signType,
									paySign: payData.paySign,
									success: _ => {
										this.$Router.replace({
											path: '/pages/order/payment/result',
											query: {
												orderSn: r.data.order_sn,
												type: 'wechat',
												pay: 1
											}
										});
										// uni.navigateBack()
									},
									fail: err => {
										console.log('支付取消或者失败:', err);
										if (err.errMsg !== "requestPayment:fail cancel") {
											this.$Router.replace({
												path: '/pages/order/payment/result',
												query: {
													orderSn: r.data.order_sn,
													type: 'wechat',
													pay: 0
												}
											});
										}
									}
								});
						})
			},
			getPayInfo() {
				this.$api('order.other', {order_sn: this.$Route.query.orderSn})
					.then(r => {
						this.showOtherBuy = true

						let { consignee, total_fee, address, province_name, city_name, area_name, phone } = r.data
						this.otherPayInfo = {
							consignee,
							total_fee,
							address, province_name, city_name, area_name, phone
						}
					})
			}
		},
		computed: {
      ...mapState('device', ['navBarHeight', 'navBarPadding']),
      ...mapState({
        userInfo: state => state.user.userInfo
      }),
    },
		watch: {
			'count': {
				handler() {
					this.getTotal()
				}
			}
		},
		onShareAppMessage({from}) {
			if (from == 'button') {
				return this.shareObj
			}
		}
	}
</script>

<style lang="scss">
.content{
	background-image: linear-gradient(to bottom,#7F3CEE,#608BE7);
	height: 70vh;
	.head{
		height: 88rpx;
		padding-top: 50rpx;
		display: flex;
		align-items: center;
		image{
			width: 40rpx;
			height: 40rpx;
			margin-left: 20rpx;
		}
		text{
			font-size: 30rpx;
			font-weight: bold;
			color: white;
			margin-left: 40rpx;
		}
	}
	.count-down{
		margin: 0 auto;
		width: 340rpx;
		height: 60rpx;
		background-color:#5A3FCC ;
		border-radius: 6rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		image{
			width: 30rpx;
			height: 30rpx;
			margin-right: 14rpx;
		}
		text{
			font-size: 26rpx;
			color: white;
		}
	}
	.box-content{
		height: calc(100% - 356rpx);
		display: flex;
		align-items: center;
		flex-direction: column;
		justify-content: center;
		.top{
			display: flex;
			align-items: center;
			image{
				width: 40rpx;
				height: 40rpx;
				margin-right: 16rpx;
			}
			text{
				font-size: 26rpx;
				color: white;
			}
		}
		.box{
			margin-top: 20rpx;
			width: 300rpx;
			height: 300rpx;
			image{
				width: 100%;
				height: 100%;
			}
		}
		.box-id{
			color: white;
			font-size: 24rpx;
			margin-top: 20rpx;
		}
	}
	.scroll{
		width: 100%;
		position: relative;
		height: 280rpx;
		.box{
			display: inline-flex;
			white-space: nowrap;
			.item{
				display: flex;
				flex-direction: column;
				align-items: center;
				background-image: linear-gradient(to bottom, #383E8C,#043159);
				width: 190rpx;
				height: 280rpx;
				margin: 0 16rpx;
				justify-content: center;
				border-radius: 8rpx;
				image{
					width:150rpx;
					height: 220rpx;
				}
				.text{
					font-size: 24rpx;
					background-color: white;
					width:140rpx ;
					border-radius: 50rpx;
					display: flex;
					align-items: center;
					justify-content: center;
					margin-top: 10rpx;
				}
			}
		}
	}
}
.tab{
	margin-top: 150rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	.item{
		margin: 0 40rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		image{
			width: 150rpx;
			height: 90rpx;
		}
		text{
			font-size: 24rpx;
			margin-top: 10rpx;
		}
	}
}
.btn-group{
	margin-top: 20rpx;
	display: flex;
	align-items: center;
	justify-content: space-around;
	.btn{
		width: 40%;
		height: 60rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
		border-radius: 60rpx;
	}
	.btn1{
		border: 4rpx solid #333333;
	}
	.btn2{
		border: 4rpx solid #FDC204;
		background-color: #FDC204;
	}
}
.shade{
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 10;
	background-color: rgba(0,0,0,0.6);
	
	.con{
		position: absolute;
		left: 0;
		right: 0;
		bottom: 0;
		padding-bottom: 40rpx;
		background-color: white;
		border-top-right-radius: 30rpx;
		border-top-left-radius: 30rpx;
		.title{
			height: 100rpx;
			font-size: 30rpx;
			font-weight: bold;
			display: flex;
			align-items: center;
			padding: 0 30rpx;
		}
		.box{
			padding: 0 30rpx;
			.item{
				display: flex;
				align-items: flex-start;
				font-size: 26rpx;
				margin-bottom: 30rpx;
				.lable{
					width: 80rpx;
					color: #999;
				}
				.value{
					flex: 1;
					.text{
						font-weight: bold;
						font-size: 28rpx;
					}
					.text1{
						font-size: 24rpx;
					}
				}
			}
			.address{
				border: 1rpx solid #efefef;
				padding:30rpx 20rpx;
				border-radius: 6rpx;
				display: flex;
				align-items: center;
				justify-content: space-between;
				margin-bottom: 30rpx;
				.left{
					flex: 1;
				}
				.t1{
					font-size: 28rpx;
					&::before{
						content: "默认";
						font-size: 20rpx;
						padding: 1rpx 8rpx;
						background-color:#FA7108 ;
						border-radius: 6rpx;
						margin-right: 14rpx;
						color: white;
					}
				}
				.t2{
					font-size: 26rpx;
					color: #666;
					margin-top: 10rpx;
				}
				.right{
					image{
						width: 50rpx;
						height: 50rpx;
					}
				}
			}
		}
		.btn-group{
			height: 100rpx;
			display: flex;
			align-items: center;
			justify-content: space-around;
			.btn1{
				display: flex;
				height: 80rpx;
				align-items: center;
				justify-content: center;
				font-size: 28rpx;
				padding: 0 60rpx;
				border-radius: 60rpx;
				border:1rpx solid #999;
			}
			.btn2{
				background-color: #FDC204;
				display: flex;
				height: 80rpx;
				align-items: center;
				justify-content: center;
				font-size: 28rpx;
				padding: 0 60rpx;
				border-radius: 60rpx;
			}
		}
	}
}

.address{
				border: 1rpx solid #efefef;
				padding:30rpx 20rpx;
				border-radius: 6rpx;
				display: flex;
				align-items: center;
				justify-content: space-between;
				margin-bottom: 30rpx;
				.left{
					flex: 1;
				}
				.t1{
					font-size: 28rpx;
					
				}
				.t2{
					font-size: 26rpx;
					color: #666;
					margin-top: 10rpx;
				}
				.right{
					image{
						width: 50rpx;
						height: 50rpx;
					}
				}
			}

button{
  &.reset{
    padding: 0 ;
    font-size: inherit;
    line-height: inherit;
    background-color: transparent;
    color: inherit;
    margin: 0 ;
    &::after{
      border: none;
    }
  }
}
</style>
