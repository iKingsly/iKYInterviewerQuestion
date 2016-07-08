#1. 并行和并发
简单来说，若说两个任务A和B并发执行，则表示任务A和任务B在同一时间段里被执行（更多的可能是二者交替执行）；若说任务A和B并行执行，则表示任务A和任务B在同时被执行（这要求计算机有多个运算器）；
一句话：并行要求并发，但并发并不能保证并行。

#2. Dispatch Queues
`Dispatch Queue`是一个任务执行队列，可以让你异步或同步地执行多个Block或函数。Dispatch Queue是**FIFO**的，即先入队的任务总会先执行。目前有三种类型的Dispath Queue：

- 串行队列（Serial dispatch queue）
- 并发队列（Concurrent dispatch queue）
- 主队列（Main dispatch queue）

##2.1 Serial Dispatch Queue 串行队列
serial dispatch queue中的block按照先进先出（FIFO）的顺序去执行，实际上为单线程执行。即每次从queue中取出一个task进行处理；用户可以根据需要创建任意多的serial dispatch queue，serial dispatch queue彼此之间是并发的；

```
dispatch_queue_t queue;
queue = dispatch_queue_create("com.example.MySerialQueue", DISPATCH_QUEUE_SERIAL);
```

##2.2 Concurrent Dispatch Queue 并行队列
Concurrent Dispatch Queue
相对于Serial Dispatch Queue，Concurrent Dispatch Queue一次性并发执行一个或者多个task；和Serial Dispatch Queue不同，系统提供了四个global concurrent queue，使用dispatch_get_global_queue函数就可以获取这些global concurrent queue；
和Serial Dispatch Queue一样，用户也可以根据需要自己定义concurrent queue；创建concurrent dispatch queue也使用dispatch_queue_create方法，所不同的是需要指定其第二个参数为DISPATCH_QUEUE_CONCURRENT即可：

```
dispatch_queue_t queue;
queue = dispatch_queue_create("com.example.MyConcurrentQueue", DISPATCH_QUEUE_CONCURRENT);

```

##2.3 Main Dispatch Queue 主队列
> The main dispatch queue is a globally available serial queue that executes tasks on the application’s main thread.

根据我的理解，application的主要任务（譬如UI管理之类的）都在main dispatch queue中完成；根据文档的描述，main dispatch queue中的task都在一个thread中运行，即application’s main thread（thread 1）。

P.S: 所以，如果想要更新UI，则必须在main dispatch queue中处理，获取main dispatch queue也很容易，调用dispatch_get_main_queue()函数即可。

###注意 
thread和dispatch queue之间没有从属关系

#3. dispatch_sync和dispatch_async 同步和异步
一个结论：
>`dispatch_sync` 派发的block的执行线程和 `dispatch_sync` 上下文线程是同一个线程;
>`dispatch_async` 派发的block的执行线程和 `dispatch_async` 上下文线程不是同一个线程，**主队列** 下异步任务还是在主队列下执行；
>对于serial dispatch queue中的tasks，无论是同步派发还是异步派发，其执行顺序都遵循FIFO;

#4. 使用GCD来替代performSelector的原因:
##4.1 `performSelector` 的局限性:
###4.1.1 performSelector 会导致内存泄漏问题
用performSelector:调用了一个方法，编译器并不知道将要调用的selector是什么，因此，也就不了解其方法签名及返回值，甚至连是否有返回值都不清楚。而且，由于编译器不知道方法名，所以就没办法用ARC的内存管理规则来判定返回值是不是该释放。鉴于此，ARC采用了比较谨慎的做法，就是不添加释放操作。然而，这么做可能导致内存泄漏，因为方法在返回对象时已经将其保留了。
###4.1.2 performSelector 返回值只能是void或对象类型（id类型）
如果想返回整数或浮点数等scalar类型值，那么就需要执行一些复杂的转换操作，而这种转换操作很容易出错。由于id类型表示指向任意Objective—C对象的指针，所以从技术上来讲，只要返回的大小和指针所占大小相同就行，也就是说，在32位架构的计算机上，可以返回任意32位大小的类型；而在64位架构的计算机上，则可以返回任意64位大小的类型。除此之外，还可以返回NSNumber进行转换…若返回的类型为C语言结构体，则不可使用performSelector方法。
###4.1.3 performSelector 提供的方法局限性大

```
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
```
具备延后功能的那些方法无法处理带有两个参数的选择子。而能够指定执行线程的哪些方法，则与之类似，所以也不是特别通用。如果要用这些方法，就得把很多参数打包到字典中，然后在被调用的方法中将这些参数提取出来，这样会增加开销，同时也提高了产生bug的可能性。


#5.GCD 中block的使用问题
##5.1 GCD 会对添加的Block进行复制
Dispatchqueue对添加的Block会进行复制,在完成执行后自动释放。换 句话说,你不需要在添加 Block 到 Queue 时显式地复制
##5.2 GCD 中的autorelease pool
GCD dispatch queue 有自己的autorelease pool来管理内存对象，但是不保证在什么时候会进行回收，如果在block中创建了大量的对象，可以添加自己的autorelease pool来进行管理。
##5.3 GCD 中在开新的线程执行任务一定比较快吗
如果对于工作量小的block切换线程的开销，比直接在原来线程上执行block的开销要大，那么这样的话，会导致开新的线程反而没有原来执行的快
##5.4 GCD 中的block会造成循环引用吗
会，如果控制器持有block的话，是会造成循环引用，如果我们只持有queue是不会造成循环引用。

#6. GCD 的暂停和继续
1. **dispatch_suspend** 会暂停一个队列以阻止执行block对象，调用 **dispatch_suspend** 会增加queue的引用计数
2. **dispatch_resume** 会使得队列恢复继续执行block对象，调用 **dispatch_resume** 会减少queue的引用计数

挂起和继续是异步的，只在没有执行的block上生效，挂起一个block不会导致已经开始执行的block停止执行。


#7. GCD中的信号量 Semaphore
##7.1 信号量概念
>停车场剩余4个车位，那么即使同时来了四辆车也能停的下。如果此时来了五辆车，那么就有一辆需要等待。

>　　信号量的值就相当于剩余车位的数目，dispatch_semaphore_wait函数就相当于来了一辆车，

>　　dispatch_semaphore_signal，就相当于走了一辆车。停车位的剩余数目在初始化的时候就已经指明了（dispatch_semaphore_create（long value））

>　　调用一次dispatch_semaphore_signal，剩余的车位就增加一个；调用一次dispatch_semaphore_wait剩余车位就减少一个；

>　　当剩余车位为0时，再来车（即调用dispatch_semaphore_wait）就只能等待。有可能同时有几辆车等待一个停车位。有些车主

>　　没有耐心，给自己设定了一段等待时间，这段时间内等不到停车位就走了，如果等到了就开进去停车。而有些车主就像把车停在这，

>　所以就一直等下去。

##7.2 信号量的创建和使用
1. 创建 `dispatch_semaphore_create`

  ```  /*!
 * @function dispatch_semaphore_create
   使用信号量来处理多个线程之间竞争资源的情况特别合适，在value等于0的时候进行等待，在value大于0的时候运行
 *
 * @param value
 * 初始化创建的信号量的个数
 *
 * @result
 * 当前创建的信号量
 */
__OSX_AVAILABLE_STARTING(__MAC_10_6,__IPHONE_4_0)
DISPATCH_EXPORT DISPATCH_MALLOC DISPATCH_RETURNS_RETAINED DISPATCH_WARN_RESULT
DISPATCH_NOTHROW
dispatch_semaphore_t
dispatch_semaphore_create(long value);```

2. 等待 `dispatch_semaphore_wait`
  ```/*!
 * @function dispatch_semaphore_wait
 *
 * @abstract
 * 等待一个信号量
 *
 * @discussion
 * 会对信号量进行-1,如果value小于0，接下来的方法会允许等待的时间里一直等待直到有其他线程有信号量产生即value>1 才开始执行
 *
 * @param dsema
 * The semaphore. 不允许设置为NULL
 * @param timeout
 * 允许等待的超时时间
 * 一下两个是宏定义的时间
 * DISPATCH_TIME_NOW and DISPATCH_TIME_FOREVER constants.
 *
 */
__OSX_AVAILABLE_STARTING(__MAC_10_6,__IPHONE_4_0)
DISPATCH_EXPORT DISPATCH_NONNULL_ALL DISPATCH_NOTHROW
long
dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout);
```

3. 产生 `dispatch_semaphore_signal`
  ```
  /*!
 * @function dispatch_semaphore_signal
 *
 * @abstract
 * 对信号量增加1
 *
 * @discussion
 * 对信号量增加1,如果之前的value==0,那么这个操作会唤醒一个正在等待的线程
 * 
 * @param dsema The counting semaphore.
 * The semaphore. 不允许设置为NULL
 */
__OSX_AVAILABLE_STARTING(__MAC_10_6,__IPHONE_4_0)
DISPATCH_EXPORT DISPATCH_NONNULL_ALL DISPATCH_NOTHROW
long
dispatch_semaphore_signal(dispatch_semaphore_t dsema);
  ```
##注意
`dispatch_semaphore_signal` 和 `dispatch_semaphore_wait` 必须成对出现，并且在 `dispatch_release(aSemaphore)`;之前，`aSemaphore` 的value需要恢复之前的数值，不然会导致 `EXC_BAD_INSTRUCTION`
在ARC情况下不需要使用dispatch_release来进行释放，有系统统一管理

##7.3 `Dispatch Semaphore` 的应用
###7.3.1 控制并发线程数量
```
/*
 *
 实战版本：具有专门控制并发等待的线程，优点是不会阻塞主线程，可以跑一下 demo，你会发现主屏幕上的按钮是可点击的。但相应的，viewdidload 方法中的栅栏方法dispatch_barrier_async就失去了自己的作用：无法达到“等为数组遍历添加元素后，检查下数组的成员个数是否正确”的效果。

 *
 */
void dispatch_async_limit(dispatch_queue_t queue,NSUInteger limitSemaphoreCount, dispatch_block_t block) {
//控制并发数的信号量
    static dispatch_semaphore_t limitSemaphore;
    //专门控制并发等待的线程
    static dispatch_queue_t receiverQueue;
    
    //使用 dispatch_once而非 lazy 模式，防止可能的多线程抢占问题
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        limitSemaphore = dispatch_semaphore_create(limitSemaphoreCount);
        receiverQueue = dispatch_queue_create("receiver", DISPATCH_QUEUE_SERIAL);
    });
    
    dispatch_async(receiverQueue, ^{
        //可用信号量后才能继续，否则等待
        dispatch_semaphore_wait(limitSemaphore, DISPATCH_TIME_FOREVER);
        dispatch_async(queue, ^{
            !block ? : block();
            //在该工作线程执行完成后释放信号量
            dispatch_semaphore_signal(limitSemaphore);
        });
    });
}
```


###7.3.2 为 NSURLSession 添加同步方法 在数据请求完成后才会返回

```
+ (NSData *)sendSynchronousRequest:(NSURLRequest *)request returningResponse:(NSURLResponse *__autoreleasing *)response error:(NSError *__autoreleasing *)error {
    __block NSData *data = nil;
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);

    [[[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData *taskData, NSURLResponse *taskResponse, NSError *taskError) {
        data = taskData;

        if (response)
            *response = taskResponse;

        if (error)
            *error = taskError;

        dispatch_semaphore_signal(semaphore);
    }] resume];

    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

    return data;
}
```

但是也要思考下为什么 Apple 取消了同步方法：同步方法的风险远远超过受益。

要注意：

- 除非万不得已，否则永远不要尝试在主线程上发送同步的网络请求
- 尽量只在后台线程中独占线程发送同步的网络请求
风险如下所示：

###7.3.3 加锁 
阻塞线程，知道value>1 
```
    lock = dispatch_semaphore_create(1);
    
    dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
    
    // 先从缓存中尝试获取
    _YYModelMeta *meta = CFDictionaryGetValue(cache, (__bridge const void *)(cls));
    dispatch_semaphore_signal(lock);
```

#8. CGD Group的实践
##8.1 用group wait来等待queue的一组任务
如果要等待queue中的一系列操作完成后再去执行一个相应的任务，除了用barrier之外,我们也可以通过group来进行处理，dispatch group wait会阻塞当前的线程，直到group中的任务完成才会停止阻塞，这样我们可以达到一个目的，直到前面的任务完成了，才执行后面的代码

```
    dispatch_queue_t queue = dispatch_queue_create("abc", DISPATCH_QUEUE_CONCURRENT);

    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, queue,^{
        NSLog(@"1");
    });
    
    dispatch_group_async(group, queue, ^{
        NSLog(@"2");
    });
    
    dispatch_group_async(group, queue, ^{
        
        NSLog(@"3");
    });
    
    dispatch_group_async(group, queue, ^{
        sleep(5);
        NSLog(@"4");
    });
    
    // 开启一个异步队列来等待
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
        dispatch_async(queue, ^{
            NSLog(@"done");
        });
    });
    
    NSLog(@"主线程");
```
打印结果：
```
2016-07-08 15:12:45.188 dispatch_source[45653:1777587] 3
2016-07-08 15:12:45.188 dispatch_source[45653:1777585] 1
2016-07-08 15:12:45.188 dispatch_source[45653:1777586] 2
2016-07-08 15:12:45.188 dispatch_source[45653:1777495] 主线程
2016-07-08 15:12:50.189 dispatch_source[45653:1777617] 4
2016-07-08 15:12:50.189 dispatch_source[45653:1777618] done
```
##8.2 用group notify来实现等待queue的一组任务
用 `dispatch_group_notify`方法可以等待group中的任务，notify中的任务在原来group中的任务执行结束前不会执行
```
    dispatch_queue_t queue = dispatch_queue_create("abc", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, queue,^{
        NSLog(@"1");
    });
    
    dispatch_group_async(group, queue, ^{
        NSLog(@"2");
    });
    
    dispatch_group_async(group, queue, ^{
        
        NSLog(@"3");
    });
    
    dispatch_group_async(group, queue, ^{
        sleep(5);
        NSLog(@"4");
    });
    
    // 等待group中的任务执行完成
    dispatch_group_notify(group, queue, ^{
        NSLog(@"done");
    });

    NSLog(@"主线程");
```
打印结果
```
2016-07-08 15:25:04.207 dispatch_source[45904:1789795] 3
2016-07-08 15:25:04.207 dispatch_source[45904:1789794] 1
2016-07-08 15:25:04.207 dispatch_source[45904:1789793] 2
2016-07-08 15:25:04.208 dispatch_source[45904:1789768] 主线程
2016-07-08 15:25:09.211 dispatch_source[45904:1789798] 4
2016-07-08 15:25:09.212 dispatch_source[45904:1789798] done
```

##8.3 手动进入group
`dispatch_group_enter` 手动通知 Dispatch Group 任务已经开始。你必须保证 `dispatch_group_enter` 和 `dispatch_group_leave` 成对出现，否则你可能会遇到诡异的崩溃问题。
```
  // 进入group
  dispatch_group_enter(group);
  ** 加入组执行的代码 **
  
  // 离开group
  dispatch_group_leave(group)
```
##8.4 应用例子
1. 多线程下载9张小图片，然后在都下载好后将9张小图片合成一个大图片
2. 监测多个网络请求的完成

# 9 Dispatch Source 分派源
dispatch source 是基础数据类型,协调特定底层系统事件的处理。GCD 支持 以下 dispatch source:
- Timer dispatch source: 定期产生通知
- Signal dispatch source: UNIX 信号达到时产生通知
- Descriptor dispatch source: 各种文件和socket操作的通知
    - 数据可读
    - 数据可写
    - 文件在文件系统中被删除、移动、重命名
    - 文件元素信息gaibian
- Process dispatch source: 进程相关的事件通知
    - 进程退出时
    - 当进程发起fork或exec等调用
    - 信号被递送到进程
- Mach port dispatch source: Mach 相关事件的通知
- Custom dispatch source: 你自己定义并自己触发
    
`Dispatch source` 替代了异步回调函数,来处理系统相关的事件。当你配置一个` dispatch source` 时,你指定要监测的事件、`dispatch queue`、以及处理事件的代 码(block 或函数)。当事件发生时, `dispatch source` 会提交你的 block 或函数到 指定的 queue 去执行。
和手工提交到 queue 的任务不同,dispatch source 为应用提供连续的事件源。 除非你显式地取消,dispatch source 会一直保留与 dispatch queue 的关联。只要 相应的事件发生,就会提交关联的代码到 dispatch queue 去执行。
为了防止事件积压到 dispatch queue,dispatch source 实现了事件合并机制。
**如果新事件在上一个事件处理器出列并执行之前到达,dispatch source 会将新旧事件的数据合并**。根据事件类型的不同,合并操作可能会替换旧事件,或者更新 旧事件的信息。
```
名称	                                 内容
DISPATCH_SOURCE_TYPE_DATA_ADD	      变量增加
DISPATCH_SOURCE_TYPE_DATA_OR	       变量OR
DISPATCH_SOURCE_TYPE_MACH_SEND	   MACH端口发送
DISPATCH_SOURCE_TYPE_MACH_RECV	   MACH端口接收
DISPATCH_SOURCE_TYPE_PROC	     监测到与进程相关的事件
DISPATCH_SOURCE_TYPE_READ	        可读取文件映像
DISPATCH_SOURCE_TYPE_SIGNAL	         接收信号
DISPATCH_SOURCE_TYPE_TIMER	          定时器
DISPATCH_SOURCE_TYPE_VNODE	       文件系统有变更
DISPATCH_SOURCE_TYPE_WRITE           可写入文件映像
```
##9.1 创建Dispatch Source
>分派源提供了高效的方式来处理事件。首先注册事件处理程序，事件发生时会收到通知。如果在系统还没有来得及通知你之前事件就发生了多次，那么这些事件会被合并为一个事件。

####**创建步骤**
1. 使用dispatch_source_create 函数创建 dispatch source
2. 配置dispatch source
  - 为 dispatch source 设置一个Handle
  - 对于定时器源,使用 dispatch_source_set_timer 函数设置定时器信
息
3. 为dispatchsource赋予一个取消处理器
4. 调用 dispatch_resume 函数开始处理事件

```
    //1. 使用dispatch_source_create 函数创建 dispatch source
    // 指定DISPATCH_SOURCE_TYPE_DATA_ADD，做成Dispatch Source(分派源)。设定Main Dispatch Queue 为追加处理的Dispatch Queue
    _processingQueueSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0,
                                                    dispatch_get_main_queue());
    __block NSUInteger totalComplete = 0;
    
    // 2 配置dispatch source
    dispatch_source_set_event_handler(_processingQueueSource, ^{
        
        ***执行的代码***
    });
    // 4. 调用 dispatch_resume 函数开始处理事件
    // 分派源创建时默认处于暂停状态，在分派源分派处理程序之前必须先恢复。
    dispatch_resume(_processingQueueSource);
```
##9.2 暂停、恢复与取消操作
需要记得的一点是分派源创建时默认处于暂停状态，在分派源分派处理程序之前必须先恢复。
`dispatch_suspend(queue)`: 用来对分派源进行暂停操作，只能暂停还没执行的任务，如果已经开始执行的任务不会受到影响

`dispatch_resume (source)`: 对暂停的分派源进行恢复操作

暂停一个 dispatch source 期间,发生的任何事件都会被累积,直到 dispatch source 继续。但是不会递送所有事件,而是先合并到单一事件,然后再一次递送。dispatch source中的handler的block会暂停执行，而 `dispatch_source_merge_data`会继续发送消息，只是handler不会被挂起不去处理，一旦恢复，handler会马上收到 `dispatch_source_merge_data` 发送来的所有消息

`dispatch_source_cancel(source)`: 除非你显式地调用 `dispatch_source_cancel` 函数,dispatch source 将一直保持 活动,取消一个 dispatch source 会停止递送新事件,并且不能撤销。

##9.4 Dispatch Source 和 **Dispatch Queue** 两者在线程执行上的关系
两者线程上没有关系，独立运行。 **Dispatch Queue** 像一个生产任务的生产者，而 **Dispatch Source** 像处理任务的消费者。可以一边异步生产，也可一边异步消费。你可以在任意线程上调用 `dispatch_source_merge_data` 以触发 `dispatch_source_set_event_handler` 。而句柄的执行线程，取决于你创建句柄时所指定的线程。
自定义源也需要一个队列，用来处理所有的响应句柄（block）。那么岂不是有两个队列了？没错，一个队列用来执行自定义源，一个用来执行句柄

##9.5 Dispatch Source 能通过合并事件来确保高负载下正常工作
在同一时间，只有一个处理 block 的实例被分配，如果这个处理方法还没有执行完毕，另一个事件就发生了，事件会以指定方式（ADD或 OR）进行累积。DispatchSource能通过合并事件（block）的方式确保在高负载下正常工作。

##9.6 Dispatch Source 的实际应用
###9.6.1 Dispatch Source 实现GCD中队列任务的终止
>实际上 Dispatch Queue 没有“取消”这一概念。一旦将处理追加到 Dispatch Queue 中，就没有方法可将该处理去除，也没有方法可在执行中取消该处理。编程人员要么在处理中导入取消这一概念。
要么放弃取消，或者使用 NSOperationQueue 等其他方法。
Dispatch Source 与 Dispatch Queue 不同，是可以取消的。而且取消时必须执行的处理可指定为回调用的Block形式。

###9.6.2 使用Dispatch Queue 来取代NSTimer
众所周知，定时器有NSTimer，但是NSTimer有如下弊端：
1. 必须保证有一个活跃的runloop，子线程的runloop是默认关闭的。这时如果不手动激活runloop，performSelector和scheduledTimerWithTimeInterval的调用将是无效的
2. NSTimer的创建与撤销必须在同一个线程操作、performSelector的创建与撤销必须在同一个线程操作。
3. 内存管理有潜在泄露的风险会造成循环引用

所以我们可以使用 `Dispatch Source` 的 `DISPATCH_SOURCE_TYPE_TIMER` 来实现这个效果

```
-(void) startGCDTimer{
    NSTimeInterval period = 1.0; //设置时间间隔
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
     _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    dispatch_source_set_timer(_timer, dispatch_walltime(NULL, 0), period * NSEC_PER_SEC, 0); //每秒执行
    dispatch_source_set_event_handler(_timer, ^{
        //在这里执行事件
        NSLog(@"每秒执行test");
    });
    
    dispatch_resume(_timer);
}
	
-(void) pauseTimer{
    if(_timer){
        dispatch_suspend(_timer);
    }
}
	
-(void) resumeTimer{
    if(_timer){
        dispatch_resume(_timer);
    }
}
	
-(void) stopTimer{
    if(_timer){
        dispatch_source_cancel(_timer);
        _timer = nil;
    }
}
```
###9.6.3 监控文件系统对象
设置`DISPATCH_SOURCE_TYPE_VNODE` 类型的Dispatch Source，可以中这个 Source 中接收文件删除、写入、重命名等通知。
```
    int fd = open(filename, O_EVTONLY);
    if (fd == -1)
        return NULL;
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE,
                                                       , DISPATCH_VNODE_RENAME, queue);
    if (source)
  {
      // 保持文件名
      int length = strlen(filename);
      char* newString = (char*)malloc(length + 1);
      newString = strcpy(newString, filename);
      dispatch_set_context(source, newString);
      // 设置Source的handler 来对监测文件修改的处理
      dispatch_source_set_event_handler(source, ^{
      const char* oldFilename = (char*)dispatch_get_context(source);
      MyUpdateFileName(oldFilename, fd);
      });
    
    // 做释放source之前的处理 关闭文件
      dispatch_source_set_cancel_handler(source, ^{
        char* fileStr = (char*)dispatch_get_context(source); free(fileStr);
        close(fd);
      });
    // 开始执行start
    dispatch_resume(source);
  }
```

###9.6.4 监测进程的变化
进程 dispatch source 可以监控特定进程的行为,并适当地响应。父进程可以 使用 dispatch source 来监控自己创建的所有子进程,例如监控子进程的死亡;类 似地,子进程也可以使用 dispatch source 来监控父进程,例如在父进程退出时自 己也退出。
```
    dispatch_queue_t queue = dispatch_get_main_queue();
    
    static dispatch_source_t source = nil;
    
    __typeof(self) __weak weakSelf = self;
  
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
        source = dispatch_source_create(DISPATCH_SOURCE_TYPE_SIGNAL, SIGSTOP, 0, queue);
        
        if (source){
            dispatch_source_set_event_handler(source, ^{
                
                NSLog(@"Hi, I am: %@", weakSelf);
            });
            dispatch_resume(source); 
        }
    });
```

可以在上面看到实现的结果，当执行的时候打断点，进程发现变化彩绘进去打印 `Hi, I am:XXXXX`这句话

###9.6.5 实现对搜索的截流（还没看源代码，等看完分析）

#10 dispatch_barrier 栅栏
`dispatch_barrier` 最大的作用就是用来做**阻塞**，阻塞当前的线程，做到一个**承上启下**的作用，只有在它之前的任务全部执行完之后，它和它之后的任务才能进行。可以理解为成语**一夫当关，万夫莫开**，只有在它面前的任务“死掉了”（即执行完了）后面的任务才能继续进行下去。
![Dispatch-Barrier.png](resources/8E3DEBC549E48F6124D554C5D518DB0D.png)

##10.1 使用注意
使用dispatch_barrier 是用来阻断并行任务不能确定先后任务完成的问题，**它必须使用在自定义并行队列上**，否则没有意义，为什么说没意义呢，我们接下来分析为什么没意义：
1. 如果运用在串行队列上，没有意义，因为串行队列本来就是先进先出的规则，用了栅栏跟没用没有区别
2. 如果使用全局队列，也是没有意义，我们每次 **dispatch_get_global_queue** 获取到的队列都是不同的，我们任务前后执行不在同一个线程上，也就没有了截流之分。

##10.2 可写对象的写入安全问题
系统中提供的可变对象都是线程不安全的，也就是在一个线程进行写入数据的时候，不允许其他线程访问，无论是读或者是写都是不允许的。

##10.2.1 使用dispatch_barrier来实现写入安全
```
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    dispatch_queue_t queue = dispatch_queue_create("iKingsly", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_barrier_async(queue, ^{
        for (int i = 0; i < 1000; i++) {
            dict[@(i)] = @(i);
        }
    });
    
    dispatch_async(queue, ^{
        for (int i = 0; i < 1000; i++) {
            NSLog(@"%@", dict[@(i)]);
        }
    });
```
##10.2.2 使用信号量来处理读写线程安全问题
```
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    dispatch_queue_t queue = dispatch_queue_create("iKingsly", DISPATCH_QUEUE_CONCURRENT);
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    dispatch_async(queue, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        for (int i = 0; i < 1000; i++) {
            dict[@(i)] = @(i);
        }
        dispatch_semaphore_signal(semaphore);
    });
    
    dispatch_async(queue, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        for (int i = 0; i < 1000; i++) {
            NSLog(@"%@", dict[@(i)]);
        }
        dispatch_semaphore_signal(semaphore);
    });
```

#参考链接
[《GCD实践之二 -- 多用GCD，少用performSelector系列方法》](http://zhangbuhuai.com/using-gcd-part-2/)

[Parse源码浅析系列（一）---Parse的底层多线程处理思路：GCD高级用法](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/01_Parse的多线程处理思路/Parse的底层多线程处理思路.md)

