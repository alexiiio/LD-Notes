
# 概念

KVO全称 Key-Value-Observer 键值观察者，就是观察者模式。        
**[KVO](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177-BCICJDHA) 是一种允许对象监听其他对象特定属性变化的机制**。

常用于模型和控制器层之间的通信。控制器观察模型对象的属性，视图对象通过控制器观察模型对象的属性。另外，模型对象可以观察其他对象或自身，通常用于确定依赖属性值何时改变。（比如，人的全名取决于姓氏和名字，当其中一个变化时，全名也应随之改变）

使用KVO观察属性有一对一关系和一对多关系。每个被观察的属性值变化都会通知观察者。

# KVO的使用


使用KVO包含注册、接收和取消注册3个步骤。

1. 注册观察者：使用被观察者的addObserver:forKeyPath:options:context:方法来注册观察者。

2. 接收通知：观察者实现observeValueForKeyPath:ofObject:change:context:方法来接收相关通知。只要被观察属性发生变化，就会给观察者发送消息，随后做出相应处理。

3. 移除观察者：当不再接收通知时，在观察者释放前，调用被观察者的removeObserver:forKeyPath:方法来移除观察者。

KVO的主要好处是不必在每次属性更改时都实施自己的方案来发送通知。KVO具有框架级（`framework-level`）支持使得它易于使用，不需要在项目中添加多余代码。此外，KVO基础架构（`infrastructure`）已经功能齐全，这使得支持单个属性的多个观察者以及依赖值变得容易。

## 其他特性

**手动控制通知**：有些时候我们可能希望控制通知过程，比如减少因程序特定原因而不必要的触发通知。

**依赖属性**：一个属性的值取决于另一个对象中的一个或多个其他属性的值。如果一个属性的值发生更改，则还应标记派生属性的值以进行更改。

这部分不细说，可参考[苹果KVO文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOCompliance.html#//apple_ref/doc/uid/20002178-SW1)


# 实现原理

**KVO是使用`isa-swizzling`技术实现的，依赖于Runtime的动态机制。**
顾名思义，isa是指向对象的类的指针，它维护一个派发表。这个派发表本质上包含类实现的方法的指针，以及其他数据。

当观察者注册成为某个对象属性的观察者时，被观察对象的isa指针被修改，指向中间类，而不是真正的类。因此，isa指针的值不一定反映实例的实际类。所以苹果建议在开发中不应该依赖isa指针，而是通过class实例方法来获取对象类型。

如图所示，在添加观察者前打印person的isa指针是Person类，而添加观察者后，变为NSKVONotifying_Person类。

![image](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/KVOisaChange1.png)

而系统为了伪装成原类，重写了中间类的`class`方法，所以使用`[self.person class]`获取到的依然是原类Person。

KVO的实现主要有以下几步，以Person类为例：
1. 当Person类的对象第一次添加观察者时，系统会在运行时动态生成一个Person的派生类，命名规则是在原类名前加上`NSKVONotifying_`前缀，即`NSKVONotifying_Person`。
2. 在派生类里重写被观察属性的`setter`方法，在setter方法里实现通知机制。在修改值之前调用`willChangeValueForKey:`方法，修改值之后调用`didChangeValueForKey:`方法，最终都会发送消息到`observeValueForKeyPath:ofObject:change:context:`方法。
3. 在派生类中重写`class`方法，将自己伪装成原类。还会重写`dealloc`方法释放资源。
4. 将被观察对象的`isa`指针指向新创建的派生类。


# 自己实现KVO

实现KVO会用到很多runtime的方法，之前最好先对runtime做些了解。文章末尾有完整代码。
首先，创建一个Person类作为被观察者类。
```
@interface Person : NSObject
@property(nonatomic,copy)NSString *name;
@property(nonatomic,assign)NSInteger age;
@end
```

然后创建一个NSObject的类目，添加相关方法。（方法名参照系统KVO方法）

```
@interface NSObject (LDKVO)
- (void)LD_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
- (void)LD_removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
- (void)LD_observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(void *)newValue ;
```

主要是添加观察者、移除观察者和接受通知的方法，接下来在.m
里实现。

## 创建派生类

在添加观察者的方法`- (void)LD_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath`里，先创建一个继承自Person的新类。

动态获取被观察者的类名
```
NSString *oldClass = NSStringFromClass(object_getClass(self));
```
拼接类名，以 LDKVO_ 为前缀。
```
NSString *newClass = [NSString stringWithFormat:@"LDKVO_%@",oldClass];
```
生成类，然后注册。实际代码先判断是否已经注册。
```
//  第一个参数：生成类的父类，第二个参数：类名
Class kvoClass = objc_allocateClassPair(object_getClass(self), newClass.UTF8String, 0);
//  注册类
objc_registerClassPair(kvoClass);
```
修改被观察者的isa指针。然后self就变成了LDKVO_XXX类。
```
object_setClass(self, kvoClass);
```

## 重写setter方法

使用`class_addMethod(Class cls, SEL name, IMP imp, const char *types);`给派生类添加setter方法。这个方法有4个参数。

第一个是Class类型，要添加方法的类。就是前面的kvoClass。

第二个参数是SEL类型，要添加的方法名。这里就是setter方法名，类似 `setName:` 注意“:”一定要有。
```
// 根据属性名生成setter方法名
NSRange firstRange = NSMakeRange(0, 1);
NSString *firstLetter = [keyPath substringWithRange:firstRange];
NSString *upperKey = [keyPath stringByReplacingCharactersInRange:firstRange withString:firstLetter.uppercaseString];
NSString *setMethodName = [NSString stringWithFormat:@"set%@:",upperKey];
```
生成SEL对象：
```
SEL setMethod = NSSelectorFromString(setMethodName);
```
第三个参数是`IMP`函数指针类型。即`setter`方法的指针。所以需要先**创建setter方法**。一个OC方法是一个至少包含两个参数（self和_cmd）的C函数。类似：
```
void myMethodIMP(id self, SEL _cmd)
{
    // implementation ....
}
```
而我们的`setter`方法还应该包含一个给属性赋值的参数，这个参数类型可能是对象也可能是基本数据类型。如果使用id类型传参，当接收到一个基本数据类型的参数时程序会崩溃，所以这里我想到了用`void *`，来接受任意类型参数。
```
void setterIMP(id self,SEL _cmd,void *newValue){

}
```
在`setter`方法里，我们需要完成`setter`方法本来要做的事情，即给属性赋值。而KVC赋值会调用自身`setter`方法，造成死循环。所以这里要**调用父类的`setter`**。这里写方法有个技巧，就是看方法需要传什么类型参数，我们就创建什么类型参数放进去，`objc_msgSendSuper`方法需要传一个`struct objc_super *`类型的参数，即指向`objc_super`结构体的指针，那我们就创建一个` objc_super`结构体，然后`&`取地址。`struct objc_super`的创建同理。苹果文档一般有清楚的参数说明，按照要求写就行了。

```
    //    调用父类的set方法
    struct objc_super sClass = {
        .receiver = self,
        .super_class = class_getSuperclass(object_getClass(self))
    };
    // 需要传一个 struct objc_super * 类型的结构体
    objc_msgSendSuper(&sClass, _cmd, newValue);
```

然后要**给观察者发通知**。告诉观察者属性值已经发生变化了。因为一个属性可能有多个观察者，需要遍历所有该属性的观察者发送消息。实际实现会稍微复杂些。
```
    if ([observer respondsToSelector:@selector(LD_observeValueForKeyPath:ofObject:change:)]) {
        objc_msgSend(observer, @selector(LD_observeValueForKeyPath:ofObject:change:),key,self,newValue);
    }
```
然后就是第四个参数`const char *types`，要添加的方法的**类型编码**。类型编码是编译器为了帮助运行时系统，将每个方法的返回值和参数编码成一个C字符串。可以查看[苹果文档Type Encodings](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100)来了解。

这里获取父类的`setter`方法的类型编码就行了，因为参数和返回值类型肯定是一样的。
```
  // 获取父类的setter Method
    Method superSetter = class_getInstanceMethod([self superclass], setMethod);
//  获取父类setter方法的 类型编码
    const char *setterTypes = method_getTypeEncoding(superSetter);
```
最后填充过参数的方法如下：
```
class_addMethod(kvoClass, setMethod, (IMP)setterIMP, setterTypes);
```
这样重写`setter`方法就完成了。

## 关联观察者对象

我们需要把观察者与被观察者关联起来，这样`setter`执行的时候才能找到观察者，给它发通知。

因为KVO有一对一和一对多，即一个观察者可以监听多个对象的不同属性，一个属性也可以被多个观察者对象监听。所以这里我使用字典`NSMutableDictionary`来保存观察者，以被观察的属性名为key值，所有该属性的观察者放到一个数组里`NSMutableArray`作为value。

大概是这个样子：      
![image](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/KVODictionary.png)


这样观察者数组和被观察属性对应，关系就很清晰。观察者之间互不干扰。最重要的是方便实现一对多关系，单纯用数组就没法去实现。这段代码就不贴出来了。

还有要保存下被观察属性的key值，在其他实例方法里会用到。
```
// 保证每个被观察属性的关联key值不同，可以用 setter 方法SEL类型作为关联key
objc_setAssociatedObject(self, setMethod, keyPath, OBJC_ASSOCIATION_COPY);

```
# 移除观察者

观察者和被观察者之间往往会形成循环引用。比如控制器持有Person对象，而person在添加控制器作为观察者时又会强引用控制器对象。
```
@property(nonatomic,strong)Person *person;
```
```
objc_setAssociatedObject(self, kLDKVOObserverDictionary, observerDictionary, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
```
如上代码，观察者持有被观察者person，person（self）强引用观察者字典，字典对象持有观察者，而构成循环引用。

当不再接收通知时，一定要移除观察者，破除循环引用。


# 其他


还有重写`class`方法来伪装成原类，跟重写`setter`方法是一样的逻辑，没有什么实际意义，感兴趣也可以自己写一下。KVO的基本实现就是这样，[完整代码请见LDKVO](https://github.com/alexiiio/LDKVO)，直接看代码会比较清楚。

[LDKVO](https://github.com/alexiiio/LDKVO)除了一些基本的功能，主要解决了两个问题。

1. 一对多关系。即一个观察者可以监听多个对象的不同属性，一个属性也可以被多个观察者对象监听。
2. 可以观察任意类型。OC对象类型和基本数据类型都可以。使用的时候注意下类型转换就好了。

另外，做了逻辑判断，重复添加或移除观察者都不会有问题。

**注意**：与系统KVO不兼容，即同一个被观察者对象不可以同时使用系统KVO和LDKVO，会死循环。不同被观察对象不影响。

自己实现KVO主要是用来理解KVO的实现原理，实现的过程用到了很多runtime的动态特性，也是对runtime的学习。如果实际项目使用KVO，可以看下Facebook的[KVOController](https://github.com/facebook/KVOController)。
