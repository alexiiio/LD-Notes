
参考[美团技术团队：深入理解Objective-C：Category](https://tech.meituan.com/DiveIntoCategory.html)

category是可以添加属性（property）的，只是添加的属性不会生成实例变量（Ivar）以及setter和getter方法。

# extension和category

extension在编译期决议，它就是类的一部分，在编译期和头文件里的@interface以及实现文件里的@implement一起形成一个完整的类，它伴随类的产生而产生，亦随之一起消亡。extension一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加extension，所以你无法为系统的类比如NSString添加extension。

但是category则完全不一样，它是在运行期决议的，category的主要作用是为已经存在的类添加方法。

使用category的场景：
- 可以把类的实现分开在几个不同的文件里面。
>这样做有几个显而易见的好处：      
a)可以减少单个文件的体积            
b)可以把不同的功能组织到不同的category里  
c)可以由多个开发者共同完成一个类          
d)可以按需加载想要的category 等等。
- 声明私有方法
- 模拟多继承
- 把framework的私有方法公开

分类没有自己的isa指针。

所有的OC类和对象，在runtime层都是用struct表示的，category也不例外，category用结构体category_t（在objc-runtime-new.h中可以找到此定义），它包含了：      
1)、类的名字（name）            
2)、类（cls）         
3)、category中所有给类添加的实例方法的列表（instanceMethods）         
4)、category中所有添加的类方法的列表（classMethods）         
5)、category实现的所有协议的列表（protocols）         
6)、category中添加的所有属性（instanceProperties）         
```
typedef struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
} category_t;
```

# 为什么category里不能添加实例变量？

extension可以添加实例变量，而category是无法添加实例变量的。

category的结构体`category_t`中不包含类的实例变量，系统不允许category添加实例变量。

**因为category是在运行期加载的，在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局。**

在category中添加实例变量，编译器会报错：

![categoryaddivarfail](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/categoryaddivarfail.png)

在runtime中，确实有一个`class_addIvar()`函数用于给类添加成员变量。但是苹果的注释提到：
> This function may only be called after objc_allocateClassPair and before objc_registerClassPair. Adding an instance variable to an existing class is not supported.

此函数只能在objc_allocateClassPair之后和objc_registerClassPair之前调用。不支持向现有类中添加实例变量。意思是这个函数只能在“构建一个类的过程中”调用。使用分类在运行时添加实例变量，是苹果基于安全考虑不允许的。


# category添加方法

>1)、category的方法没有“完全替换掉”原来类已经有的方法，也就是说如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA         

>2)、category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会罢休\^_^，殊不知后面可能还有一样名字的方法。

3)、顺着方法列表找到最后一个对应名字的方法，就可以调用原来类的方法

category的加载顺序跟编译顺序一致，所以不同category里的方法（如+load）加载顺序也跟编译顺序一致。

# [使用runtime为Category添加属性](http://note.youdao.com/noteshare?id=a0ec4c0a111be45e69b87a2e77af3a4e)