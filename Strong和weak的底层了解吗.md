#Strong 和weak的底层了解吗？
##1. 首先我们看一下LLVM官方文档
 >LLVM Clang项目里面的对 ARC 的文档:
	1. **For __strong objects**, the new pointee is first retained; second, the lvalue is loaded with primitive semantics; third, the new pointee is stored into the lvalue with primitive semantics; and finally, the old pointee is released. This is not performed atomically; external synchronization must be used to make this safe in the face of concurrent loads and stores.
	2. **For __weak objects**, the lvalue is updated to point to the new pointee, unless the new pointee is an object currently undergoing deallocation, in which case the lvalue is updated to a null pointer. This must execute atomically with respect to other assignments to the object, to reads from the object, and to the final release of the new pointee.


1. strong: 这里先创建一个指针，将新的值retain一次，将指针动态指向新的值，并将旧的值release一次
2. weak: 动态地将指针指向新的值，如果这个值刚被dealloc，就会将指针更新为一个null指针

##2. strong 和 weak的使用注意
1. 默认情况下对于对象都是使用strong
2. 当一个对象没有被strong指针指向时，这个对象会被释放掉，指向它的weak指针会指向nil
3. **__unsafe_unretained**与weak的区别是，weak在对象没有强指针指向的时候会被设置为nil，而__unsafe_unretained不会设置为nil
4. weak可以用来有效防止循环引用的问题

##参考资料
[Objective-C Automatic Reference Counting (ARC)](http://clang.llvm.org/docs/AutomaticReferenceCounting.html)
[Transitioning to ARC Release Notes](https://developer.apple.com/library/ios/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)