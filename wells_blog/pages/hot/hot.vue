<template>
	<view class="hot-container">
		<image class="logo" src="@/static/images/logo.png" mode="aspectFit"></image>
		<view class="search-box"><my-search placeholderText="自定义搜索组件"></my-search></view>
		<my-tabs :tabData="tabData" :defaultIndex="0" @tabClick="onTabClick"></my-tabs>
		<!-- list 视图 -->
		<!--    1.使用 mock 数据，构建 List 的基本结构
				2.美化 item 样式
				3.根据 tab 的切换，获取真实数据
				4.渲染真实数据
				5.通过 swiper 改造List
				6.完成 swiper 和 tabs 的联动效果 -->
		<view><hot-list-item v-for="(item, index) in 50" :key="index"></hot-list-item></view>
	</view>
</template>

<script>
import { getHotTabs, getHotListFromTabType } from 'api/hot';
export default {
	data() {
		return {
			tabData: [],
			currentIndex: 0
		};
	},
	created() {
		this.getHotTabs();
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
			await getHotListFromTabType('0');
		},

		onTabClick(index) {
			this.currentIndex = index;
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
}
</style>
