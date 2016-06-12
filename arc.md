## ARC

自动引用计数


### `ARC`有效时，`autorealse`

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

### 更多知识 明天更新


