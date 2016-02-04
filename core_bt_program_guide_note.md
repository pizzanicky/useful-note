#Core Bluetooth编程指南概要
本文是官方文档[Core Bluetooth Programming Guide](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257-CH1-SW1)的阅读笔记  

Core Bluetooth framework是BLE协议栈的抽象，提供了用来和BLE设备通讯的一些类，BLE无线技术基于Bluetooth 4.0规范，规范中定义了一套低功耗设备间的通讯协议。  
![图](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBTechnologyFramework_2x.png)

文档说从OS X v10.9和iOS 6起Mac和iOS设备也可以以BLE外设的方式工作，但**目前我还不知道怎样做**  

##术语和概念
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
###中心，外设，外设数据在Core Bluetooth中如何表示
####中心侧对象
App多数属于这种情况，本地中心设备用`CBCentralManager`对象来表示，可以用此对象管理已发现或已连接的外设（用`CBPeripheral`表示），包括扫描，发现，连接操作。外设的Service和Characteristic分别用`CBService`和`CBCharacteristic`这两种对象来表示。
####外设侧对象
OS X v10.9及iOS6以上的设备可以已BLE外设的方式工作，这种情况下，则用`CBPeripheralManager`对象来表示外设，可以管理外设Service和Characteristic的发布，广播，以及响应中心设备的读写请求，对侧的中心设备用`CBCentral`表示。
Service和Characteristic分别用`CBMutableService`和`CBMutableCharacteristic`来表示
##执行中心侧任务
###启动CBCentralManager
创建CBCentralManager：  

	myCentralManager =
        [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];

这里`self`被设为接收中心事件的代理，分发queue设为`nil`，CBCentralManager会使用main queue来分发事件。  
CBCentralManager创建时会调用其代理对象的[centralManagerDidUpdateState:](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBCentralManagerDelegate_Protocol/index.html#//apple_ref/occ/intfm/CBCentralManagerDelegate/centralManagerDidUpdateState:)方法，我们必须实现这个方法来确保当前设备支持BLE且可用
###发现广播中的外设
方法如下：

	[myCentralManager scanForPeripheralsWithServices:nil options:nil];

>**注意：**如果第一个参数设`nil`，CBCentralManager会返回所有发现的外设，无论它们都支持哪些服务。通常在正式app中，多是指定一个[CBUUID](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBUUID_Class/index.html#//apple_ref/occ/cl/CBUUID)数组，其中包含app真正感兴趣的那些服务的UUID

上述方法调用后，代理对象每次发现一个外设，CBCentralManager都会调用[centralManager:didDiscoverPeripheral:advertisementData:RSSI:](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBCentralManagerDelegate_Protocol/index.html#//apple_ref/occ/intfm/CBCentralManagerDelegate/centralManager:didDiscoverPeripheral:advertisementData:RSSI:)。被发现的外设以[CBPeripheral](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBPeripheral_Class/index.html#//apple_ref/occ/cl/CBPeripheral)对象的方式返回，如下：

	- (void)centralManager:(CBCentralManager *)central
 	didDiscoverPeripheral:(CBPeripheral *)peripheral
    advertisementData:(NSDictionary *)advertisementData
                  RSSI:(NSNumber *)RSSI {
 
    	NSLog(@"Discovered %@", peripheral.name);
    ...
    
找到感兴趣的外设后，为了省电，一般要停止扫描：

	[myCentralManager stopScan];
	
>暂时用不到连接和获取Service，Characteristic，后面没看，以后看了再补

##关于Service，Characteristic和UUID
###关系
外设的Service和Characteristic用128位的蓝牙UUID来标识，在Core Bluetooth框架中用`CBUUID`对象来代表。为了使用方便，蓝牙SIG组织定义发布了一些常用UUID的16位缩短版本，比如，SIG把心率服务的16位UUID定义为`180D`，对应蓝牙4.0规范中定义的128位的0000180D-0000-1000-8000-00805F9B34FB。

`CBUUID`类提供了一些便于app处理长UUID的简便方法，例如，访问心率服务时不用在代码中传入128位的UUID，而是调用`UUIDWithString`方法并传入该服务预定义的16位UUID：

	CBUUID *heartRateServiceUUID = [CBUUID UUIDWithString: @"180D"];
	
创建`CBUUID`对象时，Core Bluetooth会自动帮我们补足128位UUID
###创建自定义Service和Characteristic的UUID
如果我们要用的Service和Characteristic不属于预定义的蓝牙UUID，就得用命令行工具`uuidgen`自己生成一个128位的UUID：

	$ uuidgen
	71DA3FD1-7E10-41C1-B16F-4430B506CDE7
	
##Core Bluetooth的后台处理
默认情况下，Core Bluetooth的许多任务在app后台运行或挂起时是不能工作的，但通过把app声明为支持Core Bluetooth后台执行模式，可以允许将app从挂起状态唤醒，以处理一些蓝牙事件。如果app不需要完全运行于后台模式，也可以仅仅在重要事件发生时让系统通知app。

不过，就算app支持了Core Bluetooth的全部后台执行模式，app也不能保证一直运行下去，在必要的时候，系统可能还是会为了前台app释放内存而干掉你的app。从iOS7开始，Core Bluetooth支持了保存中心或外设对象的状态信息，以及在app启动时恢复状态，如果要保证**蓝牙连接等长时间操作**，这个是不错的功能。
###Core Bluetooth后台执行模式
如果app要在后台执行蓝牙任务，必须在`Info.plist`文件中进行声明，系统会将app从挂起状态唤醒，来处理蓝牙事件。可声明的后台执行模式有两种：分别是作为中心设备和作为外设。做声明时添加`UIBackgroundModes`这个key即可

##Best Practice
###注意Radio的使用和电量消耗
电量消耗不用说，BLE通讯要用到设备的radio模块，而其他的无线通讯可能都要用到设备的radio，比如WiFi，经典蓝牙，而且其他app也可能会使用BLE，调用[scanForPeripheralsWithServices:options:](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBCentralManager_Class/index.html#//apple_ref/occ/instm/CBCentralManager/scanForPeripheralsWithServices:options:)时，只要你不明确要求它停止，中心设备会一直使用radio来监听广告包，所以尽量最小化自己app对radio的使用
####只在必要的情况下设置CBCentralManagerScanOptionAllowDuplicatesKey
外设每秒钟可能会发出多个广告包来把自己的存在告知中心设备，用[scanForPeripheralsWithServices:options:](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBCentralManager_Class/index.html#//apple_ref/occ/instm/CBCentralManager/scanForPeripheralsWithServices:options:)方法扫描设备时，默认会把扫到的多个广告包合并为**单个发现事件**，也就是说，CBCentralManager每发现一个新外设，或者已发现外设的广告数据有变化才会调用一次[centralManager:didDiscoverPeripheral:advertisementData:RSSI](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBCentralManagerDelegate_Protocol/index.html#//apple_ref/occ/intfm/CBCentralManagerDelegate/centralManager:didDiscoverPeripheral:advertisementData:RSSI:)方法，和它收到多少个广告包**没有直接关系**。

如果需要持续性地获得变化的RSSI，可以在调用扫描方法时把[CBCentralManagerScanOptionAllowDuplicatesKey](https://developer.apple.com/library/ios/documentation/CoreBluetooth/Reference/CBCentralManager_Class/index.html#//apple_ref/c/data/CBCentralManagerScanOptionAllowDuplicatesKey)设为`YES`，这样会去除重复广告包的过滤，每次从外设收到广告包都会产生一个发现事件。当然，这样做对耗电量和性能会有一定影响。

>用蓝牙做定位时可能需要设置此KEY

