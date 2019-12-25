# Stream

Stream 相当于高级版的 Iterator，可以通过 Lambda 表达式对集合进行各种非常便利、高效的聚合操作（Aggregate Operation），大批量数据操作（Bulk Data Operation）。

在 Java 8 中，Collection 接口新增了两个 Stream 相关的接口：

```java
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}

default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```

Stream 的操作可以按照如下分类：

* 中间操作（Intermediate operations）： 对操作进行记录，只返回一个流，不会进行计算。
  * 无状态（Stateless）：元素处理不受之前元素的影响。
  * 有状态（Stateful）：只有拿到所有元素才能继续。
* 终结操作（Terminal operations）：触发计算。
  * 短路（Short-circuiting）：遇到某些符合条件的元素就可以得到最终结果。
  * 非短路（Unshort-circuiting）：必须处理完所有元素才能得到最终结果。

![](../../.gitbook/assets/image%20%2823%29.png)

![](../../.gitbook/assets/image%20%28159%29.png)

* BaseStream：定义了流的基本方法，如 isParallel。
* Stream：定义了流的常用操作，如 map、filter。
* ReferencePipeline：定义了 Head、StatelessOp、StatefulOp 内部类来组装各种操作流。
  * Head：定义数据源操作，比如调用 stream\(\) 方法。
* Sink：定义每个 Stream 操作之间关系的协议。

