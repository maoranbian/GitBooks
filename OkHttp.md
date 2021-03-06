参考资料：

1.介绍网络OSI和TCP/IP模型以及Socket基本含义：[网络编程 -- 从 Socket 编程 到 OkHttp 框架](https://segmentfault.com/a/1190000011220027)

2.介绍Cookie的：[Android 关于Https中Cookie的使用（PersistentCookieJar）](http://blog.csdn.net/pengguichu/article/details/73339329)

其中需要了解1是因为OKHttp底层采用的是Socket连接。2是因为OKHttp3默认不保存Cookie，要自己在OkHttpClient.Builder中使用cookieJar自己设置，有个开源的PersistentCookieJar可以设置，让OKHttp下发Cookie给客户端，客户端保存后下次请求带着这个cookie。

cookie的基本认识：我们应该对后台的数据处理有一定的认识。由于HTTP协议无状态的特性，后台是无法保存用户的信息的，在此情形下，Cookie就诞生了。Cookie的作用是在客户端保存数据，然后在每一次对该站点进行访问的时候都会携带此Cookie中的数据，于是后台就可以通过客户端Cookie中的数据来识别用户。早期很多网站甚至将用户名和密码保存在Cookie中。在Web应用开发中有一句真理：**任何的客户端行为都是不可信赖的**。Cookie作为客户端技术，也有着同样的困境。Cookie会被攻击、被篡改，黑客可以从Cookie中查看到用户的用户名和密码，甚至是信用卡的密码。在此情形下，Session的概念被提出。 Session是一种服务端技术。服务端将数据保存在Session中，仅仅将此Session的ID发送给客户端，客户端在请求该站点的时候，只需要将Cookie中的SESSIONID这个数据发送给服务端即可。这样一来就避免了用户信息泄露的尴尬。参考于[OkHttp3 \(四\)——Cookie与拦截器](https://www.jianshu.com/p/3360f4b6b3fe)

边看源码，边看解析，有一个系列介绍基于3.7，跟原来app中我们用的3.2版本不太一样，请求部分全都改为了拦截器链的形式：

* [OkHttp源码分析——整体架构](https://yq.aliyun.com/articles/78105?spm=5176.8091938.0.0.hlEONd)
* [OkHttp源码分析——拦截器](https://yq.aliyun.com/articles/78104?spm=5176.8091938.0.0.hlEONd)
* [OkHttp源码分析——任务队列](https://yq.aliyun.com/articles/78103?spm=5176.8091938.0.0.hlEONd)
* [OkHttp源码分析——缓存策略](https://yq.aliyun.com/articles/78102?spm=5176.8091938.0.0.hlEONd)
* [OkHttp源码分析——多路复用](https://yq.aliyun.com/articles/78101?spm=5176.8091938.0.0.hlEONd)

 

