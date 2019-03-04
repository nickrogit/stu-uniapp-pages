<template>
	<view class="bluetooth">
		<!-- 发送数据 -->
		<view class="send" v-if="send_data_onoff">
			<view class="uni-textarea">
				<textarea placeholder-style="color:#F76260" v-model="send_data_list" placeholder="请输入发送数据" />
				<view @tap="send_data" class="fasong">发送</view>
            </view>
        </view>
        <!-- 没有匹配的蓝牙设备 -->
        <view class="no_match_bluetooth_list" v-if="no_match_list.length!=0">
            <view class="h1">已搜索的蓝牙：</view>
            <block  v-for="(item,index) in no_match_list" :key="index">
                <view class="uni-list" @tap="search_bluetooth(item.SN)">
                    <view class="uni-list-cell" hover-class="uni-list-cell-hover">
                        <view class="uni-list-cell-navigate uni-navigate-right uni-media-list ">
                            <view class="uni-media-list-body">
                                <view>蓝牙名称:{{item.name}}</view>
                                <view>SN:{{item.SN}}</view>
                            </view>
                        </view>
                    </view>
                </view>
            </block>
        </view>
        
        <!-- 已匹配的蓝牙设备 -->
        <view class="no_match_bluetooth_list" v-if="match_list.length!=0">
            <view class="h1">已匹配的蓝牙：</view>
            <block  v-for="(item,index) in match_list" :key="index">
                <view class="uni-list" @tap="print(item.SN)">
                    <view class="uni-list-cell" hover-class="uni-list-cell-hover">
                        <view class="uni-list-cell-navigate uni-navigate-right uni-media-list ">
                            <view class="uni-media-list-body">
                                <view>蓝牙名称:{{item.name}}</view>
                                <view>SN:{{item.SN}}</view>
                            </view>
                        </view>
                    </view>
                </view>
            </block>
        </view>
        
        
        
        <view class="btn">
            <view class="btn_1" @tap="open_bluetooth">打开蓝牙</view>
            <view class="btn_2" @tap="close_bluetooth">关闭蓝牙</view>
            <view class="btn_3" @tap="search_bluetooth">搜索蓝牙</view>
        </view>
    </view>
</template>

<script>
    var main, Context, BluetoothManager, BluetoothAdapter, BManager, BAdapter,BluetoothDevice,IntentFilter,bluetoothSocket,device;
    export default {
        data() {
            return {
                bArray:[],//用于搜索蓝牙去重用的
                no_match_list:[],//没有配对的蓝牙列表
                match_list:[],//已配对的蓝牙列表
                send_data_onoff:false,
                send_data_list:'测试打印数据',//要发送的数据
            };
        },
        onShow() {
            //获取android应用Activity对象
            main = plus.android.runtimeMainActivity();
            Context = plus.android.importClass("android.content.Context");
            BManager = main.getSystemService(Context.BLUETOOTH_SERVICE);
            //蓝牙适配器
            BluetoothAdapter = plus.android.importClass("android.bluetooth.BluetoothAdapter");
            //蓝牙本地适配器
            BAdapter = BluetoothAdapter.getDefaultAdapter();
            //蓝牙设备
            BluetoothDevice = plus.android.importClass('android.bluetooth.BluetoothDevice');
            //过滤器
            IntentFilter = plus.android.importClass('android.content.IntentFilter');
        },
        methods:{
            /**
            * 根据蓝牙地址，连接设备
            * @param {Object} address
            * @return {Boolean}
            */
            print(address){
                console.log(address)
                let that=this
				
				uni.showLoading({
				    title: '蓝牙连接中...'
				});
				that.print_bluetooth(address)
				
                /* uni.showModal({
                    title: '提示',
                    content: '是连接此蓝牙？',
                    success: function (res) {
                        if (res.confirm) {
                            console.log('用户点击确定');
                            uni.showLoading({
                                title: '蓝牙连接中...'
                            });
                            that.print_bluetooth(address)
                        }
                    }
                }); */
            },
            //连接蓝牙
            print_bluetooth(mac_address){
                let that=this
                uni.hideLoading()
                let UUID = plus.android.importClass("java.util.UUID");
                let uuid = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");
                device = BAdapter.getRemoteDevice(mac_address);
                plus.android.importClass(device);
                bluetoothSocket = device.createInsecureRfcommSocketToServiceRecord(uuid); // bluetoothSocket = device.createInsecureRfcommSocketToServiceRecord(uuid);
                plus.android.importClass(bluetoothSocket);
				
				var islink = bluetoothSocket.isConnected()
				console.log('islink:')
				console.log(islink)
				uni.showToast({
				    title: '准备连接',
				    duration: 2000
				})
                if (!bluetoothSocket.isConnected()) {
					uni.showToast({
					    title: '检测到设备未连接，尝试连接....',
					    duration: 2000
					})
                    console.log('检测到设备未连接，尝试连接....');
					try {
                        bluetoothSocket.connect();
                    } catch (e) {
						uni.showToast({
							title: '准备出错',
							duration: 2000
						})
                        console.log('连接出错')
						return
                    }
                }
				uni.showToast({
				    title: '连接成功',
				    duration: 2000
				})
                console.log('设备已连接');
                // 打开输入发送数据
                that.send_data_onoff=true;
                
            },
            //发送蓝牙数据
            send_data(){
                let that=this
                console.log()
                if (bluetoothSocket.isConnected()) {
					uni.showToast({
					    title: '已连接，打印',
					    duration: 2000
					})
                    console.log('已连接，打印')
                    var outputStream = bluetoothSocket.getOutputStream();
                    plus.android.importClass(outputStream);
                    var string = that.send_data_list
                    var bytes = plus.android.invoke(string, 'getBytes', 'gbk');//创建输出流失败
                    outputStream.write(bytes);
                     outputStream.flush();
                     device = null //这里关键
                     bluetoothSocket.close(); //必须关闭蓝牙连接否则意外断开的话打印错误
                } else {
					uni.showToast({
					    title: '未连接，不打印',
					    duration: 2000
					})
					console.log('未连接')
				}
            },
            open_bluetooth(){
                if(!BAdapter.isEnabled()) {
                    BAdapter.enable();//启动蓝牙
                    uni.showToast({
                        title: '蓝牙已启动',
                        duration: 2000
                    })
                }
            },
            //关闭蓝牙
            close_bluetooth(){
                let that =this
                //关闭蓝牙都把之前的清空
                that.no_match_list=[];
                that.match_list=[]
                if (BAdapter.isEnabled()) {
                    BAdapter.disable();//关闭蓝牙
                    uni.showToast({
                        title: '蓝牙已关闭',
                        duration: 2000
                    })
                }
            },
            //获取已匹配蓝牙设备
            bluetooth_list(){
                let that=this
                //先清空
                that.match_list=[]
                
                var lists = BAdapter.getBondedDevices();
                plus.android.importClass(lists);
                var iterator = lists.iterator();
                plus.android.importClass(iterator);
                while (iterator.hasNext()) {
                    var d = iterator.next();
                    plus.android.importClass(d);
                    let arr={
                        "name": d.getName(),
                        "SN": d.getAddress()
                    }
                    that.match_list.push(arr)
					uni.hideLoading()
                }
            },
            //搜索没匹配的蓝牙设备
            search_bluetooth(address){
                let that=this
                //判断蓝牙是否开启
                if (!BAdapter.isEnabled()) {
                    uni.showToast({
                        title: '请先打开蓝牙',
                        duration: 3000
                    });
                    return
                }
                if(address.length!=undefined){
                    uni.showModal({
                        title: '提示',
                        content: '是否配对此蓝牙？',
                        success: function (res) {
                            if (res.confirm) {
                                console.log('用户点击确定');
                                uni.showLoading({
                                    title: '蓝牙匹配中...'
                                });
                                that.search_pipei(address)
                            }
                        }
                    });
                }else{
                    uni.showLoading({
                        title: '蓝牙搜索中...'
                    });
                    that.search_pipei()
                }
            },
            //搜索和匹配蓝牙
            search_pipei(address){
				
                let that=this
                //获取已匹配的蓝牙
                that.bluetooth_list()
                //每次搜索都把之前的清空
                that.bArray=[];
                that.no_match_list=[];
                
                var filter = new IntentFilter();
                var BDevice = new BluetoothDevice();
                BAdapter.startDiscovery(); //开启搜索
                var receiver = plus.android.implements('io.dcloud.android.content.BroadcastReceiver', {
                    onReceive: function(context, intent) { //回调
                        try {               
                            plus.android.importClass(intent); //通过intent实例引入intent类
                            if(intent.getAction() == "android.bluetooth.adapter.action.DISCOVERY_FINISHED") {
                                uni.hideLoading()
                                main.unregisterReceiver(receiver); //取消监听 
                            } else {
                                //从Intent中获取设备对象
                                BDevice= intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
                                //配对蓝牙
                                if (address == BDevice.getAddress()) {
                                    if (BDevice.createBond()) { //配对命令.createBond()
                                        console.log("配对成功");
										uni.showToast({
										    title: '配对成功',
										    duration: 2000
										})
                                        uni.hideLoading()
                                    }
                                }
                                if(BDevice == null) {
                                    main.unregisterReceiver(receiver); //取消监听
                                    uni.hideLoading()
                                    //获取已匹配的蓝牙
                                    that.bluetooth_list()
                                    return;
                                }
                                var name=BDevice.getAddress()+BDevice.getName();
                                if(that.bArray.indexOf(name) == -1){     //去重
                                    that.bArray.push(name);//用于去重的 
                                    let arr={
                                        "name": BDevice.getName(),
                                        "SN": BDevice.getAddress()
                                    }
                                    that.no_match_list.push(arr)
                                    console.log(JSON.stringify(that.no_match_list))
                                }

                            }
                        } catch(e) {
                            console.error(e);
                        }
                    }
                });
                filter.addAction(BDevice.ACTION_FOUND);
                filter.addAction(BAdapter.ACTION_DISCOVERY_STARTED);
                filter.addAction(BAdapter.ACTION_DISCOVERY_FINISHED);
                filter.addAction(BAdapter.ACTION_STATE_CHANGED);
                main.registerReceiver(receiver, filter); //注册监听 
            },
        }
    }
</script>

<style>
	.bluetooth{
		padding: 20upx 20upx 120upx;
	}
	
    .bluetooth .h1{
        padding: 20upx 0;color: #1482D1;border-top: 1upx solid #EFEFF4;
    }
    
    .bluetooth .btn{
        display:flex;position:fixed;bottom:0upx;padding:20upx 10upx;justify-content:space-around;width: 100%;color:#fff;border-top:1px solid #f2f2f2;background-color: #fff;
        
    }
		.bluetooth .btn view{
		    width: 200upx;height: 80upx;line-height:80upx;text-align: center;border-radius:10upx;
		}
		.bluetooth .btn .btn_1{
		    background-color: #009BDE;
		}
		.bluetooth .btn .btn_2{
		    background-color: red;
		}
		.bluetooth .btn .btn_3{
		    background-color: #3CB371;
		}
    .bluetooth .uni-textarea textarea{
					border: 1px solid #EFEFF4;border-radius: 10px;width: 100%;padding: 10upx;box-sizing: border-box;
			}
		.bluetooth .uni-textarea .fasong{
		    width: 100%;padding:15upx 10upx;box-sizing: border-box;border-radius: 10px;background: #1482D1;text-align: center;color: white;margin: 10px 0;
		}

</style>
