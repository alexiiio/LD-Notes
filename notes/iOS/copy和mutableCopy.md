
**copy**：指针拷贝，又叫浅拷贝。   
**mutableCopy**：内容拷贝，又叫深拷贝。

几种情况：

1. 对集合对象和非集合类对象的 copy 与 mutableCopy 操作；
2. 对可变类对象和不可变对象的 copy 与 mutableCopy 操作

**对非集合类对象的copy操作**：

在非集合类对象中：对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：

- [immutableObject copy] // 浅复制
- [immutableObject mutableCopy] //深复制
- [mutableObject copy] //深复制
- [mutableObject mutableCopy] //深复制

**集合类对象的copy与mutableCopy**：

- [immutableObject copy] // 浅复制
- [immutableObject mutableCopy] //单层深复制
- [mutableObject copy] //单层深复制
- [mutableObject mutableCopy] //单层深复制

总结：
- 不可变对象的`copy`操作都是浅拷贝，可变对象的`copy`操作都是深拷贝；
- `mutableCopy`操作无论是可变还是不可变对象都是深拷贝，但对集合对象只是单层深拷贝。
- 无论原始值可变与否，`copy` 操作返回值都是不可变对象，`mutableCopy` 返回值都是可变对象。
