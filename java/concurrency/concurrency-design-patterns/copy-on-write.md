# Copy-on-Write

[不可变对象](immutable.md)的写操作一般用 Copy-on-Write（写时复制）技术解决，比如 [String](../../grammar/data-types.md#string)。

之前介绍的 [CopyOnWriteArraySet](../thread-safe-collection.md#list) 和 [CopyOnWriteArrayList](../thread-safe-collection.md#set) 的设计思想也是写时复制。

除了 Java，在其它领域也广泛应用了写时复制。比如：

*  Unix 下创建进程的[ API fork\(\)](../../../other/linux/#jin-cheng-guan-li)。
* 文件系统 Btrfs\(B-Tree File System\)、aufs\(advanced multi-layered unification system\)等。
* Docker 容器镜像。
* 分布式源码管理系统 Git。
* 函数式编程领域。

函数式编程的基础是不可变性，所以函数式编程的修改操作都需要 Copy-on-Write。

Copy-on-Write 适用于**读多写少**的场景，这也是为什么 Java 没有 CopyOnWriteLinkedList 实现，因为 LinkedList 适合于读少写多的场景。

