# FlyWeight

[String](../../../java/grammar/data-types.md#string) 和 [包装类](../../../java/grammar/data-types.md#bao-zhuang-lei)使用了**享元模式**（蝇量模式）。

前提：享元对象**不可变**。

**与对象池、连接池、线程池的区别**

* 以 C++ 为例，预先申请一片内存作为对象池，每次创建对象时，从对象池取出一个对象使用，使用完后，再放回对象池供后续使用。
* 池化技术的复用是为了节约时间，享元模式的复用是为了节约内存。
* 任意时刻，池化技术中的对象只能被一处使用，使用完后放回，是重复使用；而享元模式同一时刻可以被多处使用，属于共享使用。

