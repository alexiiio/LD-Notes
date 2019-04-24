**nil：表示空对象**，指向oc中对象的空指针。一般我们把对象置为空写作 `obj = nil`。

**NiL：表示空类**，指向oc中类的空指针。nil 和 NiL 基本没有区别，可以通用，但一般严谨的做法是用nil来描述类的实例，NIL描述类。

**NULL：是c语言中的空指针**。

nil、NiL、NULL实际都是可以通用的，都表示**空对象**：不存在或已经释放了的对象。

**NSNull：表示值为空的对象**，已经分配了内存地址的对象类型，对象的值为空。NSNull是一个继承自NSObject的单例类，用于表示不允许空值的集合对象中的空值。该类只有一个类方法`+ (NSNull *)null`。在集合对象中不允许插入空对象，但是可以插入值为空的对象。


代码验证：
```
    /*
      nil、NiL、NULL、NSNull
     */
    if (nil == Nil) {
        NSLog(@"%@ == %@",nil,Nil);
    }else {
        NSLog(@"%@ != %@",nil,Nil);
    }
    if (Nil == NULL) {
        NSLog(@"%@ == %@",nil,NULL);
    }else {
        NSLog(@"%@ != %@",nil,NULL);
    }
    if (nil == [NSNull null]) {
        NSLog(@"%@ == %@",nil,[NSNull null]);
    } else {
        NSLog(@"%@ != %@",nil,[NSNull null]);
    }
    // NSNull单例对象地址
    NSLog(@"[NSNull null]: %p",[NSNull null]);
```
打印结果：
```
 (null) == (null)
 (null) == (null)
 (null) != <null>
 NSNull: 0x10eed9ea8
```