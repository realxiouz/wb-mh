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
				<!-- <view class="top">
					<image src="/static/imgs/shake.png" mode="aspectFit"></image>
					<text>摇一摇盲盒</text>
				</view> -->
				<view class="box">
					<view class="top">
						<image :src="image" mode="widthFix"></image>
					</view>
					<view class="box-list">
						<view class="item" v-for="(item,index) in skus" :key="index" @click="detail(item.isOpen)">
							<!-- <image v-if="!item.isOpen" src="../../static/image/temp16.png" mode="aspectFit"></image>
							<image v-else src="../../static/image/temp15.png" mode="aspectFit"></image> -->
              <image :src="item.image" mode="aspectFit" @click="goSku(item)" />
							<view class="position" v-if="!item.isOpen">
								{{index+1}}
							</view>
						</view>
					</view>
				</view>
				<text class="box-id">NO.123457·5</text>
			</view>
		</view>
		<view class="bottom">
			<view class="b1">
				<view class="left">
					<view class="item">
						<image :src="userInfo.avatar" mode="aspectFill"></image>
					</view>
					<view class="item">
						<image src="/static/imgs/you.png" mode="aspectFill"></image>
					</view>
				</view>
				<view class="right">
					请选择盲盒
				</view>
			</view>
			<view class="b2">
				<image src="../../static/image/temp13.png" mode="aspectFit"></image>
				<text>点击查看商品详情</text>
			</view>
		</view>
		
	</view>
</template>

<script>
  import { mapState } from 'vuex';

	export default {
    onLoad(opt) {
      this.getGoodsDetail()
    },
		data() {
			return {
				type:1,
				showShade1:false,
				list:[],


        title: '',
        skus: [],
        image: '',
			};
		},
		methods:{
			back(){
				uni.navigateBack()
			},
			detail(status){
				if(status){
					return
				}
				uni.navigateTo({
					url:"/pages/detail/detail"
				})
			},
      getGoodsDetail() {
        let that = this;
        this.$api('goods.detail', {
          id: that.$Route.query.id
        }).then(res => {
          if (res.code === 1) {
            let { title, sku_price, image} = res.data
            this.title = title
            this.skus = sku_price
            this.image = image
          }
          if (res.code == 0) {
            that.$tools.toast(res.msg);
          }
        });
      },
			goSku() {
				this.$Router.push({
					path: '/pages/goods/mh/sku',
					query: {
						id: this.$Route.query.id
					}
				});
			}
		},
    computed: {
      ...mapState('device', ['navBarHeight', 'navBarPadding']),
      ...mapState({
        userInfo: state => state.user.userInfo
      })
    }
	}
</script>

<style lang="scss">
.content{
	background-image: linear-gradient(to bottom,#7F3CEE,#608BE7);
	height: calc(100vh - 300rpx);
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
			.top{
				width: 500rpx;
				image{
					width: 100%;
				}
			}
			.box-list{
				display: flex;
				flex-wrap: wrap;
				width: 500rpx;
				.item{
					width: 125rpx;
					height: 125rpx;
					position: relative;
					image{
						width: 100%;
						height: 100%;
					}
					.position{
						position: absolute;
						left: 10rpx;
						top: 6rpx;
						font-size: 24rpx;
						z-index: 10;
					}
				}
			}
		}
		.box-id{
			font-size: 20rpx;
			color: white;
			margin-top: 10rpx;
		}
	}
}
.bottom{
	.b1{
		height: 180rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 30rpx;
		.left{
			display: flex;
			align-items: center;
			.item{
				margin: 0 10rpx;
				width: 80rpx;
				height: 80rpx;
				image{
					width: 100%;
					height: 100%;
					border-radius: 50%;
					border: 4rpx solid #FF6300;
				}
				&:nth-last-child(1){
					image{
						border: 0;
					}
				}
			}
		}
		.right{
			height: 70rpx;
			display: flex;
			align-items: center;
			padding: 0 100rpx;
			border-radius: 40rpx;
			background-color: #F1F1F1;
			font-size: 28rpx;
			color: #C0C3CD;
		}
	}
	.b2{
		height: 120rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		background-color: white;
		box-shadow: 0 0 40rpx rgba(0,0,0,0.1);
		border-top-left-radius: 60rpx;
		border-top-right-radius: 60rpx;
		
		image{
			width: 50rpx;
			height: 50rpx;
				margin-right: 20rpx;
			text{
				font-size: 26rpx;
			}
		}
	}
	}
</style>