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
							<uni-number-box :max="99" :min="0" :value="curSku.count" />
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
					<view class="address" v-if="type==2" @click="chooseAddress">
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
					<view class="btn1" @click.stop="hideShade1(1)" v-if="type==2">他人付款</view>
					<view class="btn2" @click.stop="hideShade1(2)">确认上述信息并支付</view>
				</view>
			</view>
		</view>
		
	</view>
</template>

<script>
	import uniNumberBox from "@/components/uni-number-box/uni-number-box.vue"
  import { mapState } from 'vuex';

	export default {
		onLoad(opt) {
      this.getGoodsDetail()
			this.getDefaultAddress()
    },
		components:{
			uniNumberBox
		},
		data() {
			return {
				type:1,
				showShade1:false,

				address: {
					is_default: 0
				},
				curSku: {
					count: 0,
				},
				title: '',
				image: '',
				skus: [],
				total: '0.00'
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
						goods_num: this.curSku.count,
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
				let t = this.curSku.count * this.curSku.price
				this.total = t.toFixed(2)
			}
		},
		computed: {
      ...mapState('device', ['navBarHeight', 'navBarPadding']),
      ...mapState({
        userInfo: state => state.user.userInfo
      }),
    },
		watch: {
			'curSku.count': {
				handler() {
					this.getTotal()
				}
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
</style>
