#load 与 initialize 的区别
##1. 调用顺序
以main为分界，load方法在main函数之前执行，initialize在main函数之后执行
##2.相同点和不同点
###2.1 相同点
1. **load**和**initialize**会被自动调用，不能手动调用它们。
2. 子类实现了**load**和**initialize**的话，会隐式调用父类的**load**和**initialize**方法
3. load和initialize方法内部使用了锁，因此它们是线程安全的。
###2.2 不同点
1. 子类中没有实现**load**方法的话，不会调用父类的**load**方法；而子类如果没有实现**initialize**方法的话，也会自动调用父类的**initialize**方法。
2. **load**方法是在类被装在进来的时候就会调用，**initialize**在第一次给某个类发送消息时调用（比如实例化一个对象），并且只会调用一次，是懒加载模式，如果这个类一直没有使用，就不回调用到**initialize**方法。

##3. load
在执行load方法之前，会调用**load_images**方法，用来扫描镜像中的**+ load符号**，将需要调用 load 方法的类添加到一个列表中**loadable_classes**，在这个列表中，会先把父类加入到待加载列表，**这样保证父类在父类在子类钱调用load方法**，而分类中的load方法会在类的load的方法后面加入另外一个待加载列表**loadable_categories**，这样保证了两个规则：
1. 父类先于子类调用
2. 类先于分类调用

在扫描完load方法加入到待加载方法后，会调用**call_load_methods**，先从**loadable_classes**调用**类的load方法**，call_class_loads；调用完**loadable_classes**后会调用**loadable_categories**中分类的load方法，**call_category_loads**。

调用顺序如下：
1. 父类load先于类添加到**loadable_classes**列表，通过**call_class_loads**，调用列表中的load方法，这样父类的load先于类的load执行
2. 当**loadable_classes**为空的时候，查看**loadable_classes**是否为空，如果不为空则调用**call_category_loads**加载分类中的load方法，这样分类的load在类之后执行
##4. initialize
**initialize** 只会在对应类的方法第一次被调用时，才会调用，**initialize** 方法是在 alloc 方法之前调用的，alloc 的调用导致了前者的执行。

initialize的调用栈中，直接调用其方法的其实是_class_initialize 这个C语言函数，在这个方法中，主要是向为初始化的类发送**+initialize**消息，不过会强制父类先发送。

与 load 不同，initialize 方法调用时，所有的类都已经加载到了内存中。

##5. 使用场景
###5.1 load
load一般是用来交换方法Method Swizzle，由于它是线程安全的，而且一定会调用且只会调用一次，通常在使用UrlRouter的时候注册类的时候也在load方法中注册
###5.2 initialize
initialize方法主要用来对一些不方便在编译期初始化的对象进行赋值，或者说对一些静态常量进行初始化操作
##参考资料
[你真的了解 load 方法么？](https://segmentfault.com/a/1190000005025068)

[细说OC中的load和initialize方法](http://www.jianshu.com/p/d25f691f0b07)

[懒惰的 initialize 方法](http://draveness.me/initialize/)