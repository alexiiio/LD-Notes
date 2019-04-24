
BAD_ACCESS 报错属于内存访问错误，会导致程序崩溃。

# BAD_ACCESS 在什么情况下出现？

1. 访问了野指针。
2. 死循环的时候。

# 如何调试BAD_ACCESS错误

1. 重写object的respondsToSelector方法，显示出EXEC_BAD_ACCESS前访问的最后一个object。
2. 通过 Zombie 。

    （1）打开Xcode->Product->Scheme->Edit Scheme->Run->Eable Zombie Objects    
    （2）使用Instrument->Zombie
3. 设置全局断点快速定位问题代码所在行

全局断点添加方法如下所示：

![breakpoint1](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/breakpoint1.png)
![breakpoint2](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/breakpoint2.png)