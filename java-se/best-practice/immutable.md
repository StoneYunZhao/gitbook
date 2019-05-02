# Immutable

不可变类有很多好处，比如线程安全等。创建不可变类一般需要做如下处理：

* 将 class 声明为 final。
* 所有成员变量声明为 private final，并不要暴露 setter 方法。
* 对于 getter 或其它可能会返回内部状态的方法，使用 copy-on-write 的方式。
* 构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值。

