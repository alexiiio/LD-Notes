
**NSInteger**的定义如下：
```
#if __LP64__ || 0 || NS_BUILD_32_LIKE_64
typedef long NSInteger;
typedef unsigned long NSUInteger;
#else
typedef int NSInteger;
typedef unsigned int NSUInteger;
#endif
```
**NSInteger是对c整数类型的一个封装，如果是64位系统，NSInteger就是long类型，在32位系统就是int类型。使用NSInteger系统会自动选择较长的整数类型。**

**int**：表示整数。目前在一般电脑中，int占4个字节，一个字节8位，共32位，所以取值范围是：`-2^31~2^31-1`。

**long**：长整型。long的长度大于等于int类型。


在iPhone XR模拟器下，打印NSInteger和int的字节长度：
```
/*
 NSInteger 和 int 字节长度区别
 */
NSLog(@"NSInteger字节长度：%lu",sizeof(NSInteger));
NSLog(@"int字节长度：%lu",sizeof(int));
```
输出结果：
```
NSInteger字节长度：8
int字节长度：4
```
注意：虽然`NSInteger`名字前面有`NS`前缀，但`NSInteger`并不是对象类型，和`int`一样是基本数据类型，`NSNumber`才是对象类型。