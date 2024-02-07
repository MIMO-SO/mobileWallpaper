<template>
	<div>
		<view class="container">
			<u-navbar :title="titleName" :autoBack="true" bgColor="#000" leftIconColor="#fff" titleStyle="color: #fff" placeholder="true">
			</u-navbar>
			<swiper :current="currentIndex" @change="swiperChange" class="image-swiper" duration="300"
				:style="{height: '240px', 'margin-top': '50%'}">
				<swiper-item v-for="(item, index) in imageList" :key="index">
					<scroll-view scroll-x="true" scroll-y="true" class="image-container"
						:style="{transform: `scale(${scaleValue})`}">
						<image :src="item" mode="aspectFit" class="image"></image>
					</scroll-view>
				</swiper-item>
			</swiper>
		</view>
		<view class="description">{{ info.name_zh }}</view>
		<view class="dingwei">
			<u-row>
				<u-col span="3">
					<view class="demo-layout">
						<u-icon label="返回" @click="Back" labelSize="13" labelColor="#fff" size="20"
							name="https://shopimges.oss-cn-hangzhou.aliyuncs.com/wallpaper/fh.png" labelPos="bottom"
							space="5"></u-icon>
					</view>
				</u-col>
				<u-col span="3">
					<view class="demo-layout">
						<u-icon label="点赞" @click="showToast" labelSize="13" labelColor="#fff" size="20"
							name="https://shopimges.oss-cn-hangzhou.aliyuncs.com/wallpaper/dz.png" labelPos="bottom"
							space="5"></u-icon>
					</view>
				</u-col>
				<u-col span="3">
					<view class="demo-layout">
						<u-icon label="下载" @click="downloadFile" labelSize="13" labelColor="#fff" size="20"
							name="https://shopimges.oss-cn-hangzhou.aliyuncs.com/wallpaper/xz.png" labelPos="bottom"
							space="5"></u-icon>
					</view>
				</u-col>
				<u-col span="3">
					<view class="demo-layout">
						<button data-name="shareBtn" open-type="share">
							<u-icon label="分享" @click="onShareAppMessage" labelSize="13" labelColor="#fff" size="20"
								name="https://shopimges.oss-cn-hangzhou.aliyuncs.com/wallpaper/fx.png" labelPos="bottom"
								space="5"></u-icon>
						</button>
					</view>
				</u-col>
			</u-row>
		</view>
		<u-toast ref="uToast"></u-toast>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				baseurl: "https://scene.liuyufeng.fun/app",
				imageList: [], // 图片列表数据
				currentIndex: 0, // 当前显示的图片索引
				scaleValue: 1, // 缩放值
				info: null,
			};
		},
		computed: {
			titleName: function() {
				return this.currentIndex + 1 + '/' + this.imageList.length
			}
		},
		onLoad(options) {
			this.info = JSON.parse(decodeURIComponent(options.info));
			let url = this.baseurl + '/getAllPictureByCode?code=' + this.info.code;
			uni.request({
				url: url,
				success: (res) => {
					let length = res.data.list.length
					this.getImageCache(this.info.preview_url_cache)
					for (let i = 0; i < length; i++) {
						this.getImageCache(res.data.list[i]);
					}
					this.currentIndex = options.index <= length ? +options.index : 0;
				},
				fail: () => {
					this.getImageCache(this.info.preview_url_cache);
				}
			})
			uni.showShareMenu({
				withShareTicket: true,
				menus: ["shareAppMessage", "shareTimeline"]
			})
		},
		methods: {
			getImageCache(url) {
				let cache = uni.getStorageSync('imageCache:' + url);
				if (cache) {
					// 缓存中有图片，直接使用缓存
					this.imageList.push(cache.path);
				} else {
					// 缓存中没有图片或者路径不相同，重新加载图片
					this.imageList.push(url);
					this.loadImage(url);
				}
			},
			// 加载图片
			loadImage(url) {
				// 获取图片信息
				uni.getImageInfo({
					src: url,
					success: (res) => {
						// 图片加载成功后将图片信息保存到本地缓存
						uni.setStorageSync('imageCache:' + url, {
							path: res.path
						});
					},
					fail: (error) => {
						console.log(error);
					}
				});
			},
			swiperChange(event) {
				this.currentIndex = +event.detail.current;
			},
			onShareAppMessage() {
				let index = +this.currentIndex
				return {
					title: '沄樊-' + this.info.name_zh,
					path: '/pages/home/photo-viewer?index=' + +index + '&info=' + encodeURIComponent(JSON
						.stringify(this
							.info)),
					imageUrl: this.imageList[+this.currentIndex]
				}
			},
			Back() {
				if (getCurrentPages().length > 1) {
					uni.navigateBack({
						delta: 1,
					})
				} else {
					uni.redirectTo({
						url: '/pages/home/index'
					})
				}
			},
			showToast() {
				this.$refs.uToast.show({
					...{
						type: 'success',
						title: '点赞成功',
						message: "点赞成功",
						iconUrl: 'https://cdn.uviewui.com/uview/demo/toast/success.png'
					},
					complete() {}
				})
			},
			downloadFile() {
				uni.showLoading({
					title: '正在保存图片'
				});
				// 获取图片信息
				let url = this.imageList[+this.currentIndex];
				if (url.indexOf("https://www.sony.net") == 0) {
					if (this.currentIndex == 0) {
						url = this.baseurl + "/photo/fp_hd?code=" + this.info.code + "&preview=" + this.info
							.preview
					} else {
						url = this.baseurl + "/photo/download?url=" + url
					}
					console.log(url)
					uni.downloadFile({
						url: url,
						success: (res) => {
							let benUrl = res.tempFilePath
							console.log(benUrl)
							//图片保存到本地相册
							this.saveImage(benUrl);
						},
						fail: (error) => {
							console.log(error)
							uni.hideLoading()
						}
					})
				} else {
					this.saveImage(url)
				}
			},
			saveImage(benUrl) {
				//图片保存到本地相册
				uni.saveImageToPhotosAlbum({
					filePath: benUrl,
					//授权成功，保存图片
					success: function(data) {
						uni.hideLoading()
						uni.showModal({
							title: '温馨提示',
							content: '保存相册成功',
							showCancel: false,
							success(res) {
								if (res.confirm) {

								}
							}
						});
					},
					//授权失败
					fail: function(err) {
						if (err.errMsg) {
							console.log(err.errMsg)
							//重新授权弹框确认
							uni.showModal({
								title: '提示',
								content: '无权限，请打开下载权限后再试！',
								showCancel: false,
								success(res) {
									if (res.confirm) {
										//重新授权弹框用户点击了确定
										uni.openSetting({
											//进入小程序授权设置页面
											success(settingdata) {
												if (settingdata
													.authSetting[
														'scope.writePhotosAlbum'
													]) {
													//用户打开了保存图片授权开关
													uni.saveImageToPhotosAlbum({
														filePath: benUrl,
														success: function(
															data
														) {
															uni.hideLoading()
															that.adShow =
																true
														}
													});
												} else {
													//用户未打开保存图片到相册的授权开关
													uni.showModal({
														title: '温馨提示',
														content: '授权失败，请稍后重新获取',
														showCancel: false
													});
												}
											}
										});
									}
								}
							});
						}
						uni.hideLoading()
					}
				});
			}
		}
	};
</script>

<style>
	page {
		background-color: #000;
	}

	this .container {
		/* display: flex;
		flex-direction: column;
		height: 100%; */
	}

	.header {
		color: #fff;
		text-align: center;
		padding-top: 15%;
	}

	.image-swiper {
		flex: 1;
		overflow: hidden;
	}

	.image-container {
		width: 100vw;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.image {
		width: 100%;
		max-width: 100vw;
		max-height: 100vh;
		object-fit: contain;
	}

	.description {
		text-align: center;
		color: #fff;
		padding: 10px;
	}

	.dingwei {
		position: fixed;
		bottom: 5vh;
		left: 5vw;
		width: 80vw;
		background-color: #0b0f12;
		opacity: 0.6;
		padding: 10rpx 30rpx;
		border-radius: 50rpx;
		text-align: center;
	}

	button {
		border: none;
		background-color: transparent;
		outline: none;
	}
</style>