# Collection

Java 集合框架和分为 Collection 和 [Map](collection.md#map) 两大类。其中 Collection 又由 [List](collection.md#list-gai-shu)、[Set](collection.md#set)、[Queue](collection.md#queue) 三类组成。

本节主要讲线程不安全的容器，线程安全的容器见 [Thread-safe Collection](../concurrency/thread-safe-collection.md)。

![Collection &#x7C7B;&#x7EE7;&#x627F;&#x5173;&#x7CFB;](../../.gitbook/assets/image%20%2894%29.png)

## List 概述

![](../../.gitbook/assets/image%20%285%29.png)

存取有序，有索引，可以根据索引来进行取值，元素可以重复。主要实现有 [ArrayList](collection.md#arraylist)、[LinkedList](collection.md#linkedlist)、Vector。

其中 Vector 已经过时，被ArrayList取代了。线程安全，通过在每个方法上加`synchronized`实现。

## ArrayList

底层是使用数组实现，实现了自动扩容数组大小。实现了如下接口：

* List：列表。
* Cloneable：可以克隆。
* Serializable：可以实现序列化。
* RandomAccess：是一个标志接口，表示可以快速随机访问。

## LinkedList

是基于链表结构实现的，所以查询速度慢，增删速度快，提供了特殊的方法，对头尾的元素操作（进行增删查）。

## Set

元素不可以重复。

### HashSet

**存取无序**，基于**哈希表**实现。

通过`hashCode`和`equals`方法来共同保证元素唯一的。底层也维护了一个数组。

根据存储的元素计算出`hashCode`值，然后根据计算得出的`hashCode`值和数组的长度进行计算出存储的下标；如果下标的位置无元素，那么直接存储；如果有元素，那么使用要存入的元素和该元素进行`equals`方法，如果结果为真，则已经有相同的元素了，所以直接不存；如果结果假，那么进行存储，以**链表**的形式存储。

### LinkedHashSet

基于**链表**和**哈希表**共同实现的，所以具有**存取有序**，元素唯一。

### TreeSet

**存取无序**，可以进行排序（排序是在添加的时候进行排序）。基于**二叉树**的数据结构。

TreeSet 保证元素的唯一性是有两种方式：

1. 存取的元素必须实现`Comparable`接口。
2. 在创建`TreeSet`的时候向构造器中传入一个`Comparator`对象，`Comparator`对象用于比较存取元素的类型。

## Queue

队列通常是指“先进先出”（FIFO）的容器。新元素插入（`offer`）到队列的尾部，访问元素（`poll`）操作会返回队列头部的元素。

通常，队列不允许随机访问队列中的元素。

## Map

Map保存的是键值对，键要求保持唯一性，值可以重复。

![Map &#x7C7B;&#x7EE7;&#x627F;&#x5173;&#x7CFB;](../../.gitbook/assets/image%20%289%29.png)

### HashMap

基于**哈希表**结构实现的，所以存储自定义对象作为键时，必须重写`hasCode`和`equals`方法。**存取无序**的。

HashMap 在多线程使用时可能会出现 CPU 100% 的情况，原因是 JDK1.7 出现 Hash 冲突时，采用的是头插法，会改变节点顺序，会造成循环链表，发生死循环。详细分析见[这篇文章](https://www.jianshu.com/p/1e9cf0ac07f4)。

### LinkedHashMap

基于**链表**和**哈希表**结构的，所以具有**存取有序**。

### TreeMap

TreeMap 底层使用的**二叉树**，其中存放进去的所有数据都需要排序，所以要求对象具备比较功能。与 TreeSet 类似，存取的元素需要实现`Comparable`接口，或者给 TreeMap 集合传递一个`Comparator`接口对象。

### HashTable

HashTable 是一个**线程安全**的 Map 实现，其它与 HashMap（非线程安全）类似。

HashTable 不允许`Null`作为 Key 和 Value，但是 HashMap 可以。

