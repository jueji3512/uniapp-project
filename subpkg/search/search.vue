<template>
	<view>
		<view class="search-box">
			<uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
		</view>
		<!-- 商品搜索结果建议 -->
		<view class="sugg-list" v-if="searchList.length !== 0">
			<view class="sugg-item" v-for="(item, i) in searchList" :key="i" @click="gotoDetail(item.goods_id)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
		<!-- 搜索历史 -->
		<view class="history-box" v-else>
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
			</view>
			<view class="history-list">
				<uni-tag :text="item" v-for="(item, i) in historyList" :key="i"></uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		onLoad() {
			this.historyList = JSON.parse(uni.getStorageSync('keyWord') || '[]')
		},
		data() {
			return {
				timer: null,
				// 搜索关键词
				keyWord: '',
				searchList: [],
				// 搜索历史
				historyList: []
			};
		},
		methods: {
			input(e) {
				clearTimeout(this.timer)
				this.timer = setTimeout(() => {
					// 防抖处理，500ms内未继续输入则为keyWord赋值
					this.keyWord = e
					this.getSearchList()
				}, 500)
			},
			// 搜索商品
			async getSearchList() {
				if (this.keyWord === '') {
					this.searchList = []
					return
				}
				const {
					data: res
				} = await uni.$http.get('/api/public/v1/goods/qsearch', {
					query: this.keyWord
				})
				if (res.meta.status !== 200) return uni.$showMsg()
				this.searchList = res.message
				this.saveSearchHistory()
			},
			// 点击跳转到商品详情
			gotoDetail(goods_id) {
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
				})
			},
			// 保存搜索历史
			saveSearchHistory() {
				this.historyList.unshift(this.keyWord)
				const set = new Set(this.historyList)
				this.historyList = [...set]
				// 将搜索历史存储到本地
				uni.setStorageSync('keyWord', JSON.stringify(this.historyList))
			},
			// 删除搜索历史
			cleanHistory() {
				this.historyList = []
				uni.setStorageSync('keyWord', '[]')
			}
		},
	}
</script>

<style lang="scss">
	.search-box {
		position: sticky;
		top: 0;
		z-index: 999;
	}

	.sugg-list {
		padding: 0 5px;

		.sugg-item {
			font-size: 12px;
			padding: 13px 0;
			border-bottom: 1px solid #efefef;
			display: flex;
			align-items: center;
			justify-content: space-between;

			.goods-name {
				// 文字不允许换行（单行文本）
				white-space: nowrap;
				// 溢出部分隐藏
				overflow: hidden;
				// 文本溢出后，使用 ... 代替
				text-overflow: ellipsis;
				margin-right: 3px;
			}
		}
	}

	.history-box {
		padding: 0 5px;

		.history-title {
			display: flex;
			justify-content: space-between;
			align-items: center;
			height: 40px;
			font-size: 13px;
			border-bottom: 1px solid #efefef;
		}

		.history-list {
			display: flex;
			flex-wrap: wrap;

			.uni-tag {
				margin-top: 5px;
				margin-right: 5px;
			}
		}
	}
</style>
