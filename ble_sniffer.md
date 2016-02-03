#BLE sniffer
对BLE不够了解，想看看市面上蓝牙设备的数据包到底怎么发  
马云网买了[EN Dongle](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.R2YyiY&id=524612675637&_u=u3dbp0l3b67)，虽然不是Nordic nRF的官方解决方案，但目前没发现兼容性问题


安装CP210x USB to UART Bridge的[驱动](https://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx#windows)，以便windows能够识别Dongle

安装[Wireshark](https://www.wireshark.org/download.html)
>注意1：Wireshark必须是1.12.1以上，2.0.x以下的版本，因为plugin是针对1.12.x编写的。32/64位都可以  
>注意2：安装Wireshark是确保安装libpcap库，因为python库抓下来的信息都是pcap格式

下载Nordic的绿色嗅探器[nRF sniffer(ble-sniffer_win_1.0.1.zip)](https://www.nordicsemi.com/eng/nordic/download_resource/31920/14/15495034)  

把Dongle插入USB，识别无误后运行sniffer，会出现一个终端窗口，窗口呈现端口信息，版本号，命令帮助。然后下面就是可用设备列表（Available devices）

##主要命令：  
**l**：列出可供嗅探的设备  
**方向键**：在设备列表中选择（嗅探器同时只能嗅探一个设备，所以。。。）  
**[#]或回车**：选择设备进行嗅探  
**e**：类似回车，不过只嗅探广告帧  
**w**：启动Wireshark，观察嗅探结果  
**x/q**：退出  

##使用流程：  
按**l**刷新列表，方向键选择感兴趣的设备，回车，按**w**启动Wireshark（初次启动会自动安装plugin），然后就可以在Wireshark查看抓包信息了。Wireshark可以把抓到的包分字段显示为可读的树形信息，在Bluetooth Low Energy Link Layer中可以看到。  

##备注：
* 这款嗅探器基本在3米之外就扫不到设备了，这个距离太短了。不知道商用探针产品是否也如此
* 小米手环在非连接情况下发布比较频繁，连接后发包间隔明显变长，几十秒到一分钟不等

###参考链接：
[https://learn.adafruit.com/introducing-the-adafruit-bluefruit-le-sniffer/nordic-nrfsniffer](https://learn.adafruit.com/introducing-the-adafruit-bluefruit-le-sniffer/nordic-nrfsniffer)


