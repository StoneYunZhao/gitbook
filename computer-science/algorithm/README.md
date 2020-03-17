# Algorithm

**数据结构是指一组数据的存储结构**，如队列、栈、堆，**算法是操作数据的一组方法**，如二分查找、动态规划。

数据结构与算法是相辅相成的，**数据结构是为算法服务的，算法需要作用在特定的数据结构上**。比如数组具有随机访问的特点，二分查找需要用数组来存储数据，若用链表，二分查找就无法工作了。

十个重要的数据结构：数组、链表、栈、队列、散列表、二叉树、堆、跳表、图、Trie树。

十个重要的算法：递归、排序、二分查找、搜索、哈希算法、贪心算法、分治算法、回溯算法、动态规划、字符串匹配算法。

对于数据结构和算法，不只是要了解它是什么，更重要的是要知道：它的**来历**、**特点**、**解决的问题**、**应用场景**。

* [复杂度分析](complexity.md)
  * [时间复杂度](complexity.md#shi-jian-fu-za-du)
  * [空间复杂度](complexity.md#kong-jian-fu-za-du)
* [线性表结构](list.md)
  * [Array](list.md#array)
  * [List](list.md#list)
  * [Stack](list.md#stack)
  * [Queue](list.md#queue)
* [排序](sort.md)
* [查找](search.md)
* [跳表](skip-list.md)
* [哈希表](hash-table.md)
* [Tree](tree.md)
  * [二叉查找树](tree.md#er-cha-cha-zhao-shu)
  * [平衡二叉查找树](tree.md#ping-heng-er-cha-cha-zhao-shu)
  * [红黑树](tree.md#hong-hei-shu)
  * [递归树](tree.md#di-gui-shu)
  * [堆](tree.md#dui)
  * [B 树](tree.md#b-shu)
  * [B+ 树](tree.md#b-shu-1)
* [图](graph.md)
* [字符串匹配算法](string-matching.md)
* 算法思想：
  * [贪心算法](greedy-algorithm.md)
  * [分治算法](divide-and-conquer.md)
  * [回溯算法](back-tracking.md)
  * [动态规划](dynamic-programming.md)

四种算法思想比较：

* 分治算法解决的问题大部分都不能抽象成多阶段决策问题；贪心、回溯、动态规划解决的问题都可以抽象成多阶段决策。
* 回溯算法是“万金油”，能用动态规划、贪心解决的问题，基本都能用回溯解决，回溯相当于穷举，时间复杂度非常高。
* 动态规划能解决的问题需要满足最优子结构、无后效性、重复子问题三个特性；动态规划高效的原因就是回溯算法中有大量重复子问题，而分治算法相反，要求子问题不能重复。
* 贪心算法本质是动态规划的一种特殊情况，解决问题需要满足最优子结构、无后效性、贪心选择性，不用强调重复子问题。贪心选择性的意思是通过局部的最优选择，能产生全局的最优选择。



![](../../.gitbook/assets/image%20%28225%29.png)

