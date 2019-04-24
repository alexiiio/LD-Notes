# 概念
**Runtime是一个运行时系统，用来执行编译链接后的可执行文件；它将很多静态语言在编译和链接时期做的事放到了运行时来处理。这样我们写代码更具灵活性，可以动态地创建类和对象、进行消息传递和转发。**

Runtime 基本是用 C 和汇编写的，可见苹果为了动态系统的高效而作出的努力。你可以在[这里](https://opensource.apple.com/source/objc4/)下到苹果维护的开源代码。苹果和GNU各自维护一个开源的runtime版本，这两个版本之间都在努力的保持一致。


**静态语言**：如C语言，编译阶段就要决定调用哪个函数，如果函数未实现就会编译报错。    
**动态语言**：如OC语言，编译阶段并不能决定真正调用哪个函数，只要函数声明过即使没有实现也不会报错。      

我们常说OC是一门动态语言，就是因为它总是把一些决定性的工作从编译阶段推迟到运行时阶段。OC代码的运行不仅需要编译器，还需要运行时系统(Runtime Sytem)来执行编译后的代码。         

**Runtime是一套底层纯C语言API，OC代码最终都会被编译器转化为运行时代码，通过消息发送机制决定函数调用方式，这也是OC作为动态语言使用的基础**。


# Runtime消息发送机制

**所有OC方法的调用都是通过消息发送机制实现的**。    

## 消息传递

OC的方法调用都是类似[obj foo]的形式，编译器转成消息发送objc_msgSend(obj, foo)，Runtime时执行的流程是这样的：
- 首先，通过obj的isa指针找到它的 class ;
- 在 class 的 method list 找 foo ;
- 如果 class 中没到 foo，继续往它的 superclass 中找 ;
- 一旦找到 foo 这个函数，就去执行它的实现IMP 。

如果找不到则会沿着继承树向上一直搜索直到继承树根部（通常为NSObject），如果还是找不到并且**消息转发**都失败了就会执行`doesNotRecognizeSelector:`方法报`unrecognized selector`错误。

## 消息转发
如果消息传递到继承树根部依然找不到要调用的方法，就会启动3次消息转发：
- 动态方法解析；重写`+resolveInstanceMethod:`或者`+resolveClassMethod:`动态添加方法，并返回YES。
- 备用接收者；resolve返回NO时，到这一步。如果目标对象实现了`-forwardingTargetForSelector:`，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。
- 完整消息转发。首先它会调用`-methodSignatureForSelector:`来获得方法签名。如果`-methodSignatureForSelector:`返回nil ，Runtime则会发出 `-doesNotRecognizeSelector:` 消息，程序这时就会崩溃。可以返回一个我们手动生成的方法签名，然后Runtime就会创建一个`NSInvocation` 对象并调用 `-forwardInvocation:`方法，在这里把消息转发给其他对象。`-forwardInvocation:`转发消息跟第二步类似，可以在这里改变消息内容、追加参数等。

![methodForwding](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/methodForwding.jpg)

**说明：OC调用方法[receiver selector]，编译阶段确定了要向哪个接收者发送message消息，但是接收者如何响应决定于运行时的判断。**


`objc_class`中另一个重要成员`objc_cache`。`objc_cache`用来缓存常用函数。 在找到foo 之后，把foo 的`method_name` 作为`key` ，`method_imp`作为`value`给存起来。当再次收到foo 消息的时候，可以直接在cache 里找到，避免去遍历`objc_method_list`。

# isa指针

**isa是指向对象所属的类的指针**。在Objective-C中，所有的类自身也是一个对象，类对象在编译期产生，用于创建实例对象，是单例。类对象的Class里面也有一个isa指针，它指向`metaClass`(元类)。

即：**一个类实例的isa指针指向该类对象（Class），类对象的isa指针指向类的元类`metaClass`。**

元类（`metaClass`）存储着一个类的所有类方法。每个类都会有一个单独的`metaClass`，因为每个类的类方法基本不可能完全相同。`metaClass`也是一个类，也可以向它发送一个消息，那么它的isa又是指向什么呢？为了不让这种结构无限延伸下去，Objective-C的设计者让所有的`metaClass`的isa指向基类的`metaClass`，以此作为它们的所属类。

- 任何`NSObject`继承体系下的`metaClass`都使用`NSObject`的`metaClass`作为自己的所属类；
- 而`NSObject`的`metaClass`的`isa`指针是指向它自己。
- `NSObject metaClass`的`superClass`是`NSObject`。


整个结构如下图所示:

![isa](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/isa.png)

**isa指针的作用**是帮助一个对象找到它的方法。
# runtime应用

- 关联对象(Objective-C Associated Objects)[给分类增加属性](http://note.youdao.com/noteshare?id=a0ec4c0a111be45e69b87a2e77af3a4e)
- (Method Swizzling)方法添加和替换，如[KVO实现](http://note.youdao.com/noteshare?id=d8b798e97282ce967ee079a736d8fdc1)
- 消息转发，如热更新解决Bug(JSPatch)
- 实现NSCoding的自动归档和自动解档
- 实现字典和模型的自动转换(MJExtension)

