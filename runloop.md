## runloop


### runloop基本概念
`runloop`可以看成和线程是一对一的关系，但是`runloop`可以进行嵌套。`runloop`主要用来处理各种事件，能够节省`CPU`资源，在需要处理的时候唤醒，空闲的时候休眠。

### 猜想内部实现

	  function loop() {
	      initialize();
	      do {
	          var message = get_next_message();
	          process_message(message);
	      } while (message != quit);
	  }
	  
### 唤醒和休眠

线程休眠前，指定用于唤醒我的`mach_port`,然后去休眠后，系统内核会将线程挂起，处于`mach_msg_trap()`状态，当其他线程（比如有一个进程在后面控制用户输入，一直在跑）向内核发送`mach_msg`的时候，内核去`mach_port`唤醒休眠的线程，休眠线程的`trap`状态被唤醒，`runloop`继续干活
	  
### 实际运用

* `AFNetworking`：担心线程提前推出，导致`NSOperation` 无法接受回调，于是作者单独起一个`thread`,内置一个`runloop`，回调都由它接收，不占用主线程，也不耗`CPU`资源。类似于常驻服务的线程。`runloop`一直监听`port`，使`runloop`一直等待，怕他没事干，退出
* `TableView`中实现平滑滚动延迟加载图片:利用`CFRunLoopMode`的特性，可以将图片的加载放到`NSDefaultRunLoopMode`的`mode`里，这样在滚动`UITrackingRunLoopMode`这个`mode`时不会被加载而影响到。
* 监控卡顿的方法
	* [iOS 实时卡顿监控](https://github.com/suifengqjn/PerformanceMonitor) 
	* [简单监测iOS卡顿的demo](http://www.jianshu.com/p/71cfbcb15842) 
	* [检测iOS的APP性能的一些方法](http://www.starming.com/index.php?v=index&view=91)
	* [微信iOS卡顿监控系统](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207890859&idx=1&sn=e98dd604cdb854e7a5808d2072c29162&scene=4#wechat_redirect)
	* [iOS实时卡顿监控](http://www.tanhao.me/code/151113.html/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
* `runloop`处理大量大图片加载问题
	* [iOS Fast Scrolling with RunLoop Work Distribution
](https://github.com/diwu/RunLoopWorkDistribution)
* 在遇到崩溃的时候，自主处理例如弹出提示等
	* [让Crash的App回光返照](http://www.jianshu.com/p/549c37f60bf7)
	* [iOS 启动连续闪退保护方案](http://wereadteam.github.io/2016/05/23/GYBootingProtection/)
	* [漫谈 iOS Crash 收集框架](http://mp.weixin.qq.com/s?__biz=MjM5NTIyNTUyMQ==&mid=208483273&idx=1&sn=37ee88e06e7426f59f3074c536370317&scene=21) 

###  拓展阅读

* [RunLoop学习笔记，从CF层面了解由于CFRunLoopMode机制iOS程序ScrollView的滑动为何如此平滑的原因。还有介绍AFNetworking如何单独发起一个global thread内置runloop达到不占用主线程又不耗CPU资源的。](http://www.starming.com/index.php?v=index&view=74)
* [深入理解RunLoop](http://blog.ibireme.com/2015/05/18/runloop/)

