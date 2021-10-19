<template>
	<view class="hot-container">
		<image class="logo" src="@/static/images/logo.png" mode="aspectFit"></image>
		<view class="search-box"><my-search placeholderText="自定义搜索组件"></my-search></view>
		<view class="tab-sticky"><my-tabs :tabData="tabData" :defaultIndex="currentIndex" @tabClick="onTabClick"></my-tabs></view>

		<!-- list 视图 -->
		<!--    1.使用 mock 数据，构建 List 的基本结构
				2.美化 item 样式
				3.根据 tab 的切换，获取真实数据
				4.渲染真实数据
				5.通过 swiper 改造List
				6.完成 swiper 和 tabs 的联动效果 -->

		<swiper class="swiper" :current="currentIndex" :style="{ height: currentSwiperHeight + 'px' }" @animationfinish="onSwiperEnd" @change="onSwiperChange">
			<swiper-item class="swiper-item" v-for="(tabItem, tabIndex) in tabData" :key="tabIndex">
				<view>
					<uni-load-more status="loading" v-if="isLoading" />
					<block v-else>
						<hot-list-item
							:class="'hot-list-item-' + tabIndex"
							v-for="(item, index) in listData[tabIndex]"
							:key="index"
							:data="item"
							:ranking="index + 1"
						></hot-list-item>
					</block>
				</view>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
import { getHotTabs, getHotListFromTabType } from 'api/hot';
export default {
	data() {
		return {
			tabData: [],
			currentIndex: 0,
			//以 index 为 key
			listData: {},
			isLoading: true,
			currentSwiperHeight: 0,
			swiperHeightData: {},
			currentPageScrollTop:0
		};
	},
	created() {
		this.getHotTabs();
	},
	onPageScroll(res) {
		this.currentPageScrollTop = res.scrollTop;
	},
	methods: {
		/**
		 * 获取热搜标题数据
		 */
		async getHotTabs() {
			// uniapp 支持 async await
			const { data: res } = await getHotTabs();
			this.tabData = res.list;

			// 获取列表数据
			this.loadHotListFromTab();
		},

		async loadHotListFromTab() {
			if (!this.listData[this.currentIndex]) {
				this.isLoading = true;
				const id = this.tabData[this.currentIndex].id;
				const { data: res } = await getHotListFromTabType(id);
				this.listData[this.currentIndex] = res.list;
				this.isLoading = false;
				setTimeout(async () => {
					this.currentSwiperHeight = await this.getCurrentSwiperHeight();
					console.log(this.currentSwiperHeight);
					this.swiperHeightData[this.currentIndex] = this.currentSwiperHeight;
				}, 0);
			} else {
			}
		},

		onTabClick(index) {
			this.currentIndex = index;
			// this.loadHotListFromTab();
		},

		onSwiperChange(e) {
			console.log(e);
			if(this.currentPageScrollTop > 130){
				uni.pageScrollTo({
					scrollTop:130
				});
			}
			this.currentIndex = e.detail.current;
		},
		onSwiperEnd() {
			if (!this.listData[this.currentIndex]) {
				this.loadHotListFromTab();
				return;
			}
			this.currentSwiperHeight = this.swiperHeightData[this.currentIndex];
		},
		getCurrentSwiperHeight() {
			return new Promise((resolve, reject) => {
				let sum = 0;
				const query = uni.createSelectorQuery().in(this);
				query
					.selectAll(`.hot-list-item-${this.currentIndex}`)
					.boundingClientRect(res => {
						res.forEach(item => {
							sum += item.height;
						});
						resolve(sum);
					})
					.exec();
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.hot-container {
	background-color: $uni-bg-color;
	.logo {
		width: 100%;
		height: 80px;
	}
	.search-box {
		padding: 0 16px;
		margin-bottom: $uni-spacing-col-base;
	}
	.tab-sticky{
		position: sticky;
		z-index: 99;
		top: 0;
	}
}
</style>
