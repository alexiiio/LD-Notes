# 什么是RunLoop？

**RunLoop是通过内部维护的事件循环来对事件/消息进行管理的一个对象**，它是线程的基础架构部分，每一个线程都有一个与之对应的runloop对象，一个runloop就是一个事件处理循环，用来不停地调配工作和处理输入事件。使用runloop的目的是使线程在有工作的时候工作，没有的时候休眠。在程序启动时，主线程会自动创建并运行runloop，而其他线程的runloop需要手动启动。  
**RunLoop就是控制线程生命周期并接收事件进行处理的机制**。
# iOS中的RunLoop
iOS系统中，提供了两种RunLoop：**NSRunLoop**和**CFRunLoopRef**。  
CFRunLoopRef是在CoreFundation框架内的，它提供了纯C函数的API，是线程安全的。    
NSRunLoop是基于 CFRunLoopRef 的封装，提供了面向对象的API，但是这些API不是线程安全的。   
CFRunLoopRef 的代码是开源的。   

# 为什么要使用RunLoop？
当有持续的异步任务需求时，我们需要**创建一个独立的生命周期可控的线程**。（RunLoop就是控制线程生命周期并接收事件进行处理的机制。）


# RunLoop结构

RunLoop结构图如下。    


![RunLoop结构](https://upload-images.jianshu.io/upload_images/7929400-e0120322b8bc6bdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500) 

一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。

> 每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。

> Mode之间可以切换。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

> Source/Timer/Observer 被统称为 mode item，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。


# Mode
Mode即为行为模式。包含一系列事件、动作。

Mode一共有几种模式，网上众说纷纭。这里可以参考苹果文档 [NSRunLoopModel](https://developer.apple.com/documentation/foundation/nsrunloop/run_loop_modes?language=objc) 

其中：
 > **NSConnectionReplyMode**     
 > **NSModalPanelRunLoopMode**     
 > **NSEventTrackingRunLoopMode**       

然而以上3种都是macOS的，具体含义就不想解释了。

## iOS上常用的3种mode
1. **kCFRunLoopDefaultMode**（CFRunLoop）/**NSDefaultRunLoopMode**（NSRunLoop）  默认模式，没有指定模式时就运行在该模式下。一般情况下app都是运行在该模式下。

官方注释是：用来处理非NSConnection对象的输入源的模式。

2. (CFStringRef)**UITrackingRunLoopMode**(CFRunLoop)/**UITrackingRunLoopMode**(NSRunLoop) 界面跟踪 Mode,一般作用于ScrollView滚动的时候，保证滑动的时候不受其他事件影响。

3. **kCFRunLoopCommonModes**（CFRunLoop）/**NSRunLoopCommonModes**（NSRunLoop）
这是一个占位用的Mode，作为标记NSDefaultRunLoopMode和UITrackingRunLoopMode用，并不是一种真正的Mode。在主线程中默认包含了NSDefaultRunLoopMode和 UITrackingRunLoopMode。子线程中只包含NSDefaultRunLoopMode。

还有2种在网上资料比较常见到：    

> **UIInitializationRunLoopMode** : 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用。   
> **GSEventReceiveRunLoopMode** : 接受系统事件的内部 Mode，通常用不到。


苹果内部还有[更多Mode](http://iphonedevwiki.net/index.php/CFRunLoop)，但在开发中很难碰到。这些加起来都有二十多种Mode了，所以到底RunLoop有几种Mode，我们无从得知，目前来看也并不需要知道。只需要记得常用的3种就可以了。

# Source

source就是输入源事件，分为source0和source1两种。

> 1. source0：非基于Port的**用于用户主动触发的事件，诸如UIEvent（触摸，滑动等），performSelector这种需要手动触发的操作**。
> 2. source1：基于Port的**通过内核和其他线程相互发送消息**。处理系统内核的mach_msg事件（系统内部的端口事件）。诸如**唤醒RunLoop或者让RunLoop进入休眠节省资源等**。

> - Source0 只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。
 
> - Source1 包含了一个 mach_port 和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程。

一般来说日常开发中我们需要关注的是source0，source1只需要了解。
日常开发中，我们需要对常驻线程进行操作的事件大多都是source0。

注意：Source1在处理的时候会分发一些操作给Source0去处理


# Timer

 Timer即为定时源事件。通俗来讲就是我们很熟悉的NSTimer（其实就是CFRunloopTimerRef,底层基于使用mk_timer实现），其实NSTimer定时器的触发正是基于RunLoop运行的，所以使用NSTimer之前必须注册到RunLoop，但是RunLoop为了节省资源并不会在非常准确的时间点调用定时器，如果一个任务执行时间较长，那么当错过一个时间点后只能等到下一个时间点执行，并不会延后执行。      
 
 NSTimer的创建通常有两种方式，尽管都是类方法，一种是timerWithXXX，另一种scheduedTimerWithXXX。二者最大的区别就是后者除了创建一个定时器外会自动以NSDefaultRunLoopMode添加到当前线程RunLoop中，不添加到RunLoop中的NSTimer是无法正常工作的。
 
 
 # Observer
 
 它相当于消息循环中的一个监听器，随时通知外部当前RunLoop的运行状态。NSRunLoop没有相关方法，只能通过CFRunLoop相关方法创建。
 ```
 // 创建observer
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {

        NSLog(@"----监听到RunLoop状态发生改变---%zd", activity);

    });

    // 添加观察者：监听RunLoop的状态
    CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);
 ```
 
 RunLoop的几种运行状态
 ```
 /* Run Loop Observer Activities */
    typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
        kCFRunLoopEntry = (1UL << 0), // 进入RunLoop 
        kCFRunLoopBeforeTimers = (1UL << 1), // 即将开始Timer处理
        kCFRunLoopBeforeSources = (1UL << 2), // 即将开始Source处理
        kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
        kCFRunLoopAfterWaiting = (1UL << 6), //从休眠状态唤醒
        kCFRunLoopExit = (1UL << 7), //退出RunLoop
        kCFRunLoopAllActivities = 0x0FFFFFFFU
    };

 ```
 
  # RunLoop运行流程
 
 ![流程图](https://upload-images.jianshu.io/upload_images/7929400-b57d3df8fac3c234.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

 
 # Call out
 
当 RunLoop 进行回调时，一般都是通过一个很长的函数调用出去 (call out)。在开发过程中几乎所有的操作都是通过Call out进行回调的(无论是Observer的状态通知还是Timer、Source的处理)，而系统在回调时通常使用如下几个函数进行回调(换句话说你的代码其实最终都是通过下面几个函数来负责调用的，即使你自己监听Observer也会先调用下面的函数然后间接通知你)，所以在调用堆栈中经常看到这些函数：
 
 ```
     static void __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__();
    static void __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__();
    static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__();
 ```
 call out函数定义这么长就是为了让我们见名知意，结合RunLoop的内部逻辑，整理如下：
 ```
{
    /// 1. 通知Observers，即将进入RunLoop
    /// 此处有Observer会创建AutoreleasePool: _objc_autoreleasePoolPush();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopEntry);
    do {
 
        /// 2. 通知 Observers: 即将触发 Timer 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeTimers);
        /// 3. 通知 Observers: 即将触发 Source (非基于port的,Source0) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeSources);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
 
        /// 4. 触发 Source0 (非基于port的) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__(source0);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
 
        /// 6. 通知Observers，即将进入休眠
        /// 此处有Observer释放并新建AutoreleasePool: _objc_autoreleasePoolPop(); _objc_autoreleasePoolPush();
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeWaiting);
 
        /// 7. sleep to wait msg.
        mach_msg() -> mach_msg_trap();
        
 
        /// 8. 通知Observers，线程被唤醒
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopAfterWaiting);
 
        /// 9. 如果是被Timer唤醒的，回调Timer
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__(timer);
 
        /// 9. 如果是被dispatch唤醒的，执行所有调用 dispatch_async 等方法放入main queue 的 block
        __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(dispatched_block);
 
        /// 9. 如果如果Runloop是被 Source1 (基于port的) 的事件唤醒了，处理这个事件
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__(source1);
 
 
    } while (...);
 
    /// 10. 通知Observers，即将退出RunLoop
    /// 此处有Observer释放AutoreleasePool: _objc_autoreleasePoolPop();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopExit);
}
 ```






# RunLoop的应用场景

1. 维护线程的生命周期,让线程不自动退出，在完成任务或达到某个条件时退出。
``` 
   NSRunLoop *runloop = [NSRunLoop currentRunLoop];
    [runloop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    while (!self.isCancelled && !self.isFinished) {
        @autoreleasepool {
            [runloop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:10]];
        }
    }
```
2. 创建常驻线程，执行一些会一直存在的任务。该线程的生命周期跟App相同。
``` 
    @autoreleasepool {
        NSRunLoop *runloop = [NSRunLoop currentRunLoop];
        [runloop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runloop run];
    }
```
3. 在一定时间内监听某种事件或执行某种任务的线程。如下，在30分钟内，每隔30s执行onTimerFired：。
```
    @autoreleasepool {
        NSRunLoop *runloop = [NSRunLoop currentRunLoop];
        NSTimer *updateTimer = [NSTimer timerWithTimeInterval:30 target:self selector:@selector(onTimerFired:) userInfo:nil repeats:YES];
        [runloop addTimer:updateTimer forMode:NSRunLoopCommonModes];
        [runloop runUntilDate:[NSDate dateWithTimeIntervalSinceNow:60*30]];
    }
```
# 特性
- 主线程的RunLoop在程序启动时就会自动创建并启动，子线程默认没有RunLoop，需要手动获取并启动。  
- 只能在一个线程的内部获取其 RunLoop（主线程除外）。
- 一个线程只有一个唯一的RunLoop与之对应，但是与线程对应的那个RunLoop可以包含子RunLoop。
- RunLoop的本质就是do while循环。只是不同于我们自己写的循环它在休眠时几乎不会占用系统资源。
- RunLoop是懒加载方式创建的。只有当我们使用线程的方法主动get Runloop时才会在第一次创建该线程的Runloop，同时将它保存在全局的Dictionary中（线程和Runloop二者一一对应），默认情况下线程并不会创建Runloop（主线程的Runloop比较特殊，任何线程创建之前都会保证主线程已经存在Runloop），同时在线程结束的时候也会销毁对应的Runloop。  
- NSTimer的创建方法中 scheduledTimerWithTimeInterval：会自动以NSDefaultRunLoopMode把timer添加到当前线程RunLoop中。  
- RunLoop想要正常启用需要运行在添加了事件源的Mode下。RunLoop有三种启动方式run、runUntilDate:(NSDate *)limitDate、runMode:(NSString *)mode beforeDate:(NSDate *)limitDate。第一种无条件永远运行RunLoop并且无法停止，线程永远存在。第二种会在时间到后退出RunLoop，同样无法主动停止RunLoop。前两种都是在NSDefaultRunLoopMode模式下运行。第三种可以选定运行模式，并且在时间到后或者触发了非Timer的事件后退出。  
- RunLoop并不是线程安全的，所以需要避免在其他线程上调用当前线程的RunLoop。  
- RunLoop负责管理autorelease pools。子线程的autoreleasepool需要我们手动创建。NSThread和NSOperationQueue开辟子线程需要手动创建autoreleasepool。GCD开辟子线程不需要手动创建autoreleasepool，因为GCD的每个队列都会自行创建autoreleasepool。
- RunLoop每次只能运行在一个Mode下，其意义是让不同Mode中的item互不影响。
- RunLoop可以切换Mode，但是因为RunLoop不会处理非当前Mode的item，之前处理的item不会再执行。可以把一个item同时加入多个Mode中来解决。
- Runloop 依赖 mach_msg 进行进程间通信，也就是消息接收发送。


# 线程间怎么进行通信？

线程间通信一般是指：
- 1个线程传递数据给另1个线程
- 在1个线程中执行完特定任务后，转到另1个线程继续执行任务

线程间通信的常用方法：
```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
```
```
//开启一个全局队列的子线程
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
            //1. 开始请求数据
            //...
            // 2. 数据请求完毕
            //我们知道UI的更新必须在主线程操作，所以我们要从子线程回调到主线程
        dispatch_async(dispatch_get_main_queue(), ^{
                //回到主线程更新
        });
    });
```
还有**基于端口（NSMachPort）的通信。**

**线程间通信的原理是什么呢？**

线程间的通信（不仅限于通信，几乎所有iOS事件都是如此），实际上是各种输入源，触发runloop去处理对应的事件。Runloop 依赖 mach_msg 进行进程间通信，也就是消息接收发送。

回想一下，在开启RunLoop的方法里：
```
NSRunLoop *runloop = [NSRunLoop currentRunLoop];
[runloop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
[runloop run];
```
runloop添加了一个`NSMachPort`类的对象。`NSMachPort`是用作分布式对象连接(或原始消息传递)端点的端口。`NSMachPort`是`NSPort`的一个子类，它封装了Mach端口，Mach端口是macOS中的基本通信端口。

简单说一下Mach。

Mach是iOS的XNU内核中最为核心的部分，在Mach中所有东西（Task、进程、线程、虚拟内存等）都是对象。对象与对象之间通信只能通过端口（port）收发消息。对象之间的通信采用消息传递的方式，遵从FIFO原则。用户态的Mach消息传递使用的是mach_msg()函数。

Runloop通过mach_msg()函数发送消息，如果没有port 消息，内核会将线程置于等待状态 mach_msg_trap() 。如果有消息，判断消息类型处理事件，并通过call out函数回调。

也就是说**线程间是通过Mach port通信，采用mach_msg()消息传递的方式来完成的，而通信的发起者是runloop**。



# 参考资料
https://www.jianshu.com/p/18afd628ac43?utm_source=desktop&utm_medium=timeline    
https://www.cnblogs.com/zy1987/p/4582466.html  
https://blog.ibireme.com/2015/05/18/runloop/   
https://www.cnblogs.com/kenshincui/p/6823841.html       
https://www.cnblogs.com/kenshincui/p/6823841.html      

