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