节选自：http://blog.cnbang.net/tech/3080/

一个 APP 有多个模块，模块之间会通信，互相调用，每个模块都离不开其他模块，互相依赖，耦合性太高。

组件化就是模块间解耦。

这时需要一个Mediator中间件来解耦。一个组件需要调用另一个组件的方法时，不直接调用，而是通过Mediator来操作。

但是Mediator和组件之间依然互相依赖。

为了让Mediator解除对各个组件的依赖，同时又能调到各个组件暴露出来的方法，需要用到runtime反射调用。

类似这样：
```
//Mediator.m
@implementation Mediator
+ (UIViewController *)BookDetailComponent_viewController:(NSString *)bookId {
 Class cls = NSClassFromString(@"BookDetailComponent");
 return [cls performSelector:NSSelectorFromString(@"detailViewController:") withObject:@{@"bookId":bookId}];
}
@end
```

使用 Mediator 加runtime比只使用runtime的好处是：

- 有代码提示，调用者写起来方便。
- 参数类型和个数无限制，由 Mediator 去转就行了，组件提供的还是一个 NSDictionary 参数的接口，但在Mediator 里可以提供任意类型和个数的参数，像上面的例子显式要求参数 NSString *bookId 和 NSInteger type。
- Mediator可以做统一处理，调用某个组件方法时如果某个组件不存在，可以做相应操作，让调用者与组件间没有耦合。

接下来还有两个优化的点：

1. Mediator 每一个方法里都要写 runtime 方法，格式是确定的，这是可以抽取出来的。
2. 每个组件对外方法都要在 Mediator 写一遍，组件一多 Mediator 类的长度是恐怖的。

使用target-action来解决第一点，target就是class名称，action就是selector名称，通过一些规则简化动态调用。Category 对应第二点，每个组件写一个 Mediator 的 Category，让 Mediator 不至于太长。

总结起来就是，组件通过中间件通信，中间件通过 runtime 接口解耦，通过 target-action 简化写法，通过 category 感官上分离组件接口代码。

