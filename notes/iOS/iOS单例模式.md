## 什么是单例模式？

**单例模式**：保证一个类仅有一个实例，并提供一个唯一的全局访问点。

苹果大量使用了此模式。例如：`[NSUserDefaults standardUserDefaults]`, `[UIApplication sharedApplication]`, `[UIScreen mainScreen]`, `[NSFileManager defaultManager]`，所有的这些方法都返回一个单例对象。


## 单例模式的优缺点

**优点**：
1. 将一些初始化比较耗费资源的类写成单例，避免重复的初始化耗费资源。或者一些创建和销毁比较频繁的类，用单例模式可以提高性能。
2. 单例的唯一性和全局性可以很方便的用来传值。


**缺点**：
1. 单例创建之后，对象保存在静态区，在程序结束之后才会释放，过多的单例会增加常驻内存消耗。
2. 单例的职责过重，在一定程度上违背了“单一职责原则”。
3. 单例没有抽象层，因此单例类的扩展有很大困难。
4. 由于程序的任一模块都可以访问单例，如果某个与单例进行交互的代码出现了问题，很容易影响其他使用到单例的地方。

## 单例的写法
创建一个单例类，在.h文件声明单例的初始化方法。
```
+ (instancetype)shareSingleton;
```
在.m文件里实现，使用`static`修饰，保证实例全局存在。使用GCD的`dispatch_once`方法来保证初始化代码只执行一次。

```
@implementation LDSingleton

static LDSingleton *instance;
+ (instancetype)shareSingleton {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc]init];
    });
    return instance;
}

@end
```
为了避免在外部使用其他初始化方法创建单例对象，导致单例类有多个实例对象。可以重写`allocWithZone:` 、`copyWithZone:` 和`mutableCopyWithZone:`方法，并返回唯一的单例对象。

### 禁用其他获取实例方法
也可以禁用外部调用其他初始化方法，来保证单例的唯一性。在.h文件声明：
```
- (instancetype)init NS_UNAVAILABLE;
+ (instancetype)new NS_UNAVAILABLE;
- (id)copy NS_UNAVAILABLE;
- (id)mutableCopy NS_UNAVAILABLE;
```
这些方法不用实现。由于使用`NS_UNAVAILABLE`宏，在外部调用`init`等方法时编译器会报错。

## 一个便捷的可继承单例方法

这有个[
demo](https://github.com/alexiiio/LDOptionalSingleton)。

导入文件：   
![LDOptionalSingleton](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/LDOptionalSingleton.png)

只要引入头文件，遵守协议就可以成为一个单例类。
```
#import "NSObject+LDOptionalSingleton.h"

@interface Person : NSObject<LDOptionalSingleton>

@end
```
获取单例对象：
```
[Person LD_sharedSingleton];
```
原理是使用runtime关联对象方法，加上分类和协议。单例可以继承，并且父类和子类拥有各自的单例对象。
