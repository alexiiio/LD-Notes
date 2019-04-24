


在我们给一个类声明`property`的时候，系统会自动生成带`_`的成员变量和该变量的`setter`和`getter`方法。也就是说，属性相当于一个成员变量加getter和setter方法。

```
@interface UIButton (Name)
@property(nonatomic,copy)NSString *name;
@end
```
**而在类目中声明的属性，并没有添加带`_`的成员变量，也没有实现`setter`和`getter`方法，只是在属性列表里添加了"name"属性。**

即使我们去实现`setter`和`getter`方法，也无法在`setter`和`getter`方法里访问带`_`的成员变量，因为系统就没有添加带`_`开头的成员变量，使用`self.name`也是不对的。 

![image](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/runtimeCategoryProperty.png)

此时要想实现`setter`和`getter`方法，可以使用`runtime`的关联对象。代码如下：

```
#import "UIButton+Name.h"
#import <objc/runtime.h>
@implementation UIButton (Name)
static const void *kUIButtonName = @"kUIButtonName";
- (void)setName:(NSString *)name {
    objc_setAssociatedObject(self, kUIButtonName, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
- (NSString *)name {
    NSString *name = objc_getAssociatedObject(self, kUIButtonName);
    return name;
}
@end
```

**通过 `runtime` 中 `objc_getAssociatedObject` 和 `objc_setAssociatedObject` 方法来访问和生成关联对象。通过这种方法来模拟生成属性。**      
在UIViewController里验证一下：
```
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [self.view addSubview:button];
    button.name = @"sarah";
    NSLog(@"%@",button.name);
```
控制台正常打印出：`sarah`

**在分类里使用`@property`声明属性，又实现了`setter`和`getter`方法后，在这个类外部可以正常通过点语法和KVC给该属性赋值和取值。可以认为给这个类添加上了属性。**

另外，通过打印`UIButton`的属性列表和实例变量列表，发现属性列表里有`name`，实例变量列表里没有。这与我们了解的知识相符合。在获取时应注意。