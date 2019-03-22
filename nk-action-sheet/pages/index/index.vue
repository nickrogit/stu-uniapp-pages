<template>
	<view>
		<br/><br/><br/><br/>
		<button @click="showNkAction">打开操作菜单</button>
		<view>说明：</view>
		<view>
			支持副标题subName字段，无需求可置空或删除些字段即可。
		</view>
		<!-- 操作菜单 组件 -->
		<nkActionSheet :show="nkAction.show" :actions="nkAction.actions" @close="showNkAction" @cancel="showNkAction" @select="onSelect"></nkActionSheet>
		
	</view>
</template>

<script>
	import nkActionSheet from '@/components/nk-action-sheet.vue';
	export default {
		components: {nkActionSheet},
		data() {
			return {
				nkAction: {
					actions: [{
							name: "主标题01",
							subName: "副标题01",
							value: "1"
						},
						{
							name: "主标题02",
							subName: "副标题02",
							value: "2"
						},
						{
							name: "主标题03",
							subName: "",
							value: "3"
						}
					],
					show: false
				}
			};
		},
		onLoad() {},
		onShow() {
			uni.setNavigationBarTitle({
				title: "操作菜单-优化版"
			})
		},
		onReady() {},
		methods: {
			showNkAction: function() {
				this.nkAction.show = !this.nkAction.show;
			},
			onSelect: function(e) {
				console.log(e)
				this.nkAction.show = false
			},
		}
	}
</script>

<style scoped>
</style>
