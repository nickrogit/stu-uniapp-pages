<template>
	<view>
		<button type="primary" size="mini" @tap="openBluetoothAdapter">打开蓝牙模块</button>
		<button type="primary" size="mini" @tap="getBluetoothState">获取蓝牙状态</button>
		<button type="primary" size="mini" @tap="closeBluetoothAdapter">关闭蓝牙模块</button>
		<button type="primary" size="mini" @tap="listenerStateChange">监听状态变化</button>
		<view style="height: 20upx;"></view>
		<button type="primary" size="mini" @tap="startBluetoothDiscovery">开始搜索蓝牙</button>
		<button type="primary" size="mini" @tap="listenerDeviceFound">监听发现新设备</button>
		<button type="primary" size="mini" @tap="getBluetoothDevices">获取已搜索到的蓝牙设备</button>
		<button type="primary" size="mini" @tap="stopBluetoothDiscovery">结束搜索蓝牙</button>
		<view style="height: 20upx;"></view>
		<button type="primary" size="mini" @tap="createConnection">连接蓝牙设备</button>
		<button type="primary" size="mini" @tap="getConnectedDevices">获取已连接的蓝牙设备</button>
		<button type="primary" size="mini" @tap="closeConnection">断开蓝牙设备</button>
		<view style="height: 20upx;"></view>
		<button type="primary" size="mini" @tap="listenerConnection">监听连接状态变化</button>
		
		<view>搜索到的蓝牙设备</view>
		<block  v-for="(item,index) in devicesList" :key="index">
		    <view @tap="createConnection(item.deviceId)">{{item.deviceId}} ：{{item.name}}</view>
		</block>
		<view>已连接的蓝牙设备</view>
		<block  v-for="(item,index) in devicesList2" :key="index">
		    <view>{{item.deviceId}} ：{{item.name}}</view>
		</block>
	</view>
</template>

<script>
	var Contacts
	export default {
		data() {
			return {
				deviceId: '',
				devicesList: [],
				devicesList2: []
			}
		},
		onShow() {
			uni.setNavigationBarTitle({
				title: '蓝牙测试'
			});
		},
		methods: {
			// 打开蓝牙模块
			openBluetoothAdapter(){
				var that = this
				plus.bluetooth.openBluetoothAdapter({
					success:function(e){
						console.log('open success: '+JSON.stringify(e));
					},
					fail:function(e){
						console.log('open failed: '+JSON.stringify(e));
					}
				});
			},
			// 获取蓝牙状态
			getBluetoothState(){
				var that = this
				plus.bluetooth.getBluetoothAdapterState({
					success:function(e){
						console.log('state success: '+JSON.stringify(e));
					},
					fail:function(e){
						console.log('state failed: '+JSON.stringify(e));
					}
				});
			},
			// 关闭蓝牙模块
			closeBluetoothAdapter(){
				var that = this
				plus.bluetooth.closeBluetoothAdapter({
					success:function(e){
						console.log('close success: '+JSON.stringify(e));
					},
					fail:function(e){
						console.log('close failed: '+JSON.stringify(e));
					}
				});
			},
			// 监听状态变化
			listenerStateChange(){
				var that = this
				plus.bluetooth.onBluetoothAdapterStateChange(function(e){
					console.log('state changed: '+JSON.stringify(e));
				});
			},
			// 开始搜索蓝牙
			startBluetoothDiscovery(){
				var that = this
				plus.bluetooth.openBluetoothAdapter({
					success:function(e){
						console.log('open success: '+JSON.stringify(e));
						plus.bluetooth.startBluetoothDevicesDiscovery({
							success:function(e){
								console.log('start discovery success: '+JSON.stringify(e));
							},
							fail:function(e){
								console.log('start discovery failed: '+JSON.stringify(e));
							}
						});
					},
					fail:function(e){
						console.log('open failed: '+JSON.stringify(e));
					}
				});
			},
			// 监听发现新设备
			listenerDeviceFound(){
				var that = this
				plus.bluetooth.onBluetoothDeviceFound(function(e){
					var devices = e.devices;
					// console.log('device found: '+e.length);
					for(var i in devices){
						console.log(i+': '+JSON.stringify(devices[i]));
					}
				});
			},
			// 获取已搜索到的蓝牙设备
			getBluetoothDevices(){
				var that = this
				plus.bluetooth.getBluetoothDevices({
					success:function(e){
						var devices = e.devices;
						that.devicesList = devices
						console.log('get devices success: '+e.length);
						for(var i in devices){
							console.log(i+': '+JSON.stringify(devices[i]));
						}
						that.stopBluetoothDiscovery() //搜索后关闭
					},
					fail:function(e){
						console.log('get devices failed: '+JSON.stringify(e));
					}
				});
			},
			// 结束搜索蓝牙
			stopBluetoothDiscovery(){
				var that = this
				plus.bluetooth.stopBluetoothDevicesDiscovery({
					success:function(e){
						console.log('stop discovery success: '+JSON.stringify(e));
					},
					fail:function(e){
						console.log('stop discovery failed: '+JSON.stringify(e));
					}
				});
			},
			// 连接蓝牙设备
			createConnection(deviceId){
				console.log('开始连接：' + deviceId)
				var that  = this
				that.deviceId = deviceId
				plus.bluetooth.createBLEConnection({
					deviceId:that.deviceId,
					success:function(e){
						console.log('create connection success: '+JSON.stringify(e));
					},
					fail:function(e){
						console.log('create connection failed: '+JSON.stringify(e));
					}
				});
			},
			// 获取已连接的蓝牙设备
			getConnectedDevices(){
				var that = this
				plus.bluetooth.getConnectedBluetoothDevices({
					success:function(e){
						var devices = e.devices;
						that.devicesList2 = devices
						console.log('connected devices success: '+e.length);
						for(var i in devices){
							console.log(i+': '+JSON.stringify(devices[i]));
						}
					},
					fail:function(e){
						console.log('connected devices failed: '+JSON.stringify(e));
					}
				});
			},
			// 断开蓝牙设备
			closeConnection(){
				var that  = this
				plus.bluetooth.closeBLEConnection({
					deviceId:that.deviceId,
					success:function(e){
						console.log('close success: '+JSON.stringify(e));
					},
					fail:function(e){
						console.log('close failed: '+JSON.stringify(e));
					}
				});
			},
			// 监听蓝牙设备连接状态
			listenerConnection(){
				var that = this
				plus.bluetooth.onBLEConnectionStateChange(function(e){
					console.log('connection state changed: '+JSON.stringify(e));
				});
			}
		}
	}
</script>

<style>

</style>
