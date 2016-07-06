## iOS 做过哪些优化


 
###  iOS 安装包优化

* 资源优化：对资源文件下手，压缩图片/音频，去除不必要的资源
* 编译优化：
	* `release`版应该选择`Fastest`, `Smalllest`，这个选项会开启那些不增加代码大小的全部优化（新版`Xcode`默认）
	* 去除符号信息(都是新版`Xcode`默认) 
* 可执行文件优化：
 	* 查看第三方库编译之后，.o文件大小，如果对大小影响很大可以考虑替换
 	* 删除无用代码，如果项目历时长，代码遗留多，这样还是比较有效的（可以通过脚本查找没有引用的类和没有被调用的方法）
 	
 	
###  	内存优化

* 延迟加载 `Views` 、一些资源
* 做好缓存工作，如果服务器没有更新资源，客户端没必要再多请求一次，直接加在本地资源就好－`YYCache`
* `Autorelease Pool `降低内存峰值
* 图片缓存


###  	性能优化

* `view` 设置为不透明，避免图层混合，消耗GPU资源
* 圆角、阴影、光栅化，重写`DrawRect`这些导致离屏渲染的情况，都应该尽量避免
* 层次过多的视图，可以考虑使用 `Frame `布局， `autolayout` 这时候性能明显低于 `Frame `布局
* `I/O`不要在主线程


* `TableView` 优化

	* 设置合适的`reuseIdentifier`，重用 `cell`
	* 对于高度固定直接设置`rowHight`，保证不必要的高度计算和调用，对于可变的`Cell`，预估算行高
	* 以前看到过一个第三方框架：使用`runloop`，页面处于空闲状态时执行计算，正在滑动列表时显然不应该执行计算任务影响滑动体验，当 `UI` 没在滑动时，默认的 `Mode` 是 `NSDefaultRunLoopMode`，当用户正在滑动 `UIScrollView` 时，`RunLoop` 将切换到 `UITrackingRunLoopMode` 接受滑动手势和处理滑动事件（包括减速和弹簧效果），此时，其他 `Mode `（除 `NSRunLoopCommonModes` 这个组合 `Mode`）下的事件将全部暂停执行，来保证滑动事件的优先处理
	* `cell` 圆角，避免离屏渲染


## 资料

[检测iOS的APP性能的一些方法](http://www.starming.com/index.php?v=index&view=91)

[微信读书 iOS 性能优化总结](http://wereadteam.github.io/2016/05/03/WeRead-Performance/)

[iOS可执行文件瘦身方法](http://blog.cnbang.net/tech/2544/)