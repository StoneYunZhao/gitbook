# List

## 概念

**缓存淘汰策略**：

* 先进先出FIFO（First in, First out）
* 最少使用LFU（Least Frequently Used）
* 最近最少使用LRU（Least Recently Used）

**数组与链表比较**，从存储结构来说，数组需要连续的内存空间，容易产生内存不足的错误，不能动态扩容，单连续内存的特性可借助CPU的缓存机制，访问效率更高；而链表不需要，天然支持动态扩容，但是频繁的插入删除会产生较多的内存碎片。

**链表的种类**：单链表、双向链表、循环链表。

**定义**：链表通过指针把零散的内存块串联在一起。内存块称为结点，结点上有数组和后继指针next。第一结点叫头结点，记录链表的基地址；最后一个结点叫尾结点，指针指向NULL。

**时间复杂度**：链表插入和删除时间复杂度为O\(1\)，注意仅是单纯的插入和删除操作，但是一般要找到删除的位置，这个查找过程时间复杂度为O\(n\)，所以删除给定值的总时间复杂度为O\(n\)。随机访问第k个元素，平均时间复杂度为O\(n\)。

**循环链表：**尾结点指向头结点。相比于单链表，从链尾到链头比较方便，处理具有环形结构时合适。

**双向链表**：

* 每个结点有后继指针`next`和前驱指针`prev`。比单链表占用更多空间，但支持双向遍历。
* 可以支持`O(1)`时间复杂度找到前驱结点。
* 删除给定指针指向的节点，单链表时间复杂度为`O(n)`，而双向链表为`O(1)`，因为单链表需要遍历找到前驱结点。插入类似。
* 对于**有序链表**，双向链表的查询效率比单链表高很多，因为可以记录前一次查询p的位置，在下一次查询与p比较，决定向前还是向后查询。
* **`LinkeHashMap`**用到了双向链表。
* 双向链表即是**空间换时间**的例子。

**双向循环链表**：即循环链表与双向链表的组合。

**LRU实现**：维护一个有序单向链表，当有新数据访问时，重头至尾遍历，如果找到，则删除原位置，插入到头部；若没有找到且未满，则插入头部；若没有找到且已满，则删除尾结点，插入头部。时间复杂度为`O(n)`，可用`HashTable`优化，时间复杂度为`O(1)`。

链表代码技巧：

* 利用**哨兵**简化难度，针对链表的插入、删除，需要对插入第一个结点和删除最后一个结点做特殊处理。可以增加一个哨兵结点，不存储数据，head指针一直指向这个结点，有哨兵的链表叫**带头链表（有头结点链表）**。
* 重点留意边界条件，若链表为空，若链表只有一个结点，若只有两个结点，在处理头结点和尾结点时，能否正常工作。
* 画图辅助思考。

链表检测题：

## 代码

### 链表

```java
/**
 * 有头结点的列表
 */
public class LNode {
    Integer data;
    LNode next;

    public LNode() {
    }

    public LNode(int data) {
        this.data = data;
    }

    public static void printList(LNode head) {
        if (head == null) {
            System.out.println("null");
        }

        StringBuilder result = new StringBuilder();
        for (LNode current = head; current != null; current = current.next) {
            result.append(current.data).append(" ");
        }
        System.out.println(result);
    }
}
```

### 链表反转

都是针对带头链表。

```java
public class ReverseList {

    /**
     * 就地逆序，遍历，让列表的结点指向其前驱结点，
     */
    private static void reverseInPlace(LNode head) {
        System.out.println("reverse in place");
        if (head == null || head.next == null || head.next.next == null) {
            return;
        }
        LNode first, second;

        for (first = head.next, second = first.next; second != null; ) {
            LNode tmp = second.next;
            second.next = first;
            first = second;
            second = tmp;
        }
        head.next.next = null;
        head.next = first;
    }

    /**
     * 插入法，遍历列表，每次将遍历到的结点插入 head 结点之后
     */
    private static void reverseInsert(LNode head) {
        System.out.println("reverse by insert");
        if (head == null || head.next == null) {
            return;
        }
        LNode end = head.next;

        for (LNode current = head.next; current != null; ) {
            LNode tmp = current.next;
            current.next = head.next;
            head.next = current;
            current = tmp;
        }
        end.next = null;
    }
}
```

### 无序链表移除重复项



* 链表中检测环。
* 两个有序链表合并。
* 删除链表倒数第n个结点。
* 求链表的中间结点。

