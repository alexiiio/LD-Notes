
![UIView&CALayer](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/UIView&CALayer.png)


View初始化的时候会自动创建并持有Layer，而Layer会自动指定代理是View，View相当于Layer的管理者。Layer负责图形的绘制，View 中大部分显示属性实际是从 Layer 映射而来；View负责显示内容的管理和事件的响应。

Layer 的 delegate 在这里是 View，当其属性改变、动画产生时，View 能够得到通知。
 
 UIView主要是对显示内容的管理，而CALayer主要是对显示的绘制。View中对于图形的渲染，和一些动画操作都是Layer完成的，View负责呈现Layer的工作结果和处理交互响应事件。

 
 UIView 继承自 UIResponder；CALayer 继承自 NSObject。所以UIView可以响应事件，而CALyer则不能响应事件。
 
 UIView 和 CALayer 不是线程安全的，并且只能在主线程创建、访问和销毁。

