<!-- 基于HTML5+ API获取手机通讯录联系人（姓名，手机号） -->
<template>
	<view class="page">
		<button type="primary" @tap="getContacts">获取联系人</button>
		<block  v-for="(item,index) in list" :key="index">
		    <view>{{item.displayName}}</view>
				<block v-for="(subitem,idx) in item.phoneNumbers" :key="idx">
				    <view>{{subitem.value}}</view>
				</block>
		</block>
	</view>
</template>

<script>
	var Contacts
	export default {
		data() {
			return {
				list: []
			}
		},
		onShow() {
			uni.setNavigationBarTitle({
					title: '通讯录联系人'
			});
		},
		methods: {
			getContacts: function() {
				var that = this
				// 获取通讯录对象
				plus.contacts.getAddressBook( plus.contacts.ADDRESSBOOK_PHONE, function( addressbook ) {
					uni.showToast({
					    title: '获取通讯录对象成功',
					    duration: 2000
					})
					console.log('获取通讯录对象成功')
					console.log(addressbook)
					// 查找联系人
					addressbook.find(["displayName","phoneNumbers"],function(contacts){
						uni.showToast({
						    title: '获取联系人成功',
						    duration: 2000
						})
						console.log('获取联系人成功')
						console.log(JSON.stringify(contacts))
						that.list = contacts
					}, function () {
						uni.showToast({
						    title: '获取联系人失败',
						    duration: 2000
						})
					},{multiple:true});
				}, function ( e ) {
					uni.showToast({
					    title: '获取通讯录对象失败:' + e.message,
					    duration: 2000
					})
				});
			}
		}
	}
</script>

<style>
	.page{
		padding: 20upx;
	}
</style>
