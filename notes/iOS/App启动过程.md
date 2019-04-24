一个 iOS App 的 `main` 函数位于 main.m 中，是整个程序的入口，在程序启动之前，系统会调用`exec()`函数。程序在进入 `main` 函数前已经执行了很多代码，比如熟知的 `+ load` 方法等。

# main函数执行前

http://blog.sunnyxx.com/2014/08/30/objc-pre-main/

程序会做一系列的初始化工作，动态加载依赖库

**dyld**：（the dynamic link editor），Apple 的动态链接器

**ImageLoader**：image表示一个二进制文件（可执行文件或 so 文件），里面是被编译过的符号、代码等，所以 ImageLoader 作用是将这些文件加载进内存，且每一个文件对应一个ImageLoader实例来负责加载。

> 1. 首先当程序启动时，系统`exec()`函数会读取程序的可执行文件（mach-o）, 从里面获取动态加载器(dyld)的路径;
> 2. 加载dyld, dyld会初始化运行环境，配合ImageLoader将二进制文件加载到内存中去;
> 3. 动态链接依赖库, 初始化依赖库，初始化 runtime;
> 4. runtime 会对项目中的所有类进行类结构初始化，调用所有的 load 方法;
> 5. 最后 dyld 会返回 main 函数地址，main 函数被调用，进入程序入口

# main函数执行之后


> 1. 内部会调用 UIApplicationMain 函数，创建一个UIApplication对象和它的代理Appdelegate
> 2. 开启一个事件循环(main runloop), 监听系统事件
> 3. 读取info.plist文件。寻找是否Main,有则进入Main storyboard.
> 4. 通知代理Appdelegate, 调用 didFinishLaunching 代理方法，在这里会创建 UIWindow,设置它的rootViewController,
> 5. 最后调用 self.window makeKeyAndVisable显示窗口


`UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]))` 函数有四个参数：


> 1. int argc: 表示参数的个数
> 2. char * _Nonnull * _Null_unspecified argv: 表示装载函数的数组
> 3. NSString * _Nullable principalClassName: 表示 UIApplication类名或子类，若为 nil,则默认使用 UIApplication 类名
> 4. NSString * _Nullable delegateClassName: 表示协议UIApplicationDelegate 的实例化对象名，这个类就是我们熟悉的 AppDelegate



**AppDelegate**加载顺序
1. application:didFinishLaunchingWithOptions:
2. applicationDidBecomeActive:

**ViewController**中的加载顺序
1. loadView
2. viewDidLoad
3. viewWillAppear
4. viewWillLayoutSubviews
5. viewDidLayoutSubviews
6. viewDidAppear

**View**中的加载顺序
1. initWithCoder（如果没有storyboard就会调用initWithFrame，这里两种方法视为一种）
2. awakeFromNib
3. layoutSubviews
4. drawRect


# 启动时间优化

main函数执行前的优化：

1. 移除不必要的动态库
2. 移除不必要的类
3. 合并功能类似的类和扩展（Category）
4. 压缩资源图片

在Environment Variables中添加`DYLD_PRINT_STATISTICS`字段，并设置为YES，在控制台就会打印加载时长。

main函数执行后的优化：

1. 优化applicationWillFinishLaunching
2. 优化rootViewController加载
3. 减少启动时网络请求，防止拥堵
4. 纯代码方式而不是storyboard加载首页UI
5. 考虑延迟加载或者懒加载

可使用Time Profiler查看程序启动时各个方法的耗时，进行针对性优化。
