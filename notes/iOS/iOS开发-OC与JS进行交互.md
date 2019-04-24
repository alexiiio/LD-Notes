本篇文章只讲述UIWebView和JS进行交互，WKWebView不做叙述。iOS7之后苹果开放了JavaScriptCore框架，该框架处理Objective-C和JavaScript之间的交互很是简便易用。在GitHub上也有WebViewJavaScriptBridge这个第三方封装库，本文并未涉及。

[demo地址](https://github.com/alexiiio/iOS-JavaScriptCore-demo)
# 准备工作
引入框架

```
#import <JavaScriptCore/JavaScriptCore.h>
```
JavaScriptCore框架下有以下类

![JavaScriptCore.png](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/JavaScriptCoreClasses.png)

**JSContext**：每个JSContext对象代表一个JS执行环境。     
**JSValue**: 是OC和JavaScript值相互转化的桥梁，提供很多数据类型的转化方法。        
**JSManagedValue**：一个 JSManagedValue 对象包装一个 JSValue 对象，JSManagedValue 对象通过添加“有条件的持有（conditional retain）”行为来实现values的自动内存管理。     
**JSVirtualMachine**：JS运行的虚拟机。一个JSVirtualMachine对象代表了一个独立的JavaScript执行环境。用来支持并行的JavaScript执行，管理JavaScript和OC(或Swift)转换中的内存管理。    
**JSExport**：一个协议，它提供一种方法把OC的方法和对象转换成JS代码。    

使用UIWebView加载html，本例使用本地html。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.webView = [[UIWebView alloc]initWithFrame:[UIScreen mainScreen].bounds];
    [self.view addSubview:self.webView];
    self.webView.delegate = self;
    NSString *path = [[[NSBundle mainBundle] bundlePath]  stringByAppendingPathComponent:@"index.html"];
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL fileURLWithPath:path]];
    [self.webView loadRequest:request];
}
```
在webView的代理方法里获取JS执行环境
```
-(void)webViewDidFinishLoad:(UIWebView *)webView{
    self.title = [webView stringByEvaluatingJavaScriptFromString:@"document.title"];
         
    // 通过 UIWebView 来获取 JSContext ，通过获取到的 context 来执⾏ JS 代码。

    self.context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];

    self.context[@"Alex"] = self;
    
    self.context.exceptionHandler =
    ^(JSContext *context, JSValue *exceptionValue)
    {
        context.exception = exceptionValue;
        NSLog(@"%@", exceptionValue);
    };
}
```
   
 `self.context[@"Alex"] = self;` 这一步是把对象传到JS环境，用来调用self对象实现的方法。
JS的运行环境中需要设置一个能调用OC方法的对象，即`self`,这个对象按照约定命名为`Alex`(这个名字是跟前端约定好的，没有特殊要求)。最好把JS要调用的方法写到另一个类里，把实现类的实例对象传给JS环境。另外，直接使用block给context对象添加的方法，不需要传调用对象。

 
在实际项目中，由于web页面可能在页面加载的时候就调用OC的方法，而我们在页面加载完成时才把native和web关联，会无法调用。所以把上述代码写到`webViewDidStartLoad`方法里更为稳妥。

# JS调用OC方法
可能是最常用的JS交互方式了，我们写好方法，web页面可以直接使用方法名来调用。这就好比app与后台进行通讯，平时都是后台写好接口，等app来调用，现在反过来了，接口我们来写。我们需要定义好方法名（相当于接口的url），参数和返回值，把这些写成规范。

供JS调用的OC方法有不同写法
### 使用block

```
    // JS调用OC方法
    self.context[@"showLog"] = ^(){
        NSArray *args = [JSContext currentArguments]; // 获取参数
        for (JSValue *jsVal in args) {
            NSLog(@"%@", jsVal.toString);
        }
    };
    
    __block typeof(self) weakSelf = self;
    self.context[@"addSubView"] = ^{
        dispatch_async(dispatch_get_main_queue(), ^{
            UIView *view = [[UIView alloc]initWithFrame:CGRectMake(10, 250, 100, 100)];
            view.backgroundColor = [UIColor greenColor];
            [weakSelf.view addSubview: view];
        });
    };
```
### 使用JSExport协议
JSExport是两种语言的互通协议。JSExport中没有约定任何的方法，所有继承于该协议的协议，所声明的方法和属性，都可以被JS调用。方法传参数，获取返回值都不是问题。直接上例子，声明一个遵守了`JSExport`协议的协议：

```
@protocol TestJSExport <JSExport>
-(NSString *)getUserInfo;
@end

@interface ViewController : UIViewController<TestJSExport>
@property (strong, nonatomic)  UIWebView *webView;
@property (strong, nonatomic) JSContext *context;

@end
```
写在协议内的如`getUserInfo`方法就可以被JS直接调用了
或者使用**JSExportAs**宏定义创建方法，这样方法名会更短。该写法要按固定格式来写，否则会报错。

```
@protocol TestJSExport <JSExport>
// getUserInfo、testArgumentTypes被转化为方法名
JSExportAs(getUserInfo,
           -(NSString *)getUser_info:(NSString *)saw
           );

JSExportAs(testArgumentTypes,
           - (NSString *)testArgumentTypesWithInt:(int)i double:(double)d
           boolean:(BOOL)b string:(NSString *)s number:(NSNumber *)n
           array:(NSArray *)a dictionary:(NSDictionary *)o
           );

-(NSString *)getUserInfo;
@end
```

```
-(NSString *)getUserInfo{
    return @"dsy621fa3";
}
```


index.html里的代码举例（实际会由前端工程师负责）：

```
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
        <title>测试程序</title>
    </head>
    <body>
        <div id="bg"></div>
        <br></br>
        测试OC和JS交互
        <br/>
        <button style="height:40px;width:150px;" onClick="showLog('hello world')">JS调用OC_打印log</button>
            <br/>
            <br></br>
            <button style="height:40px;width:150px;" onClick="callMethodAddSubView()">JS调用OC_添加视图</button>
            <br></br>
                <button style="height:40px;width:150px;" onClick="Person.sayHi()">JS调用Person SayHi</button>
                <br></br>
                    <input type="text" id="input_text" name="input_text">
                        <script type="text/javascript">
<!--                            OC里实现的同名方法会覆盖掉该方法。-->
                            function showLog(){
                                window.Alex.showLog();
                            }
<!--                        可以在点击按钮时先调用js自己的方法，做完某些操作之后再调用OC里的方法。-->
                            function callMethodAddSubView(){
                                alert("添加了一个view");
                                document.getElementById("input_text").value = 'addSubView';
                                window.Alex.addSubView();
                            }
                        
                            function changeColor(){
                                var ranColor = '#' + ('00000' + (Math.random() * 0x1000000 << 0).toString(16)).slice(-6); //随机生成颜色
                                document.bgColor = ranColor;
                                document.getElementById("input_text").value = '背景色：'+ranColor;
                                return ranColor;
                            }
                        </script>
                        </body>
</html>

```
其中`window.Alex.showLog()`，`Alex`就是开始设置的webview类对象，`showLog`则是我们OC里写好的方法名。

运行工程，显示效果如图：

![index.png](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/OC&JSpicture1.png)

点击按钮后，在控制台正常输出OC相应方法的打印信息：
```
hello world
hello,how are you?
```
页面上也添加上了view：

![JS调用OC.png](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/JSCallOC.png)

具体查看demo。
# OC调用JS
无论是OC调JS，还是JS调OC，都是直接使用方法名。先写一个按钮来触发JS方法，详见demo。
我们示例两个方法，调用JS的`showLog`和`changeColor`方法，分别是打印信息和改变界面背景色。`showLog`在加载webview完成时调用，查看控制台看效果。
```
    // OC调用JS方法
    NSString *fs=@"showLog('OC调用JS方法showLog')";
    [self.context evaluateScript:fs];
```
`changeColor`在点击按钮时调用：
```
// OC调用JS方法
-(void)callJS_ChangeColor{
    JSValue *function = [self.context objectForKeyedSubscript:@"changeColor"];
    JSValue *color = [function callWithArguments:nil];
    NSLog(@"背景色：%@",color);
}
```
点击改变背景色如图：

![JS调用OC.png](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/OCCallJS.png)

这里展示了两种调用JS的方法，要遵循JS方法的调用方式，参数使用单引号 ' ,多个参数中间加逗号。实践出真知，自己动手写几个例子就会明了了。


# 在实际场景的应用
常用的JS获取app里的用户信息，使用手机相机相册之类的，看过上面这些，实现上都不是问题。

举个在公司里的例子，web页面调用本地支付。首先把支付宝、微信支付封装起来便于调用，留一个调用的方法。JS调用该方法，传过来商品信息，app弹出支付页面，并调起相应的支付方式进行支付，支付完成之后要把支付结果返回给JS，我的做法是在获取到支付结果时，主动调用JS写好的接收支付结果的方法。这就是所谓的回调模式。JS在收到支付结果后刷新界面。

流程图：

![JS调本地支付流程图.png](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/JSCallNativePay.png)

这里用到了3次交互，iOS开发者负责2个方法，前端工程师负责1个方法。只要细心，团队共同调试，还是很容易实现的。再次提示，文档一定要规范明确。
#遇到的bug 
### Alert问题
上个例子中，最后web页面刷新后弹出alert 提示用户。然而alert没弹出来，界面也卡死了，怎么点击都没反应。百度一番，貌似是web的alert方法在iphone上会有bug。解决方法是拦截web的alert，转用iOS原生的alert 把信息显示出来。

给UIWebView添加一个类目

```
#import <Foundation/Foundation.h>

@interface UIWebView (JavaScriptAlert)
-(void) webView:(UIWebView *)sender runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame;
- (BOOL)webView:(UIWebView *)sender runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame;
@end
```

```
#import "UIWebView+JavaScriptAlert.h"

@implementation UIWebView (JavaScriptAlert)

- (void)webView:(UIWebView *)sender runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame {
    UIAlertView* customAlert = [[UIAlertView alloc] initWithTitle:message message:@"" delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil];
    [customAlert show];
}
static BOOL diagStat = NO;
static NSInteger bIdx = -1;
- (BOOL)webView:(UIWebView *)sender runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(id)frame {
    UIAlertView *confirmDiag = [[UIAlertView alloc] initWithTitle:message
                                                          message:@""
                                                         delegate:self
                                                cancelButtonTitle:@"取消"
                                                otherButtonTitles:@"确定", nil];
    
    [confirmDiag show];
    bIdx = -1;
    
    while (bIdx==-1) {
        //[NSThread sleepForTimeInterval:0.2];
        [[NSRunLoop mainRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:0.1f]];
    }
    if (bIdx == 0){//取消;
        diagStat = NO;
    }
    else if (bIdx == 1) {//确定;
        diagStat = YES;
    }
    return diagStat;
}

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex{
    bIdx = buttonIndex;
}
```

也可以根据需求自定义alert显示效果。

## 内存问题

~~前面在把OC和JS关联起来的方法：    `self.context[@"Alex"] = self;`
这句代码就造成了循环引用，导致webView无法被释放。我试过几个方法，并没有效果。由于一些原因，这个问题被搁置了，等解决了再来更新。~~

后来经过测试，发现跟真正的内存泄漏还是有些差别的，应该是内存释放不及时，在UIWebview所在控制器pop之后，控制器并没有立即释放，再次push进入页面时，上一个控制器才释放。

而实际导致这个问题出现的原因处在获取JS执行环境的方法：
```
self.context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
```
通过 UIWebView 来获取 JSContext ，会出现控制器延迟释放的情况。这个问题目前还有待解决。
