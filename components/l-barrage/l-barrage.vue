<template>
	<view class="l-barrage">
		<block v-for="(item,index) in items" :key="index">
			<!-- #ifdef H5 -->
			<text v-if="item.display" class="aon" :style="{top: `${item.top}%`,color: item.color}">
				{{item.text}}
			</text>
			<!-- #endif -->

			<!-- #ifndef H5 -->
			<view v-if="item.display" class="aon" :style="{top: `${item.top}%`,color: item.color,
				  animation: `mymove ${Number(item.time)}s linear forwards`
				  }">
				<image :src="item.icon" mode="aspectFill"></image>
				<text>{{item.text}}</text>
			</view>
			<!-- #endif -->

		</block>
	</view>
</template>

<script>
	let cycle;

	// 弹幕字体颜色
	function getRandomColor() {
		let rgb = []
		for (let i = 0; i < 3; ++i) {
			let color = Math.floor(Math.random() * 256).toString(16)
			color = color.length == 1 ? '0' + color : color
			rgb.push(color)
		}
		// return '#' + rgb.join('')
		return "#FFFFFF"
	}

	export default {
		name: 'l-barrage',
		props: {
			minTime: {
				type: Number,
				default: 4
			},
			maxTime: {
				type: Number,
				default: 9
			},
			minTop: {
				type: Number,
				default: 10
			},
			maxTop: {
				type: Number,
				default: 30
			}
		},
		data() {
			return {
				items: [],
			}
		},
		methods: {
			add(text = '', icon = '', time = Math.ceil(Math.floor(Math.random() * (this.maxTime - this.minTime + 1) + this.minTime))) {
				this.items.push({
					icon,
					text,
					time,
					top: Math.ceil(Math.random() * (this.maxTop - this.minTop + 1) + this.minTop),
					color: getRandomColor(),
					display: 1,
				});
			},
			start(items = []) {
				this.items = [];
				cycle && (clearInterval(cycle));
				let i = 0,
					len = items.length;

				cycle = setInterval(() => {
					let time = 5;
					// #ifndef H5
					time = Math.ceil(Math.floor(Math.random() * (this.maxTime - this.minTime + 1) + this.minTime));
					// #endif

					if (i < len) {
						this.add(items[i].text, items[i].icon, time);
						i++;
					} else {
						clearInterval(cycle);
						setTimeout(() => {
							this.$emit("end", {});
						}, time * 1000)
					}
				}, 500)
			}
		}
	}
</script>

<style>
	.aon {
		position: fixed;
		white-space: nowrap;

		animation: mymove 5s linear forwards;
		animation-timing-function: linear;
		-webkit-animation-timing-function: linear;
		animation-fill-mode: forwards;
		display: flex;
		align-items: center;
		background-color: rgba(0, 0, 0, 0.7);
		height: 70rpx;
		border-radius: 35rpx;
		padding: 0 20rpx;

	}

	.aon image {
		width: 54rpx;
		height: 54rpx;
		border-radius: 50%;
		margin-right: 10rpx;
		border: 1rpx solid #ffffff;
	}

	.aon text {
		font-size: 26rpx;
	}

	.l-barrage {
		z-index: 999;
		width: 100%;
		position: fixed;
	}

	@keyframes mymove {
		from {
			left: 100%;
		}

		to {
			left: -200%;
		}
	}

	@-moz-keyframes mymove

	/* Firefox */
		{
		from {
			left: 100%;
		}

		to {
			left: -200%;
		}
	}

	@-webkit-keyframes mymove

	/* Safari and Chrome */
		{
		from {
			left: 100%;
		}

		to {
			left: -200%;
		}
	}

	@-o-keyframes mymove

	/* Opera */
		{
		from {
			left: 100%;
		}

		to {
			left: -200%;
		}
	}
</style>
