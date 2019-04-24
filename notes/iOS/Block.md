# 概念
Block：带有自动变量（局部变量）的匿名函数。

- 匿名函数是指不带函数名称的函数；
- “带有自动变量（局部变量）”，这是因为Block拥有捕获外部变量的功能。在Block中访问一个外部的局部变量，Block会持用它的临时状态，自动捕获变量值，外部局部变量的变化不会影响它的的状态。

> 在异步编程时（比如网络请求），我们需要在执行完异步操作再执行另一个函数，而我们并不知道前一个方法什么时候执行完，这时候就需要用到Block，将方法作为一个参数传递，在异步操作完成后回调该方法。

比如AFNetworking网络请求，把Block作为参数传递给方法，在网络请求之后进行回调:

```
[[AFHTTPSessionManager manager] POST:@"" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
   
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
}];
```
Block是添加到C、Objective-C和C++中的语言级特性，它也是运行时特性，允许你创建独特的代码段。这些代码段可以像传递值一样传递给方法或函数。**Block是OC对象**，这意味着Block可以被添加到集合中，比如NSArray或者NSDictionary。Block还会捕获局部变量的值，相当于其他编程语言中的闭包或lambda。

**Block是OC对象，所以它可以作为参数传递给方法，也可以存储，并且在多线程中使用。**

Block在OC中的实现：
```
struct Block_layout {
    void *isa;
    int flags;
    int reserved;
    void (*invoke)(void *, ...);
    struct Block_descriptor *descriptor;
    /* Imported variables. */
};

struct Block_descriptor {
    unsigned long int reserved;
    unsigned long int size;
    void (*copy)(void *dst, void *src);
    void (*dispose)(void *);
};
```

而Block代码块和普通函数的区别是：

> Block代码：是一个函数对象，是在程序运行过程中产生的；       
普通函数：是一段固定代码，产生于编译期；

[Block苹果文档1](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Block.html#//apple_ref/doc/uid/TP40008195-CH3-SW1)      
[Block苹果文档2](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8)

# Block的声明及定义语法

Block的标准声明，Xcode快捷键`inlineBlock`：   
![inlineBlock](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/inlineBlock.png)
```
返回值类型 (^Block变量名)(形参列表) = ^返回值类型 (形参列表){ 内容 }
```
等号左边是声明部分，右边是赋值部分。      
返回值和参数可以为void。    
声明部分可以省略形参名。    
在赋值部分，如果返回值或参数为void，可以省略。

```
// 有返回值有参数的block
int(^MyBlock)(int,int) = ^int (int a,int b){    
    NSLog(@"%d",a + b);
    return a + b; 
};  
MyBlock(12,56); // block使用
```


![block](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/blockdeclaration.png)

根据返回值和参数值的有无，还有以下几种变化：
```
//  根据返回值和参数值的有无，还有以下几种变化：
void(^MyBlock1)(int a) = ^void (int a){ };  // 无返回值有参数
void(^Myblock2)(int) = ^(int a){ };    // 省略写法

int (^Myblock3)(void) = ^int (void) { return 1;}; // 无参数有返回值
int (^Myblock4)(void) = ^int {return 1;};    // 省略写法

void (^Myblock5)(void) = ^void (void){ }; // 无参数无返回值
void (^Myblock6)(void) = ^{ };   // 省略写法
```

实际开发中常用`typedef`定义Block，Xcode快捷键`typedefBlock`：   
![typedefBlock](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/typedefBlock.png)

```
typedef NSString *(^PropertyBlock)(int);
```
这时，PropertyBlock就成为了一种Block类型，在定义类的属性时可以声明Block类型的属性：
```
@property(nonatomic,copy)PropertyBlock block;
```
赋值和使用，在赋值时要把省略的参数名补全：
```
self.block = ^NSString *(int a) {
    return [NSString stringWithFormat:@"%d",a];
};   
self.block(6);
```

Block作为方法的参数：
```
- (void)yourMethod:(return_type (^)(var_type))blockName;
```
例子：
```
- (void)addClickedBlock:(void(^)(id obj))clickedAction;
```

# Block的存储区域

按照在内存中的分布，Block分为3种：
- 全局block(_NSConcreteGlobalBlock)，存在于全局内存中, 相当于单例；
- 栈block(_NSConcreteStackBlock)，存在于栈内存中, 超出其作用域则马上被销毁；
- 堆block(_NSConcreteMallocBlock)，存在于堆内存中, 是一个带引用计数的对象，需要自行管理其内存。

[内存分配的5个区](http://note.youdao.com/noteshare?id=68f9491324ce8f85a1cd615c39baa71e)

## 怎么确定block的类型呢？（这点并没有去验证）

1. 不访问外部变量（包括栈和堆中的变量）的block，既不在栈也不在堆中，在代码段中，此时为全局block。
2. 访问外部变量的block
- MRC 环境下：访问外界变量的 Block 默认存储栈中。
- ARC 环境下：访问外界变量的 Block 默认存储在堆中（实际是放在栈区，然后ARC情况下自动拷贝到堆区），自动释放。

访问了外界变量的Block，最开始存放在栈内存。在Block被赋值之后，就会自动调用copy方法把Block复制到堆内存。


# ARC下，访问外界变量的 Block为什么要自动从栈区拷贝到堆区呢？

栈上的Block，如果其所属的变量作用域结束，该Block就被废弃，如同一般的自动变量。当然，Block中的__block变量也同时被废弃。

为了解决栈Block在其变量作用域结束之后被废弃（释放）的问题，我们需要把Block复制到堆中，延长其生命周期。开启ARC时，大多数情况下编译器会恰当地进行判断是否有需要将Block从栈复制到堆，如果有，自动生成将Block从栈上复制到堆上的代码。Block的复制操作执行的是copy实例方法。Block只要调用了copy方法，栈Block就会变成堆Block。

# Block为什么使用copy修饰
block默认存储在栈中，超出其作用域就会自动销毁，如果在block创建的作用域外调用block就会崩溃，使用copy可以把block从栈内存拷贝到堆内存，延长其生命周期。而在ARC中系统会默认对block进行copy操作，拷贝到堆上，这时使用`strong`修饰也是可以的。不过为了block属性声明和实际的操作一致，最好声明为`copy`。

苹果说明如下：

![blockusecopy](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/blockusecopy.png)

# 捕获局部变量

除了包含可执行代码之外，Block还能够从其封闭的范围捕获状态。

例如，在Block之外声明了一个整数，但是在定义Block时捕获该值。除非另外指定，否则只捕获值。这意味着，如果在定义Block和调用Block的时间之间更改变量的外部值，如下所示:

```
  int anInteger = 42;
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
    anInteger = 84;
    testBlock();
```
Block捕获的值不受影响。日志输出仍然显示:
```
Integer is: 42
```
默认情况下，Block不能修改捕获的值，因为Block捕获的是自动变量的const值，并非内存地址，所以Block内部不能改变自动变量的值。Block捕获的外部变量可以改变值的是静态变量，静态全局变量，全局变量。    
![ima](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/blockCaptureValues.jpeg)

**使用__Block修饰的自动变量**

如果需要在Block里修改捕获的变量的值，则可以在原始变量声明时使用`__block`修饰符。对于用`__block`修饰的外部变量引用，block是复制其引用地址来实现访问的。    
![image](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/blockCaptureValues__block.jpeg)

使用`__block`复制变量的地址之后，相当于对变量进行了强引用，就不用担心变量会提前释放的问题。

# Block引起的循环引用
某个类将 block 作为自己的属性变量，然后该类在 block 的方法体里面又使用了该类本身。比如下面的代码：
```
   self.block = ^{
        self.num = 1;
    }; 
```
self持有block，而堆上的block又会持有self，所以会导致循环引用。解决办法是通过`__weak`去声明一个弱引用的self:
```
_weak typeof(self) weakSelf = self;
self.block = ^{
    //加一下强引用，避免weakSelf提前被释放掉
    __strong typeof(weakSelf) strongSelf = weakSelf;
    //不会导致循环引用.
    if(strongSelf){
        strongSelf.num = 1;
    }
};
```
如果 self 并没有直接或者间接持有 block，就不会造成循环引用。比如调用系统的方法：
```
[UIView animateWithDuration:0.5 animations:^{
    NSLog(@"%@", self);
}];
```
因为这个block存在于静态方法中，虽然block对self强引用着，但是self却不持有这个静态方法，所以完全可以在block内部使用self。

# Block引起的释放不及时问题

发起网络请求的时候，在网络请求回调的 block 里强引用 ViewController，以便在网络请求回来的时候刷新界面。在网络请求比较慢的情况下，这种做法存在两个问题：

- ViewController 被 pop 之后，由于被 block 强引用导致释放不及时。在请求落地之后 ViewController 才会释放。
- ViewController 被 pop 之后，如果网络请求回来了，不应该继续做刷新界面的事，浪费 CPU

所以，对于这种情况，我们应该在 block 里弱引用 ViewController，而不是强引用。

# Block和代理的区别

Block和代理都是回调的方式。区别在于：

1. Block集中代码块，而代理分散代码块。所以Block更适用于轻便、简单的回调，如网络传输。而代理适用于公共接口较多的情况，这样做也更易于解耦代码架构。
2. Block运行成本高。Block出栈时，需要将使用的数据从栈内存复制到堆内存，当然，如果是对象就是加计数，使用完成或者Block置为nil后才消除；代理只是保存了一个对象指针，直接回调，并没有额外消耗。相对C的函数指针，只是多做了一个查表动作。

下面这个问题很相似。
# Block优缺点

block是一种轻量级的回调，使用简单,可以直接访问上下文，block集中代码块，并且不需要声明协议和代理，这样读代码也连贯。

缺点：
- blcok的运行成本高。block出栈需要将使用的数据从栈内存拷贝到堆内存，当然对象的话就是引用计数加1，使用完或者block置nil后才销毁。delegate只是保存了一个对象指针(一定要用weak修饰delegate，不然也会循环引用)，直接回调，没有额外消耗。就像C的函数指针，只多做了一个查表动作。
- block会延长对象的生命周期。block对全局变量是强引用，如果是异步耗时操作，比如网络请求，可能导致延长变量的生命周期。

- block很难追踪，难以维护
- block容易造成循环引用，而且不易察觉（注意可以避免）。因为为了blcok不被系统回收，所以我们都用copy关键字修饰，实行强引用。block对捕获的变量也都是强引用，所以就会造成循环引用。
