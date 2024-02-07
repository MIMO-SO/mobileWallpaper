<template>
	<view>
		<view class="nejing">
			<image src="http://shopimges.oss-cn-hangzhou.aliyuncs.com/wallpaper/hend.jpg" mode="widthFix" class="bend">
			</image>

			<view class="search-container" :style="{ height: navBarHeight + 'px', paddingTop: statusBarHeight + 'px' }">
				<u--input prefixIcon="search" type="text" placeholder="搜索(开发中)..." @click="searchClicked" shape="circle"
					customStyle="background-color: rgba(255, 255, 255, 0.7);" />
			</view>

			<view class="body">
				<view class="body_be">
					<u-tabs :list="munlist" lineHeight="0"
						:activeStyle="{ color: '#ec7e00',fontWeight: 'bold',transform: 'scale(1.05)'  }" @click="click">
					</u-tabs>
				</view>
				<view v-for="(value,key) in imageList" :key="key">
					<view class="demo-layout-body" @click="jom(value)">
						<view class="lin-body">
							<image :src="value.preview_url" mode="aspectFill"></image>
						</view>

					</view>
				</view>
				<view v-if="imageList.length==0">
					<u-empty mode="data" icon="http://cdn.uviewui.com/uview/empty/data.png">
					</u-empty>
				</view>
			</view>
		</view>
		
		<u-back-top :scroll-top="scrollTop"></u-back-top>

		<u-tabbar :value="value6" @change="change5" :fixed="true" :placeholder="true"
			:safeAreaInsetBottom="true">
			<u-tabbar-item text="首页" icon="home"></u-tabbar-item>
			<u-tabbar-item text="我的" icon="account"></u-tabbar-item>
		</u-tabbar>
	</view>
</template>

<script>
	// const db = uniCloud.database()
	export default {
		data() {
			return {
				baseurl: "https://scene.liuyufeng.fun/app",
				list: [],
				list3: [],
				imageList: [],
				title: "",
				mtitle: "",
				munlist: [],
				page: 1,
				hasNext: true,
				statusBarHeight: 0,
				menuHeight: 0,
				navBarHeight: 0,
				value6: 0,
				scrollTop: 0
			}
		},
		onPageScroll(e) {
			this.scrollTop = e.scrollTop;
		},
		onLoad() {
			let that = this;

			// 获取系统信息
			let sysInfo = uni.getSystemInfoSync();
			that.statusBarHeight = sysInfo.statusBarHeight;

			// 获取胶囊信息
			let menu = uni.getMenuButtonBoundingClientRect();
			that.menuHeight = menu.height;
			that.navBarHeight = (menu.top - that.statusBarHeight) * 2 + that.menuHeight;

			uni.request({
				url: that.baseurl + "/getAllCountry",
				success(res) {
					if (res.data.length > 0) {
						that.munlist = that.munlist.concat(res.data);
						that.getlist();
					}
				}, fail: () => {
					uni.showLoading({
						title: '加载失败',
					});
				}
			})
			
		},
		onShow() {},
		methods: {
			change5(name) {
				if (name == 1) return uni.$u.toast('开发中，敬请期待')
				else this.value6 = name;
			},
			getImageCache(e) {
				e.preview_url_cache = e.preview_url;
				let cache = uni.getStorageSync('imageCache:' + e.preview_url);
				if (cache) {
					e.preview_url = cache.path;
					// 缓存中有图片，直接使用缓存
					this.imageList.push(e);
				} else {
					// 缓存中没有图片或者路径不相同，重新加载图片
					this.imageList.push(e);
					this.loadImage(e.preview_url);
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
			searchClicked() {
				console.log("搜索框被点击了");
			},
			getlist() {
				if (this.hasNext) {
					uni.request({
						url: this.baseurl + "/getAllPicture?country=" + this.mtitle + "&page=" + this.page +
							"&limit=9",
						success: (res) => {
							uni.hideLoading()
							if (res.data.length > 0) {
								for (let i = 0; i < res.data.length; i++) {
									this.getImageCache(res.data[i]);
								}
								// this.imageList = this.imageList.concat(res.data);
							} else if (res.data.length == 0) {
								this.hasNext = false
							}
						}
					})
				}
			},
			click(e) {
				this.currentTab = e.index;
				if (e.name != this.mtitle) {
					uni.showLoading({
						title: '数据加载中'
					});
					if (e.name != "全部") {
						this.page = 1;
						this.hasNext = true;
						this.imageList = [];
						this.mtitle = e.name
					} else {
						this.page = 1;
						this.hasNext = true;
						this.imageList = [];
						this.mtitle = ""
					}
					this.getlist();
				}
			},
			jom(e) {
				uni.navigateTo({
					url: 'photo-viewer?index=0&info=' + encodeURIComponent(JSON.stringify(e))
				})
			},
			handleSwiperChange(event) {
				this.currentTab = event.detail.current;
			},
		},
		onReachBottom() {
			this.page++
			this.getlist();
		}

	}
</script>

<style>
	page {
		background-color: #fff;

	}

	.nejing {
		position: relative;
	}

	.bend {
		width: 100%;
	}

	.body {
		position: relative;
		padding: 10rpx;
		margin-top: -110rpx;
		border-top-left-radius: 50rpx;
		border-top-right-radius: 50rpx;
		background-color: #fff;
	}

	.body_be {
		padding: 10rpx 0;
	}

	.demo-layout-body {
		width: calc(100% / 3);
		float: left;
	}

	.lin-body {
		padding: 5rpx 10rpx;
	}

	.lin-body image {
		width: 100%;
		height: 450rpx;
		border-radius: 15rpx;
	}

	.search-container {
		display: flex;
		align-items: center;
		justify-content: center;
		position: fixed;
		top: 0;
		left: 50%;
		transform: translate(-50%, 0);
		width: 65%;
		border-radius: 20rpx;
		z-index: 10;
		margin-left: -11%;
	}
</style>