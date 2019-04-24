

# property的关键字有哪些？
## atomic和nonatomic
`atomic`原子性操作，是线程安全的。修饰的对象会保证setter和getter的完整性，任何线程访问它都可以得到一个初始化后的对象。因为要保证操作完成，所以速度比较慢。atomic比nonatomic安全，但也不是绝对的安全。atomic保证同一时间只有一个线程能够写入(但是同一个时间多个线程都可以取值)。例如，当多个线程同时set和get时，就会导致获得的对象值不一致。想要线程绝对安全，就要用@synchronized。  

`nonatomic`非原子性操作，是线程不安全。修饰的对象不保证setter和getter的完整性，所以，当多个线程访问它时，它可能会返回未初始化的对象。  
但是一般都用`nonatomic`，因为atomic线程安全开销大，影响性能。即便要保证线程安全，我们也可以通过自己的代码控制，更何况一般操作都是在主线程。

### 线程安全

所谓一个数据的线程安全，简单点来说就是这块数据即使有多个线程同时读写，也不会出现数据的错乱，内存的最后状态总是可以预见的，如果这块内存的数据被一个多线程读写之后，出现的结果是不可预见的，那么就可以说这块内存是“线程不安全的”。   
其实这个状态很容易理解，同一个箱子，有的人在里面放球，有的人从里面拿，如果没有一个有规则的顺序，都乱哄哄的一起进行，那么最后箱子里几个球只能靠猜了。


## strong
> strong表示指向并拥有该对象。其修饰的对象引用计数会增加1。该对象只要引用计数不为0就不会被销毁。当然，强行将其设为nil也可以销毁它。
## weak
> 表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无须手动设置，该对象会自行在内存中销毁。
## assign
> 主要用于修饰基本数据类型，如NSInteger和CGFloat，这些数值主要存在于栈中。

## copy
> 建立并指向一个索引计数为1的对象，然后释放旧对象。会在内存里拷贝一份对象，两个指针指向不同的内存地址。**copy**一般用在修饰有对应可变类型的不可变对象上，如NSString，NSArray和NSDictionary。   


## readwrite
可读写的
## readonly
只读，不可写入。    



---

# weak和assign的区别
> **assign**一般用来修饰基本数据类型，**weak**一般用来修饰对象。原因是**assign**修饰的对象被释放后，指针的地址依然存在，造成“野指针”，在堆上容易造成崩溃。而栈上的内存系统会自动处理，不会造成“野指针”。而[使用**weak**修饰的对象被释放后会置为nil](http://note.youdao.com/noteshare?id=176a9f47a926bbb0a41f3fa09d5f2fd1)。

## 什么情况使用 weak 关键字？

1. 在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
2. 自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong

# copy和strong的区别
**1. 修饰不可变类型（NSString、NSArray、NSDictionary）时**
```
@property(nonatomic,copy)NSString *name1;
@property(nonatomic,strong)NSString *name2;
```

- 当赋值对象是不可变类型时，**copy**和**strong**效果相同。不管是Strong还是Copy属性的对象，都是指向原对象，Copy操作只是做了次浅拷贝。（通过打印赋值前后对象的地址来验证，发现地址未变化）     

- 当赋值对象是是可变类型(NSMutableString、NSMutableArray)时，strong属性只是增加了原字符串的引用计数，属性的类型变成可变类型。而Copy属性则是对原字符串做了次深拷贝，产生一个新的对象，且这个Copy属性对象的类型始终是不可变类型。   
我们验证一下，当赋值对象的值变化时，看属性的值是否发生变化。
```
    NSMutableString *lucky = [NSMutableString stringWithFormat:@"lucky"];
    self.name1 = lucky;
    self.name2 = lucky;
    [lucky setString:@"tom"];
    NSLog(@"name1：%@ 地址：%p,  name2：%@ 地址：%p  lucky地址：%p",self.name1,self.name1,self.name2,self.name2,lucky);
```
打印结果：
```
name1：lucky 地址：0x8d4d499880c91e62,  name2：tom 地址：0x6000007ee520  lucky地址：0x6000007ee520
```
copy修饰的属性内容没有被改变，strong修饰的属性内容改变，指针跟赋值可变对象相同。   
再来看一下，strong修饰的属性已经变成了NSMutableString类型！而这又是为什么呢？

> 因为NSMutableString是NSString的子类，所以NSMutableString的实例可以给NSString的变量赋值，即**多态**的特性（父类指针指向子类对象），属性就由NSString变成了NSMutableString类型。这在某些情况下是错的，这个时候应该使用 copy关键字来将 NSMutableString 的实例字符串拷贝一份到 NSString实例里面。

> 在声明NSString属性时，到底是选择strong还是copy，可以根据实际情况来定。不过，一般我们将对象声明为NSString时，都不希望它改变，所以大多数情况下，我们建议用copy，以免因可变字符串的修改导致的一些非预期问题。

**2. 修饰可变类型（NSMutableString、NSMutableArray、NSMutableDictionary）时**
```
@property(nonatomic,strong)NSMutableArray *array1;
@property(nonatomic,copy)NSMutableArray *array2;
```
- copy会在赋值时，把NSMutableArray对象拷贝成NSArray对象，在给对象插入数据时会直接崩溃。
- 而strong在赋值时，只是指向NSMutableArray对象，引用计数加1，使用时一切正常。

代码验证一下：
```
    self.array1 = [NSMutableArray arrayWithCapacity:10];
    [self.array1 addObject:@"222"];
```
strong修饰的NSMutableArray运行正常。
```
    self.array2 = [NSMutableArray arrayWithCapacity:10];
    [self.array2 addObject:@"222"];
```
copy修饰的NSMutableArray在addObject时崩溃，报错：
```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException',
reason: '-[__NSArray0 addObject:]: unrecognized selector sent to instance 0x6000012ac0a0'
```


所以，结论是：**修饰不可变类型用copy，修饰可变类型用strong**


