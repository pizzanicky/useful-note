#Android的GPU过度绘制
##打开GPU过度绘制调试
开发人员工具——调试GPU过度绘制——选择“显示过度绘制区域”

这时候屏幕上会出现各种色块，这些色块就可以帮咱诊断app的过度绘制

总共5种色块：

* 原始颜色：无过度绘制
* 蓝色：多绘制一次
* 绿色：多绘制两次
* 粉色：多绘制三次
* 红色：多绘制四次以上

有些多次绘制不可避免，调试的目标是让多数区域为原始颜色，个别为蓝色

##如何处理过度绘制

Theme本身会对背景颜色进行绘制，如果再像下面这样加上背景颜色，就相当于多绘制

	<RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="#FFFFFF">

其他导致过度绘制的原因还有复杂的layout层级结构等等，我们一般要尽量把绘制次数控制在绿色（多绘2次）以内

官方工具：

[hierarchy-viewer](http://developer.android.com/tools/performance/hierarchy-viewer/index.html)

[Tracer for OpenGL ES](http://developer.android.com/tools/help/gltracer.html)

##参考链接
[http://developer.android.com/tools/performance/debug-gpu-overdraw/index.html](http://developer.android.com/tools/performance/debug-gpu-overdraw/index.html)

[Optimizing Layouts in Android – Reducing Overdraw](http://riggaroo.co.za/optimizing-layouts-in-android-reducing-overdraw/)