# Collection

Java 集合框架和分为 Collection 和 [Map](collection.md#map) 两大类。其中 Collection 又由 [List](collection.md#list-gai-shu)、[Set](collection.md#set)、[Queue](collection.md#queue) 三类组成。

本节主要讲线程不安全的容器，线程安全的容器见 [Thread-safe Collection](../concurrency/thread-safe-collection.md)。

![Collection &#x7C7B;&#x7EE7;&#x627F;&#x5173;&#x7CFB;](../../.gitbook/assets/image%20%2896%29.png)

## List 概述

![](../../.gitbook/assets/image%20%285%29.png)

存取有序，有索引，可以根据索引来进行取值，元素可以重复。主要实现有 [ArrayList](collection.md#arraylist)、[LinkedList](collection.md#linkedlist)、Vector。

其中 Vector 已经过时，被ArrayList取代了。线程安全，通过在每个方法上加`synchronized`实现。

## ArrayList（源码分析）

底层是使用数组实现，实现了自动扩容数组大小。实现了如下接口：

* List：列表。
* Cloneable：可以克隆。
* Serializable：可以实现序列化。
* RandomAccess：是一个标志接口，表示可以快速随机访问。

下面是对 ArrayList 的源码分析，Java 1.11。

### 属性

```java
private static final int DEFAULT_CAPACITY = 10;
transient Object[] elementData; // non-private to simplify nested class access
private int size;
```

可以看到数据元素 elementData 被 transient 修饰了，但是 ArrayList 又实现了 Serializable 接口，这样不是矛盾的吗？

原因：ArrayList 可以自动扩容，所以不是数组的每个元素都存储了数据，所以不能序列化整个数组。ArrayList 有两个方法 writeObject、readObject，ObjectInputStream/ObjectOutputStream 在序列化对象时会通过**反射**调用这两个方法。

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException { }
    
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException { }
```

### 构造函数

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(Collection<? extends E> c) { ... }
```

ArrayList 新增元素时，若超过数组大小，数组会扩容，导致内存复制，所以最好指定合理的初始化容量。

### 新增元素

```java
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);
    modCount++;
    final int s;
    Object[] elementData;
    if ((s = size) == (elementData = this.elementData).length)
        elementData = grow();
    System.arraycopy(elementData, index,
                     elementData, index + 1,
                     s - index);
    elementData[index] = element;
    size = s + 1;
}
```

有两个新增元素的方法，一个是将元素增加到末尾，一个是增加到任意位置。都会先确定容量大小，若大小不够，则先扩容 1.5 倍。

```java
private Object[] grow() {
    return grow(size + 1);
}

private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}

private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
```

添加元素到指定位置的方法，会使改位置之后的所有元素都移动。

若 ArrayList 在初始化时指定了容量，并且每次添加都在末尾，那么新增元素的速度比 LinkedList 高。

### 删除元素

```java
public E remove(int index) {
    Objects.checkIndex(index, size);
    final Object[] es = elementData;
    @SuppressWarnings("unchecked") E oldValue = (E) es[index];
    fastRemove(es, index);
    return oldValue;
}

private void fastRemove(Object[] es, int i) {
    modCount++;
    final int newSize;
    if ((newSize = size - 1) > i)
        System.arraycopy(es, i + 1, es, i, newSize - i);
    es[size = newSize] = null;
}
```

同样每次删除元素都会挪动一些元素，删除位置越靠前，挪动的范围越大。

### 获取元素

```java
public E get(int index) {
    Objects.checkIndex(index, size);
    return elementData(index);
}

@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```

由于是基于数组实现，所以获取元素很快。

## LinkedList（源码分析）

是基于双向链表结构实现的，所以查询速度慢，增删速度快，提供了特殊的方法，对头尾的元素操作（进行增删查）。定义了 Node 结构：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

在 Java 1.7 之前，LinkedList 只包含了一个Entry 结构的 header 属性；1.7 之后做了优化，Entry 结构换成了 Node，header 属性变成了 first 和 last 两个属性。

LinkedList 实现了这些接口：Deque、Cloneable、Serializable，没有实现 RandomAccess 接口。

### 属性

```java
transient int size = 0;
transient Node<E> first;
transient Node<E> last;
```

与 ArrayList 一样，属性也被 transient 修饰，也有 readObject、writeObject 方法。

### 新增元素

将元素添加到队尾：

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

将元素添加到任意位置，相对于 ArrayList，不需要移动元素：

```java
public void add(int index, E element) {
    checkPositionIndex(index);
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

Node<E> node(int index) {
    // assert isElementIndex(index);
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

### 删除元素

注意 node 方法，作用是找到元素位置，若位置处于前半段，则从前往后找，若处于后半段，则从后往前找。

所以删除靠前或靠后的元素效率很高，删除中间部分的元素效率相对较低。

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}

E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

### 获取元素

同样也是调用 node 方法，区分前半段和后半段。

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

所以建议遍历 LinkedList 时，使用 Iterator 的方式。

## Iterator（源码分析）

如下 remove1 方法是正确的，remove2 是不正确的，会抛出 ConcurrentModificationException。

因为虽然 foreach 方法是语法糖，会转换成 Iterator 方法，但是第 15 行还是调用的 List 的 remove 方法，而不是 Iterator 的 remove 方法。

```java
public static void remove1(ArrayList<String> list) {
        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String str = it.next();
            if (str.equals("b")) {
                it.remove();
            }
        }
    }

public static void remove2(ArrayList<String> list) {
        for (String s : list) {
            if (s.equals("b")) {
                list.remove(s);
            }
        }
    }
```

Iterator 有属性 expectedModCount，当不匹配时会抛出异常。List 的 remove 方法不会修改该值，造成不匹配，所以抛出异常。

```java
public Iterator<E> iterator() {
    return new Itr();
}

private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;
    
    ...
}
```

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

