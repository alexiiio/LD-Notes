**OC中既有私有方法，也有私有变量。但是没有绝对的私有方法和私有变量。**

**私有是指只能够在本类内部使用或访问，但是不能在类的外部被访问**。

**私有方法** - 如果不在`.h`文件中声明，只在`.m`文件中实现的方法，或在类的延展`Extension`里声明的方法，就是私有方法。但是由于OC的动态消息传递机制，所以OC中不存在真正意义上的私有方法。

```
//通过消息传递机制，即使是声明的私有方法，也可以在类的外部被调用
id objc_msgSend(id self, SEL op, ...)
```

**延展（Extension）是一种匿名类目，用来给类添加私有方法或私有变量。**

**私有变量** - 可以通过`@private`或者延展`Extension`来声明私有变量。但是可以通过`KVC`或者`runtime`来访问私有变量，所以也没有真正意义上的私有变量。

```
/*
通过 runtime 访问私有变量。
*/
@implementation MMFather {
    NSString *_name;
 }
 
 // main.m
 {
    MMSon *son = [[MMSon alloc] initWithName:@"小白"];
    Ivar nameIvar = class_getInstanceVariable([son class], "_name");
    NSString *name = object_getIvar(son, nameIvar);
    NSLog(@"son name: %@", name);
 }
```