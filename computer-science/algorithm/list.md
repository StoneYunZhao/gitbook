# Linear List

**线性表**（Linear List）就是数据排成一条线，上面的数据只有前后两个方向，**数组**、**链表**、**队列**、**栈**都是线性表结构。二叉树、堆、图是非线性表结构。

## Array

### 概念

数组（Array）是一种线性表结构，它用一组**连续**的内存空间，存储一组具有**相同类型**的数据。

连续的内存和相同类型的数据使得数组有**随机访问**的特性。

**误区**：数组查找时间复杂度为 O\(1\)。其实，排好序的数组查找时间复杂度也才 O\(logn\)，正确表述为数组支持随机访问，根据下标访问的时间复杂度为 O\(1\)。

### 插入

将一个数据插入到数组的第 k 个位置，时间复杂度最好 O\(1\)，最坏 O\(n\)，平均 O\(n\)。

**优化**：若数组有序，那没办法，必须挪动；若数组是无序的，则可以直接将插入位置的数据移到最后，时间复杂度降为 O\(1\)。快排就用到了这种思想。

### 删除

删除第 k 个位置的数据，时间复杂度最好 O\(1\)，最坏 O\(n\)，平均 O\(n\)。

**优化**：若要多次删除，可以将多个删除操作一起执行，减少数据搬迁次数。也可以仅标记数据已经删除，并不真正做删除操作，当数据空间用完时，再触发真正的删除操作。JVM 标记-整理的垃圾回收算法也是用此思想。

### 容器与数组

Java 的 ArrayList，C++ STL 的 vector 都是数组的容器类。

**容器类的优势**：

* 将很多数组的操作封装，比如插入、删除数据的数据迁移。
* 支持动态扩容，比如 Java 的 ArrayList 在容量不足时自动扩容为 1.5 倍。注意：就算用 ArrayList，若事先知道数据大小，也需在初始化的时候指定大小，减少内存申请和数据搬迁操作的次数。

**数组的优势**：

* 支持基本类型，如 Java 的 int，long 等，用 ArrayList 需要包装类，有性能消耗。
* 多维数组更加直观，Object\[\]\[\] 就行，而容器需要 ArrayList&lt;ArrayList&lt;Object&gt;&gt;。

**总结**：

* 业务开发，用容器类就行。
* 底层开发，数组优于容器。

### 数组编号从0开始

大多数编程语言，数组从 0 开始编号。理由：

1、数组寻址公式如下，从1开始多了一次CPU减法指令。

```text
# 编号从0开始
a[k]_adress = base_adress + k * type_size
​
# 编号从1开始
a[k]_adress = base_adress + (k -1) * type_size
```

2、历史原因，C 语言设计从 0 开始，之后的语言都效仿 C 语言，减少学习成本。Matlab 从 1 开始，Python 支持负数下标。

### 例题

* [LeetCode 169：找出数组中出现次数超过一半的元素。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/array/LT169.java)
* [LeetCode 42：数组中找到没有出现的最小正整数。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/array/LT42.java)

## List

**数组与链表比较**，从存储结构来说，数组需要连续的内存空间，容易产生内存不足的错误，不能动态扩容，单连续内存的特性可借助 CPU 的缓存机制，访问效率更高；而链表不需要，天然支持动态扩容，但是频繁的插入删除会产生较多的内存碎片。

**链表的种类**：单链表、双向链表、循环链表。

**单链表**：链表通过指针把零散的内存块串联在一起。内存块称为**结点**，结点上有数据和**后继指针** next。第一结点叫**头结点**，记录链表的基地址；最后一个结点叫**尾结点**，指针指向 NULL。

**时间复杂度**：链表插入和删除时间复杂度为 O\(1\)，注意仅是单纯的插入和删除操作，但是一般要找到删除的位置，这个查找过程时间复杂度为 O\(n\)，所以删除给定值的总时间复杂度为 O\(n\)。随机访问第 k 个元素，平均时间复杂度为 O\(n\)。

**循环链表：**尾结点指向头结点。相比于单链表，从链尾到链头比较方便，处理具有环形结构时合适。

**双向链表**：

* 每个结点有后继指针`next`和前驱指针`prev`。比单链表占用更多空间，但支持双向遍历。
* 可以支持`O(1)`时间复杂度找到前驱结点。
* 删除给定指针指向的节点，单链表时间复杂度为`O(n)`，而双向链表为`O(1)`，因为单链表需要遍历找到前驱结点。插入类似。
* 对于**有序链表**，双向链表的查询效率比单链表高很多，因为可以记录前一次查询 p 的位置，在下一次查询与 p 比较，决定向前还是向后查询。
* **`LinkeHashMap`**用到了双向链表。
* 双向链表即是**空间换时间**的例子。

**双向循环链表**：即循环链表与双向链表的组合。

**缓存淘汰策略**：

* 先进先出 FIFO（First in, First out）
* 最少使用 LFU（Least Frequently Used）
* 最近最少使用 LRU（Least Recently Used）

**LRU实现**：维护一个有序单向链表，当有新数据访问时，重头至尾遍历，如果找到，则删除原位置，插入到头部；若没有找到且未满，则插入头部；若没有找到且已满，则删除尾结点，插入头部。时间复杂度为`O(n)`，可用`HashTable`优化，时间复杂度为`O(1)`。

**链表代码技巧**：

* 利用**哨兵**简化难度，针对链表的插入、删除，需要对插入第一个结点和删除最后一个结点做特殊处理。可以增加一个哨兵结点，不存储数据，head指针一直指向这个结点，有哨兵的链表叫**带头链表（有头结点链表）**。
* 重点留意边界条件，若链表为空，若链表只有一个结点，若只有两个结点，在处理头结点和尾结点时，能否正常工作。
* 画图辅助思考。

### 例题

* [LeetCode 206，反转列表](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/list/LT206.java)。
* [LeetCode 24，列表元素两两反转。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/list/LT24.java)
* [LeetCode 25，列表每 k 个元素反转。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/list/LT25.java)
* [LeetCode 141，判断列表是否有环。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/list/LT141.java)
* [LeetCode 142，判断列表是否有环，并返回环的起点位置。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/list/LT142.java)

### 无序链表移除重复项

```java
/**
 * 空间换时间的方式，通过一个 HashSet 存储所有的值
 */
private static void bySet(LNode head) {
    if (head == null) {
        return;
    }
    Set<Integer> values = new HashSet<>();
    LNode pre = head, cur = head.next;
    while (cur != null) {
        if (values.contains(cur.data)) {
            pre.next = cur.next;
        } else {
            values.add(cur.data);
            pre = cur;
        }
        cur = cur.next;
    }
}

/**
 * 通过两层循环实现, 时间复杂度 O(n^2)
 */
private static void byTwoLoop(LNode head) {
    if (head == null) {
        return;
    }
    LNode cur = head.next;
    while (cur != null) {
        Integer data = cur.data;
        int count = 0;
        LNode innerCur = head.next, pre = head;
        while (innerCur != null) {
            if (Objects.equals(innerCur.data, data)) {
                count++;
            }
            if (count > 1) {
                pre.next = innerCur.next;
                count--;
            } else {
                pre = innerCur;
            }
            innerCur = innerCur.next;
        }
        cur = cur.next;
    }
}
```

## Stack

**定义**：栈是一种操作受限的线性表，后进先出，先进后出。

**操作**：入栈 push，出栈 pop。

**分类**：数组实现的栈叫**顺序栈**，链表实现的栈叫**链式栈**。

**空间复杂度**：空间复杂度为 O\(1\)，如存储数据需要大小为 n 的数组，并不是说空间复杂度为 O\(n\)，因为空间复杂度是指除了原本的数据存储空间外，算法运行需要的额外存储空间。

**时间复杂度**：出栈、入栈的时间复杂度都是 O\(1\)。

**支持动态扩容的顺序栈**：出栈时间复杂度 O\(1\)，入栈最好 O\(1\)，最坏 O\(n\)，平均 O\(1\)。但是这种栈并不常见。

**栈的应用**：

* **函数调用栈**，每进入一个函数，会将临时变量作为栈帧入栈，函数执行完后，栈帧出栈。
* **表达式求值**，从左至右遍历表达式，遇到数字压入**操作数栈**；遇到运算符，与**运算符栈**的栈顶比较优先级，若比栈顶高，则压入运算符栈，若比栈顶低或相同，则取出运算符栈的栈顶，取出两个操作数栈，计算结果压入操作数栈，继续比较。
* **括号匹配**，一个栈实现。
* **浏览器前进后退功能**，两个栈实现。

### 例题

* [LeetCode 20，判断字符串括号是否匹配。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/stack/LT20.java)
* [LeetCode 232，用栈实现队列。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/stack/LT232.java)
* [LeetCode 225，用队列实现栈。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/stack/LT225.java)
* [LeetCode 844，判断两个编辑序列得到的字符串是否相等。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/stack/LT844.java)

## Queue

**定义**：操作受限的线性表，先进先出。

**操作**：入队 enqueue，放一个数据到队列尾部；出队 dequeue，从队列头部取一个数据。

**分类**：数组实现的队列叫**顺序队列**，链表实现的队列叫**链式队列**。

**实现**：队列需要两个指针，head 指向对头；tail 指向队尾。

* 数组实现时，`tail = n`时，做一次数据整体搬迁。出队、入队时间复杂度为 O\(1\)。
* 链表实现时，入队`tail -> next = new_node, tail = tail -> next`，出队`head = head -> next`。

**循环队列**：把数组的尾的下一个结点定义为数组的头，就可以避免数据搬迁操作。循环队列的难点在于队空（`heal = tail`）和队满（`(tail+1) % n = head`）的判定条件。循环队列会浪费一个数组的存储空间，可以通过增加一个队列大小 size 来避免这个存储空间的浪费。

**阻塞队列：**在队列为空时，取数据会被阻塞；队列已满时，插入数据会被阻塞。

**并发队列：**线程安全的队列。无锁方式用 CAS 实现，入队前获取 tail 的位置，入队时判断 tail 是否变化，若变化了则本次入队失败；出队时则获取 head 的位置，进行 CAS 判断。

## Priority Queue

正常入，按照优先级出。实现方式：

* [堆](tree.md#dui)
* 二叉搜索树

### 例题

* [LeetCode 703，返回数据流中第 k 大元素。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/queue/LT703.java)
* [LeetCode 239，数组滑动窗口中的最大值。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/queue/LT239.java)

