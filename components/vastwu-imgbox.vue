<template>
	<view>
		<view class="card-box" :src="item.aname" v-for="(item,i) in list" :key="i" @click.stop="slideclick(item.uid)"
			:style="{'z-index':  zIndexs[`zIndex${i+1}`]+100,'left':lefts[`left${i+1}`],'top':`${-(zIndexs[`zIndex${i+1}`]*offsetY)-offsetY}px`}"
			@touchend="moveEnd($event,i+1)" :animation="animations[`animation${i+1}`]">
			<image class="main-img" :src="item.aname" @touchstart="moveStart($event,i+1)" @touchmove="move($event,i+1)" mode="aspectFill">
			</image>
			<!-- 可自定义内容 -->
			<view class="lookmore" >
				<view class="card-header">
					<!-- 头像 -->
					<u-avatar
						:src="item.logo"
						show-sex sex-icon="woman" show-level>
					</u-avatar>
					<!-- 昵称和时间 -->
					<view class="card-right">
						<view class="right-top-name">{{item.name}}</view>
						<view class="header-right-bottom">
							{{item.sex==0? '男':'女'}}
							<text class="header-right-info">
								<text class="header-right-point">·</text>
								{{item.constellation}}
							</text>
							<text class="header-right-info">
								<text class="header-right-point">·</text>
								{{item.height}} CM
							</text>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			list: {
				type: Array,
				required: true,
				default: function() {
					return []
				}
			},
			offsetX: {
				type: Number,
				default: 10
			},
			offsetY: {
				type: Number,
				default: 10
			},
			auto: {
				type: Boolean,
				default: true
			},
			dur: {
				type: Number,
				default: 5000
			}
		},
		data() {
			return {
				currentIndex: 1,
				animations: {},
				zIndexs: {},
				lefts: {}, //左边偏移量
				running: false,
				touching: false, //是否正按着
				//初始位置
				x: 0,
				y: 0,
				// 移动位置
				x1: 0,
				y1: 0
			}
		},
		created() {
			for (var i = 0; i < this.list.length; i++) {
				this.$set(this.animations, `animation${i+1}`, uni.createAnimation({
					duration: 300,
					timingFunction: 'ease'
				}))
				this.$set(this.zIndexs, `zIndex${i+1}`, -(i + 1))
				this.$set(this.lefts, `left${i+1}`, `${-(this.zIndexs[`zIndex${i+1}`]*this.offsetX)-this.offsetX}px`)
			}
			if (this.auto) {
				setInterval(() => {
					if (!this.running && !this.touching)
						this.action(this.currentIndex)
				}, this.dur)
			}

		},
		
		methods: {
			move(e, i) {
				this.x1 = e.touches[0].pageX;
				this.y1 = e.touches[0].pageY;

				if (this.zIndexs[`zIndex${i}`] == -1 && (this.x - this.x1) > 0)
					this.lefts[`left${i}`] = `${-(this.zIndexs[`zIndex${i}`]*this.offsetX)-this.offsetX}` - (this.x - this
						.x1) + 'px';
			},
			moveEnd(e, i) {
				this.touching = false;
				if ((this.x - this.x1) > 100 && this.x != 0 && this.x1 != 0) {
					if (this.zIndexs[`zIndex${i}`] != -1 || this.running) return
					this.action(i)
				} else {
					this.lefts[`left${i}`] = `${-(this.zIndexs[`zIndex${i}`]*this.offsetX)-this.offsetX}px`;
					this.x = 0
					this.x1 = 0
					this.running = false
				}
			},
			moveStart(e, i) {
				this.touching = true
				this.x = e.touches[0].pageX;
				this.y = e.touches[0].pageY;

			},
			action(i) {
				console.log(i, this.animations[`animation${i}`])
				this.running = true;
				this.animations[`animation${i}`].translateX(-250).translateY(0).rotate(0).opacity(0.3).step();
				this.animations[`animation${i}`].translateX(0).translateY(0).rotate(0).opacity(1).step();
				this.animations[`animation${i}`] = this.animations[`animation${i}`].export();

				setTimeout(() => {
					for (var j = 0; j < this.list.length; j++) {
						if (j + 1 === i) {
							this.zIndexs[`zIndex${j+1}`] = (-this.list.length)
						} else {
							this.zIndexs[`zIndex${j+1}`]++
						}
						this.lefts[`left${j+1}`] = `${-(this.zIndexs[`zIndex${j+1}`]*this.offsetX)-this.offsetX}px`
					}

					if (i == this.list.length) {
						this.currentIndex = 1
					} else {
						this.currentIndex++
					}

					// 重置
					for (var k = 0; k < this.list.length; k++) {
						this.$set(this.animations, `animation${k+1}`, uni.createAnimation({
							duration: 300,
							timingFunction: 'ease'
						}))
					}
					this.running = false
					this.x = 0
					this.x1 = 0
				}, 300)
			},
			slideclick(item) {
				this.$emit('slideclick', item);
			}
		}
	}
</script>

<style scoped>
	.card-box {
		position: absolute;
		height: 980rpx;
		width: 580rpx;
		top: 0;
		box-shadow: 0 0 20rpx #555;
		transition: z-index 2s easc;
		border-radius: 20rpx;
	}

	.main-img {
		width: 100%;
		height: 100%;
		border-radius: 20rpx;
	}

	.lookmore {
		position: absolute;
		width: 60vw;
		height: 53rpx;
		bottom: 95rpx;
		left: 10%;
		z-index: 110;
	}

	.card-header {
		padding: 20upx;
		display: flex;
		flex-direction: row;

	}
	
	.card-right {
		padding-left: 10upx;
	}

	.right-top-name {
		color: #FFF;
		/* font-family: PingFangSC-Medium; */
		font-size: 30upx;
	}

	.header-right-bottom {
		margin-top: 4upx;
		font-size: 20upx;
		line-height: 10upx;
		color: #999;

		
	}
	.header-right-point {
		display: inline-block;
		padding: 10upx;
		font-weight: 700;
	}
</style>
