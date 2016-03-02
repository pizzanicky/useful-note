#Bluetooth Low Energy笔记
低功耗蓝牙（BLE）也叫Bluetooth Smart，BLE无线技术基于Bluetooth 4.0规范，规范中定义了一套低功耗设备间的通讯协议，并不向下兼容经典蓝牙。  

下图是从苹果官网偷来的  
![图](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBTechnologyFramework_2x.png)

Android 4.3（API level 18）开始正式支持，iOS 7开始支持  

##基本概念：
###工作方式
BLE有两种工作方式，连接(connected)模式和广播（advertising）模式  

* 连接模式使用GATT层进行一对一数据传输
* 广播模式使用GAP层对听众进行广播  
定位主要用后者

###中心（Central）和外设（Peripheral）
*Peripheral*持有其他设备需要的数据(Server)，*Central*用外设提供的信息完成特定事务(Client)  
![图](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png)  

####中心发现并连接发送广播的外设
外设以广告包（Advertising packets）的形式把它拥有的部分数据广播出来，*广告包*是一个数据小bundle，含有外设的信息，如外设名和主要功能。“发广告”是BLE外设能被发现的主要途径。
####外设的数据结构
一个外设可包含一个或多个服务（Service），一个*服务*就是为了完成某个功能的一个数据集合及其相关行为，比如提供心率测量数据就是心率计的一个Service。  
而Service是由Characteristic（不知道中文用什么词合适，特征？特性？）或其他service组成，*characteristic*负责提供Service的详情。比如，心率数据Service的一个characteristic负责描述心率计的佩戴位置，另一个负责传送心率测量数据，见下图  
![pic](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBPeripheralData_Example_2x.png)
####中心访问外设的数据
中心和外设成功建立连接之后，就可以访问外设提供的所有Service和Characteristic（广告包有可能只包含了可用Service中的一部分），访问的方式可以是读或者写。比如，app可以向数字温控器获取室温，也可以写入一个设定值。

###Generic Attribute Profile（GATT）  
用来配置attribute收发的文件，定义了BLE在每种应用中如何工作。一个设备可以实现多个profile。目前BLE的所有应用配置都基于GATT
蓝牙技术联盟（SIG）为BLE定义了很多profile

###Attribute Protocol（ATT）
GATT都基于ATT构建。  
为BLE优化，使用尽可能少的字节数。每个attribute由一个128位的UUID（Universally Unique Identifier）字串来标识。ATT传输的attribute会被格式化为characteristic和service

###关于UUID
iBeacon广告包里发出的是proximity UUID，可以认为是iBeacon设备的标识符，然并卵，iOS并没有把这个UUID暴露给app，APP看到的只是一个Service UUID，由iOS设备在发现iBeacon时自动生成，并在范围内时保持。也就是说，同一个iPhone，在不同时段扫到同一个iBeacon的UUID有可能是完全不同的。  
那么proximity UUID有什么用？当然是定位，不过得按iOS的玩法来：只有在使用iOS的定位类CoreLocation的时候，app才能获得iBeacon的proximity UUID，并且前提是你要提前知道这个UUID。

##BLE Beacon-低功耗蓝牙信标
beacon中文叫法有：信标，基站等。低功耗beacon标准有好几种，有收费的也又免费公开的，各种beacon标准的广播包格式定义不同，iBeacon是其中的一种——不免费公开的
###iBeacon
苹果定义的标准，是最早的beacon标准，所以其他主流beacon大体参考其数据格式
####iBeacon数据格式
iBeacon广播由4部分组成：proximity UUID，Major，Minor，TX信号强度  
![pic](http://austinblackstoneengineering.com/wp-content/uploads/2015/02/iBeaconPacket.png)
####BLE广播数据格式
![pic](http://www.havlena.net/wp-content/uploads/ibeacon-packet.png)

##iOS
iOS可以通过CoreBluetooth和CoreLocation两种方式访问iBeacon。CoreBluetooth上面基本上概括了，下面记录的是CoreLocation的要点

###Region Monitoring
区域监视是CoreLocation Framework中的一部分，本身还分为基于地理位置和基于iBeacon的区域监视

以下是会导致区域监视不可用的原因：

* 设备硬件不支持
* 用户没有授权APP进行区域监视
* 用户在设置中关闭了定位服务
* 用户关闭了设备或APP的后台数据刷新
* 设备处于飞行模式，无法使用某些硬件

从iOS7开始，使用本功能前要记得调用`CLLocationManager`的[isMonitoringAvailableForClass:](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/index.html#//apple_ref/occ/clm/CLLocationManager/isMonitoringAvailableForClass:)方法和[authorizationStatus](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/index.html#//apple_ref/occ/clm/CLLocationManager/authorizationStatus)方法

`isMonitoringAvailableForClass:`方法会告诉你底层硬件是否支持，如果返回`YES`，则再调用`authorizationStatus`方法查询APP是否获准使用定位服务，如果返回结果是`kCLAuthorizationStatusAuthorized`，则设备进出注册区域时会收到越界通知，否则app收不到通知

**注意：**APP未被授权使用区域监视，不会妨碍APP对区域进行注册，这样如果用户后来通过了授权，app就自然能够收到越界通知了。另外，如果想在app未获授权试移除注册的区域，可以用`locationManager:didChangeAuthorizationStatus:`代理方法来获取授权状态的变化，然后合理移除注册区域

每个Beacon区域是用proximity UUID, Major, Minor来标识，其中Major和Minor不是必须的

创建并注册一个beacon区域：

```
- (void)registerBeaconRegionWithUUID:(NSUUID *)proximityUUID andIdentifier:(NSString*)identifier {
 
   // Create the beacon region to be monitored.
   CLBeaconRegion *beaconRegion = [[CLBeaconRegion alloc]
      initWithProximityUUID:proximityUUID
                 identifier:identifier];
 
   // Register the beacon region with the location manager.
   [self.locManager startMonitoringForRegion:beaconRegion];
}
```

注册成功之后立即生效，设备一旦发现该beacon，系统就会给app生成一个区域事件

###Beacon区域越界的处理
用户进入beacon区域，location manager调用`locationManager:didEnterRegion:`，离开区域时调用`locationManager:didExitRegion:`

设置[notifyOnEntry](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLRegion_class/index.html#//apple_ref/occ/instp/CLRegion/notifyOnEntry)和[notifyOnExit](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLRegion_class/index.html#//apple_ref/occ/instp/CLRegion/notifyOnExit)这两个属性可以指定app响应哪种越界信息，默认均为`YES`

也可以设置设备亮屏时才提醒用户，把[notifyEntryStateOnDisplay](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLBeaconRegion_class/index.html#//apple_ref/occ/instp/CLBeaconRegion/notifyEntryStateOnDisplay)设为`YES`，同时把`notifyOnEntry`设为`NO`即可

###Beacon的距离

当用户设备已经处于一个区域中时，app可以使用[startRangingBeaconsInRegion:](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/index.html#//apple_ref/occ/instm/CLLocationManager/startRangingBeaconsInRegion:)方法，Location manager会在beacon距离发生变化时调用[locationManager:didRangeBeacons:inRegion:](https://developer.apple.com/library/prerelease/ios/documentation/CoreLocation/Reference/CLLocationManagerDelegate_Protocol/index.html#//apple_ref/occ/intfm/CLLocationManagerDelegate/locationManager:didRangeBeacons:inRegion:)来通知app，这个代理方法会返回一个当前在区域中的beacon的`CLBeacon`对象数组，这个数组按照距离排序，最近的beacon在最前。

##Android
###Android权限

	<uses-permission android:name="android.permission.BLUETOOTH"/>

	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

BLUETOOTH：请求/接受连接，传输数据  
BLUETOOTH_ADMIN：发起设备发现，进行蓝牙设置

	<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>

添加这行把APP声明为只支持BLE手机  
如果需要同时支持非BLE手机，可以把required设为false，这样就可以在运行时根据手机是否支持BLE来决定程序逻辑  

	if (!getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)) {
    	Toast.makeText(this, R.string.ble_not_supported, Toast.LENGTH_SHORT).show();
    	finish();
	}

###配置BLE
如果手机支持BLE，但未开启，做下面两步：  
####1. 获取BluetoothAdapter

	// Initializes Bluetooth adapter.
	final BluetoothManager bluetoothManager =
        (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);

	mBluetoothAdapter = bluetoothManager.getAdapter();

BluetoothManager从4.3（level 18）引人

####2. 打开蓝牙

	private BluetoothAdapter mBluetoothAdapter;
	...
	// Ensures Bluetooth is available on the device and it is enabled. If not,
	// displays a dialog requesting user permission to enable Bluetooth.
	if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
    	Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    	startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
	}

###寻找BLE设备

	/**
	 * Activity for scanning and displaying available BLE devices.
	 */
	public class DeviceScanActivity extends ListActivity {

    	private BluetoothAdapter mBluetoothAdapter;
    	private boolean mScanning;
    	private Handler mHandler;

    	// Stops scanning after 10 seconds.
    	private static final long SCAN_PERIOD = 10000;
    	...
    	private void scanLeDevice(final boolean enable) {
        	if (enable) {
            	// Stops scanning after a pre-defined scan period.
            	mHandler.postDelayed(new Runnable() {
                	@Override
                	public void run() {
                    	mScanning = false;
                    	mBluetoothAdapter.stopLeScan(mLeScanCallback);
                	}
            	}, SCAN_PERIOD);

            	mScanning = true;
            	mBluetoothAdapter.startLeScan(mLeScanCallback);
        	} else {
            	mScanning = false;
            	mBluetoothAdapter.stopLeScan(mLeScanCallback);
        	}
        	...
    	}
	...

	}

###关键方法 startLeScan()
如果只要扫描特定外设，可以用

	public boolean startLeScan (UUID[] serviceUuids,BluetoothAdapter.LeScanCallback callback)
传入一个UUID数组

###扫描结果回调example
```
private LeDeviceListAdapter mLeDeviceListAdapter;
...
// Device scan callback.
private BluetoothAdapter.LeScanCallback mLeScanCallback =
        new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(final BluetoothDevice device, int rssi,
            byte[] scanRecord) {
        runOnUiThread(new Runnable() {
           @Override
           public void run() {
               mLeDeviceListAdapter.addDevice(device);
               mLeDeviceListAdapter.notifyDataSetChanged();
           }
       });
   }
};
```
注意上面两个扫描方法从4.3引入，5.0废除，改用

	public void startScan (List<ScanFilter> filters, ScanSettings settings,ScanCallback callback)
新的方法不再传入UUID数组，而是ScanFilter  
ScanFilter支持：  

* 用来标识GATT service的UUID；
* BLE设备的名称；
* BLE设备的Mac地址；
* Service数据；
* 制造商数据。

注意：同时只能扫描传统蓝牙和BLE中的一种，不能同时扫描

###Android Beacon Library
[这个Library](http://altbeacon.github.io/android-beacon-library/)可以让Android设备像iOS设备一样使用beacon，比如扫描到一个或多个beacon时app可以收到通知，以大约每秒一次的频率获得距离的更新等。要注意的是这个库默认兼容的是开放标准的[AltBeacon](http://altbeacon.org/)，如果需要扫描iBeacon或者其他beacon需要对其[BeaconParser类](http://altbeacon.github.io/android-beacon-library/javadoc/org/altbeacon/beacon/BeaconParser.html)进行layout适配，参见[这个例子](http://stackoverflow.com/questions/25027983/is-this-the-correct-layout-to-detect-ibeacons-with-altbeacons-android-beacon-li)

##参考链接
[http://developer.android.com/guide/topics/connectivity/bluetooth-le.html](http://developer.android.com/guide/topics/connectivity/bluetooth-le.html)  
[https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257-CH1-SW1](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257-CH1-SW1)  
[BLE Beacons: iBeacon, AltBeacon, URIBeacon and derivatives](http://austinblackstoneengineering.com/ble-beacons-ibeacon-altbeacon-uribeacon-and-derivatives/)  
[Android Beacon Library](http://altbeacon.github.io/android-beacon-library/)