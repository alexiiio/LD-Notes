
KVC全称key-value-coding,即键值编码。

> key-value coding is a mechanism for indirectly accessing an object’s attributes and relationships using string identifiers.      
键值编码是一种使用字符串标识符间接访问对象属性和关系的机制。

KVC是一种由非正式协议（NSKeyValueCoding）启用的机制，来间接访问对象的属性。

# 实现原理

创建一个NSObject的类目`NSKeyValueCoding`，声明了相当多的取值赋值方法。而实现这些方法中最关键的是寻找key的逻辑。

在我们使用`- (void)setValue:(nullable id)value forKey:(NSString *)key`方法给属性赋值时，KVC有一套自己的逻辑来寻找key。简单来说就是：
1. 先去找实例的`setter`方法`setKey:`或`_setKey`，找到就调用方法赋值；
2. 如果找不到`setter`方法，并且对象的类方法`+ (BOOL)accessInstanceVariablesDirectly`返回YES（默认YES），就按照`_key`、`_iskey`、`key`、`iskey`的顺序搜索实例变量并进行赋值操作；
3. 如果方法和实例变量都找不到，系统将会执行该对象的`setValue:forUndefinedKey:`方法，默认是抛出异常，NSObject的子类可以重写该方法做特定处理。

如果重写`+ (BOOL)accessInstanceVariablesDirectly`返回NO，就不会有第二步查找实例变量，直接进入第三步。

而在使用`valueForKey:`方法时，查找key的步骤跟set时相似，但更加复杂，相当于在上面第一步和第二步之间加了两种查找的方法。并且在找到key之后，会把取到的属性值包装成对象类型返回。


`valueForKey:`会自动将值类型封装成对象，但`setValue:forKey:`却不会。需要手动将值类型转换成NSNumber或者NSValue类型，因为传递进去和取出来的都是id类型，所以需要开发者自己担保类型的正确性，运行时OC在发送消息的会检查类型，如果错误会直接抛出异常。

# KVC处理异常

KVC常见的异常就是传递了不存在的key，或传递了nil值。

当传递了不存在的key，系统会调用
```
- (id)valueForUndefinedKey:(NSString *)key {

    return nil;
}

- (void)setValue:(id)value forUndefinedKey:(NSString *)key {

}
```
如果传了nil值，系统会调用
```
- (void)setNilValueForKey:(NSString *)key {

}
```
可以看情况重写以上方法处理异常情况。


# 常见用法
KVC的常见用法：
1. 对私有属性赋值，包括修改一些系统控件的内部属性
2. 字典和模型转换


KVO的的实现基于KVC机制，[关于KVO请看](http://note.youdao.com/noteshare?id=d8b798e97282ce967ee079a736d8fdc1)