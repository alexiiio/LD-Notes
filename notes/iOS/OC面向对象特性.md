# 面向对象与面向过程

**面向过程**：（procedure oriented programming 即：POP）
是一种是事件为中心的编程思想。通过分析解决问题所需的步骤，用函数把这些步骤实现，并按顺序调用。

- **特性**：模块化、流程化
- **优点**：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源;比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开 发，性能是最重要的因素。
- **缺点**：没有面向对象易维护、易复用、易扩展

**面向对象**：（object oriented programming 即：OOP）
**是按人们认识客观世界的系统思维方式，采用基于对象（实体）的概念建立模型**，模拟客观世界分析、设计、实现软件的办法。通过面向对象的理念使计算机软件系统能与现实世界中的系统一一对应。

**面向对象是将现实世界中的事物抽象成对象，现实世界中的关系抽象成类、继承**，帮助人们实现对现实世界的抽象与数字建模。**面向对象使用更利于人理解的方式，对复杂的系统进行分析、设计与编程**。此外，面向对象能有效的提高编程的效率，通过封装技术、消息机制可以快速的搭建出一个系统。

- 面向对象的主要**内容**：对象、类、方法。
- 面向对象的三大**特性**：继承、封装、多态（还有一个抽象）。
- **优点**：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统更加灵活、更加易于维护
- **缺点**：性能比面向过程低

# 类和对象

**对象**：对象是人们要进行研究的任何事物，从最简单的整数到复杂的飞机等均可看作对象，它不仅能表示具体的事物，还能表示抽象的规则、计划或事件。

对象拥有**状态**和**行为**。

**类**：具有相同特性（数据元素）和行为（功能）的对象的抽象就是类。因此，对象的抽象是类，类的具体化就是对象，也可以说类的实例是对象，类就是一种用户定义的数据类型。

- 类具有属性，它是对象的状态的抽象，用数据结构来描述类的属性。     
- 类具有方法，它是对象的行为的抽象，用方法名和方法实现方式来描述。 
- 类也是一种对象。
# 面向对象三大特性

### 封装
**封装**：指能够把一个实体的信息、功能、响应都装入一个单独的对象中的特性。

1. 封装允许类的客户不必关心类的工作机理就可以使用它；
2. 所有对数据的访问和操作都必须通过特定的方法，否则便无法使用，从而达到数据隐藏的目的。


### 继承
继承：子类自动共享父类非私有数据结构和方法的机制，是类之间的一种关系。

多继承：即一个子类可以有多个父类,它继承了多个父类的特性。OC不支持多继承，可以用协议实现类似功能。

### 多态
**多态**：允许父类指针指向子类对象，不同对象以自己的方式响应相同的消息的能力叫做多态。

多态性允许每个对象以适合自身的方式去响应共同的消息。多态性增强了软件的灵活性和重用性。

实现多态的二种方式：
1. 覆盖：（又叫重写）子类覆盖父类的方法，同样的方法会在父类和子类中有着不同的表现形式。是一种运行时的多态；
2. 重载：重载是指同一个类中有很多个同名方法，但这些方法的参数列表是不相同的，因此编译时就可以确定调用哪个方法，这是一种编译时的多态。（重载是通过不同的方法参数来区分的）
### 抽象性
**抽象性**：是指将具有一致的数据结构（属性）和行为（操作）的对象抽象成类。一个类就是这样一种抽象，它反映了与应用有关的重要性质，而忽略其他一些无关内容。任何类的划分都是主观的，但必须与具体的应用有关。

### 总结
> 封装可隐藏实现细节，使代码模块化；继承可扩展已存在的代码模块（类）；多态是为了实现另一个目的（接口重用）；它们最终需要的结果（代码重用）。多态的作用，就是为了类在继承和派生的时候，保证使用“家谱”中任一类的实例的某一属性时的正确调用。

