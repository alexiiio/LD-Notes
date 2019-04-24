
- **@synthesize**:为属性添加一个实例变量名，同时会为该属性生成 setter/getter 方法。

- **@dynamic**：告诉编译器，属性的 setter 与 getter 方法由用户自己实现，不自动生成。如果自己没有实现getter和setter方法、编译时没问题，运行时执行对应的方法（动态绑定）时会导致程序崩溃。使用`@dynamic`一般在程序运行的时候为此类属性生成getter和Setter方法，通常用不到。

在.h文件声明几个属性：
```
@property(nonatomic,copy)NSString *name;
@property(nonatomic,assign)NSInteger age;
@property(nonatomic,assign)BOOL isAdult;
```

在.m文件里：
```
@synthesize name = _name;
@dynamic age;
```
`@synthesize name = _name;`就会添加一个`_name`的实例变量，并生成`name`属性的setter和getter方法。如果不使用`_name`，使用其他名字也是可以的。

`@dynamic age;`编译器不会添加实例变量，也不会生成`age`属性的setter和getter方法。`@dynamic`仅仅是告诉编译器这两个方法在运行期会有的，无需产生警告。



- 如果一个属性（`@property`）没有用`@dynamic`或`@synthesize`修饰，那么默认的是`@synthesize var = _var`。
- 如果同时重写了`setter`和`getter`方法，需要在.m文件里使用`@synthesize`，否则编译器会报错。
