# Thread-safe Collection

在 [Collection](../class-libraries/collection.md) 一节中提到，Java 容器有四大类：List、Set、Queue、Map。

## Synchronized Collection

把非线程安全的容器类变成线程安全的容器类最简单的方法是给每个方法添加 synchronized 关键字。例如工具类 Collections 就提供了一套静态方法：

```java
List list = Collections.synchronizedList(new ArrayList());
Set set = Collections.synchronizedSet(new HashSet());
Map map = Collections.synchronizedMap(new HashMap());
```

但是要注意的是，包装后的类虽然每个方法都是线程安全的，但是组合操作或者迭代器操作就不是线程安全的。比如正确的迭代操作需要给链表加锁：

```java
List list = Collections.synchronizedList(new ArrayList()); 
synchronized (list) { 
    Iterator i = list.iterator(); 
    while (i.hasNext()) 
        foo(i.next());
}
```

基于synchronized 的容器叫做**同步容器**，Java 还提供以下同步容器：

* Vector
* Stack
* HashTable

## Concurrency Collection

同步容器性能较差，因此 Java 1.5提供了**并发容器**。

![](../../.gitbook/assets/image%20%2881%29.png)

### List

List 里面只有一个实现类就是 `CopyOnWriteArrayList`。

**`CopyOnWrite`**：如果在遍历 Array 的同时，有一个写操作，那么会先将数据复制一份，然后在新的数组里面写入数据，完成之后指向这个新的数组。所以**读写是可以并行**的。

适用场景：

1. 写操作非常少；
2. 容忍短暂的读写不一致。

注意：迭代器是只读的，因为迭代器遍历的仅仅是一个快照。

![](../../.gitbook/assets/image%20%2829%29.png)

### Map

#### ConcurrentHashMap

key 是无序的。

#### ConcurrentSkipListMap

key 是有序的。基于跳表实现，插入、删除、查询的平均时间复杂度为 O\(logn\)。

![](../../.gitbook/assets/image%20%2883%29.png)

### Set

* CopyOnWriteArraySet
* ConcurrentSkipListSet

### Queue

#### 阻塞与非阻塞

* **阻塞**：队列已满时，入队阻塞；队列以空时，出队阻塞；用 Blocking 标识。
* **非阻塞**：

#### 单端与双端

* **单端**：只能队尾入队，队首出队；用 Queue 标识。
* **双端**：队首队尾兼可入队出队；用 Deque 标识。

#### 单端阻塞队列

* **`ArrayBlockingQueue`**：数组实现；
* **`LinkedBlockingQueue`**：链表实现；
* **`SynchronousQueue`**：里面没有队列，生产者入队操作必须等待消费者出队操作；
* **`LinkedTransferQueue`**：融合 LinkedBlockingQueue 和 SynchronousQueue 的功能，性能比 LinkedBlockingQueue 好；
* **`PriorityBlockingQueue`**：支持按照优先级出队；
* **`DelayQueue`**：支持延时出队。

#### 双端阻塞队列

`LinkedBlockingDeque`

#### 单端非阻塞队列

`ConcurrentLinkedQueue`

#### 双端非阻塞队列

`ConcurrentLinkedDeque`

{% hint style="warning" %}
注意队列**是否有界**，仅 `ArrayBlockingQueue` 和 `LinkedBlockingQueue` 支持有界。
{% endhint %}

