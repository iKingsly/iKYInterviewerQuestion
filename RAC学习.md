#RAC学习
##1. RAC入门资料
[最快让你上手ReactiveCocoa之基础篇](http://www.jianshu.com/p/87ef6720a096)

[细说ReactiveCocoa的冷信号与热信号（一）](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-1.html)

[细说ReactiveCocoa的冷信号与热信号（二）：为什么要区分冷热信号](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-2.html)

[细说ReactiveCocoa的冷信号与热信号（三）：怎么处理冷信号与热信号](http://tech.meituan.com/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-3.html)

[ReactiveCocoa学习笔记](http://yulingtianxia.com/blog/2014/07/29/reactivecocoa/)

[ReactiveCocoa Tutorial – the Definitive Introduction: Part 1/2](http://southpeak.github.io/blog/2014/08/02/reactivecocoazhi-nan-%5B?%5D-:xin-hao/)

[ReactiveCocoa Tutorial – the Definitive Introduction: Part 2/2](http://southpeak.github.io/blog/2014/08/02/reactivecocoazhi-nan-er-:twittersou-suo-shi-li/)
##2. RAC进阶资料
[最快让你上手ReactiveCocoa之进阶篇](http://www.jianshu.com/p/e10e5ca413b7)

[ReactiveCocoa 和 MVVM 入门](http://yulingtianxia.com/blog/2015/05/21/ReactiveCocoa-and-MVVM-an-Introduction/)

[MVVM Tutorial with ReactiveCocoa: Part 1/2](http://southpeak.github.io/blog/2014/08/08/mvvmzhi-nan-yi-:flickrsou-suo-shi-li/)

[MVVM Tutorial with ReactiveCocoa: Part 2/2](http://southpeak.github.io/blog/2014/08/12/mvvmzhi-nan-er-:flickrsou-suo-shen-ru/)

## 随便写写

![](http://upload-images.jianshu.io/upload_images/852671-8be058eb23856b3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 响应式编程

响应式编程`(Functional Reactive Programming, FRP)`是一种和事件流有关的编程模式，关注导致状态值改变的行为事件，一系列事件组成了事件流。一系列事件是导致属性值发生变化的原因。FRP非常类似于设计模式里的观察者模式。

> 在计算机中，响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。

> 例如，在命令式编程环境中， a:=b+c表示将表达式的结果赋给  a，而之后改变  b或 c的值不会影响  a。但在响应式编程中，  a的值会随着 b或  c的更新而更新。

例如：

	int a, b, c;
	b = 1;
	c = 2;
	a = b + c;

对于传统命令式编程，可以解读为`a`是`b+c`表达式的值

而在响应式的编程中应解读成：建立了一个动态的数据流关系（当`c`或者`b`的值发生变化时，`a`的值自动发生变化）

感受到的函数响应式编程的优点：

* 直观：直观的代码容易编写、阅读和维护
* 灵活：灵活的特性便于应对变态的需求


## RAC


在编写`iOS`代码时，我们的大部分代码都是在响应一些事件：按钮点击、接收网络消息、属性变化等等。但是这些事件在代码中的表现形式却不一样：如`target-action`、代理方法、`KVO`、回调或其它。`ReactiveCocoa`的目的就是定义一个统一的事件处理接口，这样它们可以非常简单地进行链接、过滤和组合。

#### 常用宏

`RAC(TARGET, [KEYPATH, [NIL_VALUE]]):`用于给某个对象的某个属性绑定
	
    // 只要文本框文字改变，就会修改label的文字
    RAC(self.labelView,text) = _textField.rac_textSignal;
 
 `RACObserve(self, name):`监听某个对象的某个属性,返回的是信号。
 
	[RACObserve(self, username) subscribeNext:^(NSString *newName) {
		   NSLog(@"%@", newName);
	}];
	
宏`@weakify`与`@strongify`在`Extended Objective-C`库中引用，它们包含在`ReactiveCocoa`框架中。`@weakify`允许我们创建一些影子变量，它是都是弱引用(可以同时创建多个)，`@strongify`允许创建变量的强引用，这些变量是先前传递给`@weakify`的。**主要是用来避免循环引用**

	
	@weakify(self)

	[[self.searchText.rac_textSignal map:^id(NSString *text) {
	    return [self isValidSearchText:text] ? [UIColor whiteColor] : [UIColor yellowColor];
	}] subscribeNext:^(UIColor *color) {
	    @strongify(self)
	    self.searchText.backgroundColor = color;
	}];
### 常用操作方法

[ReactiveCocoa常见操作方法介绍](http://yulingtianxia.com/blog/2014/07/29/reactivecocoa/)


### MVVM
