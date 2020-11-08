# Memory Diagnosis

## 原理

Linux 给每个进程都提供了一个独立的虚拟地址空间，连续的。虚拟地址空间又分为内核空间（高位）和用户空间（低位）。进程在用户态时，只能访问用户空间内存；进入内核态后才能访问内核空间内存。**每个进程的内核空间关联的是相同的物理内存**。

![](../../.gitbook/assets/image%20%28290%29.png)

并不是所有的虚拟内存都会分配物理内存，内核为每个进程都维护了一张页表，记录虚拟地址与物理地址的映射关系。页表存储在 MMU 中。虚拟地址在页表中查不到时，系统产生一个**缺页异常**，进入内核空间分配物理内存、更新进程页表，最后再返回用户空间，恢复进程的运行。

![](../../.gitbook/assets/image%20%28292%29.png)

TLB 是 MMU 的高速缓存，若上下文切换过多，则 TLB 的刷新次数增加，进程的 MMU 是独立的，所以会导致 TLB 缓存的使用率降低。

MMU 规定了内存映射的最小单位，一般为 4KB。页仅为 4KB，页表就会很大，Linux 提供了**多级页表**和**大页**两种方式来解决此问题。Linux 用四级页表来管理内存，前四个表用于选择页，最后一个索引表示页内偏移。

![](../../.gitbook/assets/image%20%28293%29.png)

虚拟内存空间的用户空间又被分为多个不同的段。以 32位系统为例：

* 只读段，代码和常量。不会内存泄漏。
* 数据段，全局变量。不会内存泄漏。
* 堆，动态分配的内存，malloc\(\) 分配。调用 free\(\) 回收，可能产生内存泄漏问题。
* 文件映射段，动态库，共享内存 mmap\(\) 分配。可能产生内存泄漏问题。
* 栈，局部变量，函数调用的上下文。大小固定，一般 8MB。由系统自动分配与管理，内存会被系统自动回收，不会产生内存泄漏问题。

![](../../.gitbook/assets/image%20%28291%29.png)

malloc 对应到系统调用上，有 brk\(\) 和 mmap\(\) 两种，brk 适合小块内存（小于 128K），而 mmap 适合大块内存。

* brk：通过移动堆顶位置来分配内存。内存释放后不会立即归还给系统，而是被缓存起来，重复使用。可以减少缺页异常的发生，提高内存响应效率。但是会造成内存碎片。
* mmap：在文件映射段找一块空闲内存分配出去。释放时直接归还给系统，所以每次 mmap 都会发生缺页异常。

这两种调用后并没有分片真正的内存，只有在首次访问的时候才分配。

系统有一系列回收内存的机制：

* 回收缓存
* 回收不常访问的内存，通过交换分区写入磁盘中
* 杀死进程。系统通过 oom\_score 为每个进程的内存使用情况评分，消耗内存越大、占用 CPU 使用越小，oom\_score 越大，越容易被杀死。可以通过 /proc 设置 oom\_adj

可以通过 free 查看系统的整体内存情况，通过 top、ps 查看进程的内存使用情况。

## Buffer vs. Cache

free 输出的缓存一列是 Buffer 与 Cache 之和，那两者有什么区别呢？通过 man free 可以看到：

* buffers: Memory used by kernel buffers \(Buffers in /proc/meminfo\)
* cache: Memory used by the page cache and slabs \(Cached and SReclaimable in /proc/meminfo\)

进一步查看 /proc/meminfo 的定义，通过 man proc：

* Buffers: Relatively temporary storage for raw disk blocks that shouldn't get tremendously large \(20MB or so\). 原始磁盘块的临时存储。
* Cached: In-memory cache for files read from the disk \(the page cache\). Doesn't include SwapCached. 读取文件的页缓存。**实际上，用 dd 和 vmstat 观测下来，写文件也会使用 Cache缓存**。
* SReclaimable: Part of Slab, that might be reclaimed, such as caches. Slab 中可回收的部分。

**Buffer 是对磁盘数据的缓存，而 Cache 是文件数据的缓存，它们既会用在读请求中，也会用在写请求中**。

{% hint style="info" %}
磁盘是块设备，可以划分不同分区。分区之上可以再创建文件系统，挂载到某个目录。通常写文件时会经过文件系统，文件系统负责与磁盘交互。裸 IO 直接读写磁盘或分区，会跳过文件系统，存在 buffer。注意裸 IO 与直接 IO 的区别，直接 IO 跳过 buffer，直达块层，需要用户处理对齐问题。
{% endhint %}

## 缓存命中率

通过缓存获取数据的请求次数占所有请求次数的百分比。可以通过 cachestat、cachetop 查看 Linux 缓存命中的情况。

