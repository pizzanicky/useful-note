#Android Log污染
Android设备的Log污染已经相当严重，不仅app，还包括各种手机厂商的第三方系统。新买回来的手机，用`adb logcat`，瞬间刷爆你眼球。原来用`adb logcat *:I`，如今用`adb logcat *:E`都阻止不了呼啦啦滚屏。  
###影响性能？
从而想到这么多log是否会对性能有影响。  
量多了一定会，特别是不少log都涉及String的拼接等操作，另外总会浪费I/O资源吧。  
不过[官方文档建议发布前关闭log和debug](http://developer.android.com/tools/publishing/preparing.html)应该更多是从安全角度考虑：  

* 从源码中移除所有对[Log](http://developer.android.com/reference/android/util/Log.html)类方法的调用
* manifest文件中，去除`<application>`tag的`android:debuggable`属性，或将其设为`false`
* 去除所有[Debug](http://developer.android.com/reference/android/os/Debug.html)类中的tracing调用，如[startMethodTracing()](http://developer.android.com/reference/android/os/Debug.html#startMethodTracing)和[stopMethodTracing()](http://developer.android.com/reference/android/os/Debug.html#stopMethodTracing)
>重要：如果使用了[WebView](http://developer.android.com/reference/android/webkit/WebView.html)来显示支付内容，或者用了JavaScript接口，一定要确保用`WebView.setWebContentsDebuggingEnabled()`关闭debug。因为debug允许用户使用Chrome DevTools注入脚本，提取内容

