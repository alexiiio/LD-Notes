# load方法
+load方法是在把类或类目添加到运行时的时候调用的。此时，类的相关加载方法都还没执行。

执行顺序：
- 父类的load -> 子类的load；
- 类的load -> 类目的load；
- 类目之间的load加载顺序跟编译顺序一致，跟继承关系无关。

> load方法的调用并不是objc_msgSend机制，它是直接找到类的load方法的地址，然后调用类的load方法，然后再找到分类的load方法的地址，再去调用它。

# initialize方法

initialize在类第一次接收到消息时调用，也就是objc_msgSend()时。

initialize方法是通过消息机制调用的。它只是调用时机特殊，其他跟普通的OC方法一样。

调用过程：

- 如果本类的+initialize方法实现了就直接调用，如果没有实现，会调用父类的+initialize(所以父类的+initialize方法可能会被调用多次)
- 如果分类实现了+initialize，会覆盖类本身的+initialize调用。

----
**所以+load方法是在+initialize之前调用的**。事实上，+load方法在程序的main函数执行前就已经调用了。详见[App的启动过程](http://note.youdao.com/noteshare?id=e796c4237ba55a8e3c4c0e2512b9423c)