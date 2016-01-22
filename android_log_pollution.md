#Android Log污染
Android设备的Log污染已经相当严重，不仅app，还包括各种手机厂商的第三方系统。新买回来的手机，用`adb logcat`，瞬间刷爆你眼球。原来用`adb logcat *:I`，如今用`adb logcat *:E`都阻止不了呼啦啦滚屏。  
###影响性能？
从而想到这么多log是否会对性能有影响。  
量多了一定会，特别是不少log都涉及String的拼接等操作，另外总会浪费I/O资源吧。  

###官方意见
不过[官方文档建议发布前关闭log和debug](http://developer.android.com/tools/publishing/preparing.html)应该更多是从**老版本**的安全角度考虑：  

* 从源码中移除所有对[Log](http://developer.android.com/reference/android/util/Log.html)类方法的调用
* manifest文件中，去除`<application>`tag的`android:debuggable`属性，或将其设为`false`
* 去除所有[Debug](http://developer.android.com/reference/android/os/Debug.html)类中的tracing调用，如[startMethodTracing()](http://developer.android.com/reference/android/os/Debug.html#startMethodTracing)和[stopMethodTracing()](http://developer.android.com/reference/android/os/Debug.html#stopMethodTracing)
>重要：如果使用了[WebView](http://developer.android.com/reference/android/webkit/WebView.html)来显示支付内容，或者用了JavaScript接口，一定要确保用`WebView.setWebContentsDebuggingEnabled()`关闭debug。因为debug允许用户使用Chrome DevTools注入脚本，提取内容

另外，AOSP的代码贡献指导文档里也说要[有节制地使用Log](http://source.android.com/source/code-style.html#log-sparingly)，第一句就提到：  
>While logging is necessary, it has a significantly negative impact on performance and quickly loses its usefulness if not kept reasonably terse.

###常用的几种Log优化方法

#####发布前手动搜出所有打log的行，注释或删除
手工动代码，不是个好方法，容易出现问题，比如：  

		if(boring)
			Log.d(TAG, "blabla");
		callMyGirlFriend();
		goToSleep();
如果单独把第二行注释了。。你有可能就要狗带了  
>这也是为什么`if`后面哪怕只有一行语句也最好加上`{}`  
当然，也有人习惯用`;//`来做单行注释。。。

#####做一个自己的Log类
这个不难理解，封装一下Android的Log类，用一个标志去控制，像这样：  
	
	public class Log {
    	static final boolean LOG = false;

    	public static void i(String tag, String string) {
        	if (LOG) android.util.Log.i(tag, string);
    	}
    	public static void e(String tag, String string) {
        	if (LOG) android.util.Log.e(tag, string);
    	}
    	public static void d(String tag, String string) {
        	if (LOG) android.util.Log.d(tag, string);
    	}
    	public static void v(String tag, String string) {
        	if (LOG) android.util.Log.v(tag, string);
    	}
    	public static void w(String tag, String string) {
        	if (LOG) android.util.Log.w(tag, string);
    	}
	}
	
但是，问题来了。比如，这样打log：  

	Log.d(TAG, getSomeString() + "..." + getAnotherString() + "some suffix");
这时就算LOG标志为`false`, 此log方法还是会执行，log虽然不会打印出来，第二个参数中的消耗性能的**字符串拼接**动作还是照样执行，结果就只是控制台输出好看了一些。

#####用ProGuard优化
在ProGuard的config文件里添加：

	-assumenosideeffects class android.util.Log {
    	public static *** d(...);
    	public static *** v(...);
	}
这样，在用ProGuard优化时就会移除这两个调用  
但是，问题又来了，这个移除虽然不会影响代码逻辑，但是会导致back trace时的代码行数和源码对不上。。对于本身就要混淆代码的同学来说，这个可能不是什么问题
