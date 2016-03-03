#三角定位

一般说的三角定位其实应该是广义的，涵盖Triangulation(三角测量)，Trilateration(三边测量)，和Multilateration(多边测量)这几种测算方法

##三边测量
这种方法相对最简单常用，已知三个点的位置，以及到这三个点的距离来定位。GPS主要基于这种定位方法。以已知的三个点为圆心，到各自距离为半径画圆，要确定的位置就在三个圆的重叠区域里

[这里](https://github.com/jtubert/Group5_iBeacons/blob/show-work/ios/Group5iBeacons/Debug/Managers/Location/Trilateration/Trilateration.m)是个iBeacon三边测量定位的简单例子

##三角测量
三边测量是测距离，三角测量是测角度

图片来自维基，A,B为已知点，测得到未知点的两个角度就得到坐标
![link](https://upload.wikimedia.org/wikipedia/commons/8/8e/Triangulation-boat.png?1456994555585)

##iBeacon定位
根据一些有经验者的建议，三边测量和三角测量都不是很适合用来做iBeacon定位，主要因为受环境和手持设备握持的影响，会产生比较大的干扰，造成误差，建议是直接简单粗暴——按照扫描到的最强beacon的位置来确定位置，因为iBeacon本身覆盖范围就不大

##参考链接
[https://zh.wikipedia.org/wiki/三角测量](https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E6%B8%AC%E9%87%8F)  
[http://stackoverflow.com/questions/20332856/triangulate-example-for-ibeacons](http://stackoverflow.com/questions/20332856/triangulate-example-for-ibeacons)