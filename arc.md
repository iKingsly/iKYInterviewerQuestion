## ARC

自动引用计数


### `ARC`有效时，`autorealease`

1、当使用`alloc/new/copy/mutableCopy`开始的方法进行初始化时，会生成并持有对象，那么对于其他情况，例如
	
	id obj = [NSMutableArray array];

这种情况会自动将返回值的对象注册到`autorealeasepool`

	@autorealsepool{
		id __autorealeasing obj = [NSMutableArray array];
	}

2、`__weak`修饰符只持有对象的弱引用，而在访问引用对象的过程中，该对象可能被废弃。那么如果把对象注册到`autorealeasepool`中，那么在`@autorealeasepool`块结束之前都能确保对象的存在。

	id __weak obj1 = obj0;
	NSLog(@"class=%@",[obj1 class]);

对应的源代码为

	id __weak obj1 = obj0;
	id __autorealeasing tmp = obj1;
	NSLog(@"class=%@",[tmp class]);
	
3、`id`的指针或对象的指针在没有显式指定时会被附加上`__autorealeasing`修饰符

	    + (nullable instancetype)stringWithContentsOfURL:(NSURL *)url
                                            encoding:(NSStringEncoding)enc
                                               error:(NSError **)error;

等价于

	   NSString *str = [NSString stringWithContentsOfURL:
                                             encoding:
                                                error:<#(NSError * _Nullable __autoreleasing * _Nullable)#>]



### autorealeasepool使用场景
* 降低内存峰值
	* 典型的读入大量的图像的同时，改变其尺寸，这样图像文件读入到`NSData`对象，从中生成`UIImage`对象，改变尺寸后生成新的`UIImage`对象。这种情况下，产生大量的`autorealease`对象，就可以使用额外的 `autorealeasepool`来降低内存的消耗
* 使用多线程的时候，可能在其他非主线程会使用到

### ARC：——Strong 底层实现

`——Strong` 修饰符的变量在实际的程序中到底怎么运行？

	id __Strong obj = [[NSObject alloc] init];

那么底层模拟代码可以这么理解：

	id obj = objc_msgSend(NSobject, @selector(alloc));
	objc_msgSend(obj, @selector(init));
	objc_release(obj);
	
编译器自动帮我们加入了`release`，来释放对象

那么使用除了`alloc/new/copy/mutablecopy`以外的方法会是什么情况？

	id __strong obj = [NSMutableArray array];
模拟底层代码为：

	id obj = objc_msgSend(NSMutableArray, @selector(array));
	objc_retainAutoreleasedReturnValue(obj);
	objc_release(obj);
	

`objc_retainAutoreleasedReturnValue(obj)`是做什么用的呢？，是主要用于最优化程序运行

`array`方法的底层模拟是怎样的呢？

	id obj = objc_msgSend(NSmutableArray,@selector(alloc));
	objc_msgSend(obj,@selector(init));
	return objc_autorealeaseReturnValue(obj);

`objc_autorealeaseReturnValue(obj)`与`objc_retainAutoreleasedReturnValue(obj)`是成对的，`objc_autorealeaseReturnValue(obj)`会把对象注册到`autorealeasepool`中，但是如果它检测到方法执行列表中出现`objc_retainAutoreleasedReturnValue(obj)`方法，那么就不会将返回的对象注册到`autorealeasepool`，而是直接传递到方法和函数的调用方。这样直接传递可以达到最优化。

### `__Weak`底层实现 

优点：

* 避免循环引用
* 修饰的对象被废弃时，将`nil`赋值给对象
* 使用`weak`修饰符，即使使用注册到`autorealeasepool`中的对象

		id __weak obj1 = obj;

模拟源码

	id obj1;
	obj1 = 0;
	objc_storeWeak(&obj1,obj);
	objc_destoryWeak(&obj1);  等同于  objc_storeWeak(&obj1,0);


`objc_storeWeak(&obj1,obj)`函数将第二个参数的赋值对象的地址作为键值，将第一个参数的附有`__weak`修饰符的变量的地址注册到`weak`表中，如果第二个参数为`0`，则把变量的地址从`weak`中删除。一个键值可以注册多个变量的地址

由此可见，如果大量的`weak`变量，则会消耗`CPU`资源，所以`weak` 只用来避免循环引用


然后`weak`与 `autorealeasepool`的关系

	id __weak obj1 = obj;
	NSLog(@"%@",obj1);
	
模拟源码

	id obj1;
	objc_initWeak(&obj1,obj);
	id temp = objc_loadWeakRetained(&obj1);
	objc_autorelease(temp);
	NSLog(@"%@",temp);
	objc_destoryWeak(&obj1);
	
所以在`autorealeasepool`块结束前可以放心使用`weak`修饰变量

### 引用计数

	id __strong obj = [[NSObject alloc] init]; // 计数为1
	id __autoreleasing p = obj   //引用计数为 2
	－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
	id __strong obj = [[NSObject alloc] init]; // 计数为1
	id __weak o = obj;    //  计数为2，因为weak 修饰也会注册到pool中，但是使用打印函数时可能结果为1，并不能完全信任打印函数取到的值。同时在多线程情况下，也可能出现打印计数不准确的情况。
	
	
