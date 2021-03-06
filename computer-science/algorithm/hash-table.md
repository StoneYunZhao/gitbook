# Hash Table

**定义**：散列表（Hash Table），利用数组随机访问的特性，通过散列函数（Hash函数）把元素的键值（key）映射为数组的下标（Hash 值），将数据存储在数组对应下标的位置。

查询时间复杂度原理上是 O\(1\)，但是取决于散列函数、装载因子、散列冲突等。

## 散列函数

**基本要求**：

1. 计算得到的值是非负整数
2. 若 key1 = key2，则 hash\(key1\) = hash\(key2\)
3. 若 key1 != key2，则 hash\(key1\) != hash\(key2\) // 几乎不可能实现，无法避免散列冲突

**设计原则**：

* 不能太复杂，因为不能消耗过多的计算资源。
* 生成的值要尽可能随机并且均匀分布。

**例子**：

* 数据分析法，比如手机号前面几位重复叫多，所以取后四位作为散列值。
* 平方取中法
* 随机数法
* 折叠法
* 直接寻址法

## 哈希算法

**定义**：将任意长度的二进制值串映射为固定长度的二进制串，这个映射规则就是哈希算法。映射之后得到的值就是哈希值。

哈希算法的**要求**：

1. 从哈希值不能反推原始数据
2. 对输入数据敏感，修改原始数据一个 bit，得到的 hash 值也大不相同
3. 散列冲突的概率很小，但是不能完全避免（鸽巢原理）
4. 执行效率要高，针对较长的文本，也能快速计算 hash 值

Hash 算法的**应用**：

* **安全加密**，MD5（Message Digest Algorithm），SHA（Secure Hash Algorithm）。第 1、3 点很重要。
* **唯一标识**，在海量图库中搜索图片是否存在，每张图片取 hash 值，先比较 hash 值，再比较原始图片。
* **数据校验**，BT 下载文件分割成很多块，每块计算 hash 值，保存在种子文件中，这样可以保证下载数据完整。
* **散列函数**，注重第 4 点，第 1、3 点不太重要。
* **负载均衡**，会话粘滞（session sticky）的实现，计算客户端的 IP 的 hash 值，对服务器数量取余。
* **数据分片**，在第二个例子中，图片量巨大，一台机器无法存储，所以对图片计算 hash，然后对机器数取余，放入对应服务器。
* **分布式存储**，海量数据单台机器无法存储，根据 hash 值存储在不同机器上。当数据量增加，需要增加服务器，那么需要重新计算 hash，搬迁到正确的机器。[一致性 hash 算法](../distributed-system/consistent-hashing.md#yi-zhi-xing-hash-suan-fa)，可以大量减少数据搬迁。

## 散列冲突

散列函数的第三点要求几乎是不可能的，所以会存在散列冲突，解决方案有两种，**开放寻址法**（open addressing）和**链表法**（chaining）。

### **开放寻址法**

出现散列冲突，则重新探测位置，将其插入。

不管使用哪种探测方法，当空闲位置较少时，冲突概率会大大增加。可通过装载因子（Load Factor）保证空闲比例。

**优点**：

* 数据都存在数组中，可有效利用 CPU 缓存加快查询速度。
* 序列化简单。

**缺点**：

* 删除数据较麻烦，需要标记数据已删除。
* 所有数据在数组中，冲突的代价更高

**总结**：数据量较小、装载因子较小的时候，适合开放寻址。Java 的 ThreadLocalMap 使用的是开放寻址。

#### **线性探测（Linear Probing）**

**插入**数据时若发现被占用，则依次往后找，直至有空闲位置。

**查找**数据时，先计算散列值找到对应位置，然后比较，若不相等，则往后找，若遇到空闲位置还没找到，说明数据不存在。

**删除**数据时，不能直接把元素设置为空，因为查找会有问题，而应该标记为删除。

问题较大，当数据较多时，冲突更多，时间复杂度降为 O\(n\)。

#### **二次探测（Quadratic Probing）**

二次探测步长为 hash\(key\) + 0, hash\(key\) + 1 ^ 2, hash\(key\) + 2 ^ 2, …….

#### **双重散列（Double Hashing）**

双重散列使用一组散列函数，hash1\(key\), hash2\(key\), hash3\(key\), ......

### **链表法**

散列表中每个桶（bucket）或槽（slot）会对应一个列表，散列值相同的元素放入相同槽位对应的列表中。

**优点**：

* 对内存利用率高，因为结点可以在需要时再创建。
* 对装载因子的容忍度更高，即便装载因子为 10，也不会太慢。

**缺点**：

* 需要存储指针，所以对于小对象，比较消耗内存。
* 对 CPU 缓存不友好。

**总结**：比较适合存储大对象、大量数据的情况；更加灵活，支持更多优化策略，比如用红黑树替代链表。

## 装载因子

```text
散列表的装载因子 = 填入表中的元素个数 / 散列表的长度
```

当装载因子过大时，可**动态扩容**。散列表的动态扩容需要重新计算每个数据的存储位置。

动态扩容的散列表的时间复杂度，摊还分析法，插入最好 O\(1\)，最坏 O\(n\)，平均 O\(1\)。

动态扩容的散列表，删除较多数据后，空闲空间较多，需不需要缩容取决于数据。

扩容的方式：

* 一次性扩容，当需要扩容时，全部搬迁数据。这样遇到某个插入操作就会很慢。
* 分批扩容，当需要扩容时，仅申请空间。每次插入新数据，都插入到新的散列表，并将老的散列表拿一个数据放入新的散列表。查询时，需要查询新旧两个散列表。

## 工业级散列表

Java 的 HashMap：

* 默认初始大小 16，可以设置
* 默认最大装载因子 0.75，扩容时会扩容两倍。
* 散列冲突，采用链表法，JDK1.8 后，引入红黑树。链表超过默认 8 时，转换为红黑树。红黑树结点小于 8 时，转换为链表。
* 散列函数

```text
int hash(Object key) {
    int h = key.hashCode()；
    return (h ^ (h >>> 16)) & (capitity -1); //capicity 表示散列表的大小
}
```

## 散列表与数组结合

### **LRU缓存淘汰**

基于链表的 LRU 算法时间复杂度 O\(n\)（见[链表](list.md#list)）。结合散列表与双向链表时间复杂度可到 O\(1\)。

![](../../.gitbook/assets/2.jpg)

**实现：**每个结点有四个数据，data 存储主要数据，prev 和 next 用于双向链表中，hnext 用于散列表的列表中。还需维护一个双向链表的尾结点，用于快速删除数据。

**查找：**通过散列表查找，时间复杂度 O\(1\)，查到数据后，移动到双向链表的头结点。

**删除：**通过散列表 O\(1\) 查找到需要删除的结点，然后分别在散列表的链表和双向链表中删除。

**添加：**首先确认是否存在，若存在，则移动到头结点；若不存在且没满，直接放入首部；若不存在且满，还需先删除双向链表的尾结点，再将数据放入首部。

### **Redis有序集合**

Redis 有序集合中元素有两个属性，key 和 score。

Redis 有序集合的 API 有：

1. 增加
2. 按照 key 删除
3. 按照 key 查找
4. 按照 score 区间查找
5. 按照 score 排序

**实现**：先按照 score 建立跳表，可以快速实现 API 4 和 5，然后与 LRU 类似，再按照 key 构建一个散列表。

### **Java LinkedHashMap**

```text
// 10 是初始大小，0.75 是装载因子，true 是表示按照访问时间排序
HashMap<Integer, Integer> m = new LinkedHashMap<>(10, 0.75f, true);
m.put(3, 11);
m.put(1, 12);
m.put(5, 23);
m.put(2, 22);
​
m.put(3, 26); // 会替换之前的 3，这个 3 会放到双向链表尾部
m.get(5); // 访问 5 后，被访问到的数据会移动到链表尾部
​
for (Map.Entry e : m.entrySet()) {
  System.out.println(e.getKey());
}

// output: 1 2 3 5
```

LinkedHashMap **按照访问时间排序**，Linked 表示的是双向链表。本质上，与 LRU 实现原理一样。

## 例题

* [LeetCode 242：判断两个字符串是否由相同的字母构成。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/map/LT242.java)
* [LeetCode 1：两数之和。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/map/LT01.java)
* [LeetCode 15：三数之和。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/map/LT15.java)
* [LeetCode 36：判断数独问题是否有效。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/map/LT36.java)

