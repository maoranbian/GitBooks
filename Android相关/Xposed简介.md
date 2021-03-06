#### 什么是Xposed

Xposed是一款可以在不修改apk的情况下修改系统功能和应用程序的框架，需要root权限。基于它可以创建很多功能模块并在功能不冲突的情况下同时运作。所有的修改在内存中进行，停用了该框架系统便恢复原样。

原理简介：Xposed通过替换/system/bin/app\_process程序控制zygote进程，使得它在系统启动过程中会加载Xposed Framework中的一个jar文件即XposedBridge.jar，从而对zygote进程机器创建的Dalvik虚拟机劫持，允许开发者独立替换任何class，如UI或者系统变量等。（记得有版本兼容问题）

能做什么：功能强大，可以实现app读取手机信息的控制（修改手机IMEI，网络类型、GPS信息、手机号、手机型号等）、app权限控制、app行为控制和修改（自动抢红包、游戏作弊、付费音乐下载、解锁歌曲权限限制等）、内核系统修改（去除签名验证、系统参数修改等）等几乎你想到一切功能。

[Xposed框架原理深入研究](http://blog.csdn.net/zhangmiaoping23/article/details/52572447)  ：流程原理比较清晰

[Xposed的框架的使用](http://blog.csdn.net/u012417380/article/details/55254369?locationNum=13&fps=1)  使用方式的一些解释

大概关键内容，就下面这个图里的总结。版本不同会有出入，很多没展开，等以后有机会看Xposed相关的热修复时可以补上。

![](/assets/Xposed框架.png)

参考：[Android热补丁技术—dexposed原理简析\(阿里Hao\)](http://blog.csdn.net/yueqian_scut/article/details/50939034)

Dexposed修复方案借助了Xposed的hook原理，不过它只hook自己的应用，不用root。它的关键点是：在native层中先找到要修复的Java函数对应的Method对象，修改它变为native方法，把它的nativeFunc指向hookedMethodCallback。这样对这个java函数的调用就转为调用hookedMethodCallback这个native函数了，然后再用这个native函数回调java层自己实现的统一接口来处理。这个统一接口是XC\_MethodReplacement类，它主要有beforeHookedMethod、afterHookedMethod和replaceHookedMethod等几个方法，前两个在执行原java函数前后做一些事，replaceHookedMethod则是替换原java方法。

一句话概括这种hook方法，就是通过把原java方法的类型改为native来把对java函数的调用转到native层，在native层用dvm的各种函数来操作Method的指针和对象来控制函数流程。

![](/assets/Dexposed流程.png)

基于Xposed的需要区分Dalvik虚拟机和art虚拟机，因此有版本的差别，Android5.0之后默认是art的，会有所差别。差别的原因是什么呢？可以查阅两种虚拟机的差异。

