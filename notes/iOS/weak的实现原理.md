在`Runtime`中专门维护了一个用于存储 `weak`指针变量的weak表，这实际上是一个`Hash`表。这个表的 key 是 weak指针 所指向的内存地址，value 是指向这个内存地址的所有weak指针地址的数组。

![image](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/weak原理/存储weak的hash表keyvalue.png)



过程可以总结为3步

1. 初始化时：`runtime` 会调用`objc_initWeak` 函数，初始化一个新的 weak指针 指向对象的地址。   
2. 添加引用时：`objc_initWeak`函数会调用 `objc_storeWeak()` 函数， `objc_storeWeak()`的作用是更新指针指向，创建对应的弱引用表。       
3. 释放时，调用 `clearDeallocating` 函数。`clearDeallocating` 函数首先根据对象地址获取保存 weak指针 地址的数组，然后遍历这个数组把其中的数据设为 nil，最后把这个 `entry` 从weak表 中删除，最后清理对象的记录。    


https://www.jianshu.com/p/13c4fb1cedea