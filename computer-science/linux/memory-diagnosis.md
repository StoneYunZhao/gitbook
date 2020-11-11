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

* 回收缓存，基于 LRU
* 回收不常访问的内存，通过交换分区（Swap）写入磁盘中，也是基于 LRU
* 杀死进程。系统通过 oom\_score 为每个进程的内存使用情况评分，消耗内存越大、占用 CPU 使用越小，oom\_score 越大，越容易被杀死。可以通过 /proc 设置 oom\_adj。

```bash
# 查看被系统 OOM 杀死的进程
dmesg | grep -i "Out of memory"
```

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

## 内存泄漏

进程看到的是操作系统提供的虚拟内存空间，通过页表映射到物理内存。虚拟内存空间中有内核空间和用户空间，用户空间中的堆和内存映射段由应用程序自己来分配和管理内存，若没有正确释放内存，可能造成内存泄漏。避免内存泄漏通常有如下要点：

* malloc\(\) 和 free\(\) 通常并不是成对出现，所以需要在每个异常处理路径和成功路径上都释放内存 。
* 在多线程程序中，一个线程中分配的内存，可能会在另一个线程中访问和释放。
* 在第三方的库函数中，隐式分配的内存可能需要应用程序显式释放。

## Swap

在原理一节我们已经知道，当发生内存泄漏或运行了大内存的程序，从而导致系统内存资源紧张时，系统会有内存回收和 OOM 两种处理方式。

内存回收就是释放可以回收的内存，它们通常叫做文件页（File backed Page），文件页包含缓存、缓冲区、内存映射获取的文件映射页，文件页分为两种回收方式：

* 直接回收，再需要时，重新从磁盘读取。
* 文件页被应用程序修改过，暂时还没写入磁盘（脏页），得先写入磁盘再释放。写入磁盘的方式又分为两种：
  * 应用程序通过系统调用 fsync 把脏页同步到磁盘。
  * 内核线程 pdflush 负责脏页的刷新。

可回收内存除了文件页，还包含匿名页（Anonymous Page），即应用程序动态分配的堆内存。由于这些内存有应用程序管理，所以系统不能直接回收，Linux 通过 Swap 机制把不常访问的脏页写入磁盘，然后释放这些内存，再次访问这些内存时，重新从磁盘读入内存。

Swap 可以是一块磁盘，也可以是一个文件，当成内存来使用，包括两个过程：

* 换入：从磁盘读入内存。
* 换出：内存写入磁盘，释放内存。

Swap 可以使系统内存变大，笔记本电脑的休眠和快速开机功能，也是基于 Swap。

```bash
swapoff -a
swapon -a
```

可通过 sar、/proc/zoneinfo、/proc/pid/status 等方式查看系统和进程的内存使用情况。swap 会影响性能，常用如下方法降低 swap 的使用：

* 禁用 swap
* 降低 swappiness 的值
* mlock\(\), mlockall\(\) 锁定内存，阻止换出

### kswap0

触发系统回收内存有两种方式：

* 直接内存回收，有新的大块内存分配请求，但是内存剩余不足，这时系统会回收一部分内存。
* kswap0，内核线程，定期回收。

kswap0 定义了三个阈值（watermark），pages\_min, pages\_low, pages\_high，剩余内存用 pages\_free 表示。根据剩余内存与三个阈值的关系，进行内存操作：

* 小于 pages\_min，仅内核可以分配内存。
* pages\_min ~ pages\_low，执行内存回收，直到大于 pages\_high。
* pages\_low ~ pages\_high，内存有一定压力，可以满足新内存请求。
* 大于 pages\_high，没有内存压力。

![](../../.gitbook/assets/image%20%28299%29.png)

可以通过`/proc/sys/vm/min_free_kbytes`设置 pages\_min，其它两个阈值的关系是固定的：

```bash
pages_low = pages_min*5/4
pages_high = pages_min*3/2
```

### NUMA 与 Swap

有时发现系统 swap 升高，但是系统还有很多空余内存，这可能就是 NUMA（Non-Uniform Memory Access）导致的。在 NUMA 架构下，多个处理器划分到不同的 Node 上，每个 Node 有自己的本地内存空间。每个 node 内部内存又分为不同的区域：直接内存访问区（DMA）、普通内存区（NORMAL）、伪内存区（MOVABLE）。可通过 numactl 查看 node 信息。

![](../../.gitbook/assets/image%20%28297%29.png)

可通过 /proc/zoneinfo 查看每个 node 的三个阈值：

* nr\_\*\_anon：活跃和非活跃的匿名页数。
* nr\_\*\_file：活跃和非活跃的文件页数。

```bash
cat /proc/zoneinfo

Node 0, zone    DMA32
  pages free     34699
        min      9454
        low      11817
        high     14180
...
      nr_free_pages 34699
      nr_zone_inactive_anon 19437
      nr_zone_active_anon 17516
      nr_zone_inactive_file 27432
      nr_zone_active_file 42315
```

某个 Node 内存不足时，可以从其它 node 寻找空闲内存，也可以从本地内存中回收。可通过 /proc/sys/vm/zone\_reclaim\_mode 调整：

* 0 表示两种方式都可以
* 1，2，4 表示只回收本地内存，2 表示可回写脏数据回收内存，4 表示可用 Swap 的方式。

### swappiness

通过上文，我们知道内存回收：

* 文件页：直接回收或把脏页写回磁盘再回收
* 匿名页：通过 Swap 回收

那么 Linux 优先采用哪种呢，可设置 /proc/sys/vm/swappiness 来调整使用 swap 的积极程度，范围是 0 ~ 100。注意这是权重，就算设置为 0，也会使用 swap。

## 总结

### 内存指标

![](../../.gitbook/assets/image%20%28294%29.png)

#### 系统内存使用情况

* 已用内存
* 剩余内存
* 共享内存，通过 tempfs 实现，大小等于 tmpfs 的大小。
* 可用内存，剩余内存+可回收缓存
* 缓存（Cache）：
  * 磁盘读取文件的页缓存
  * slab 分配器中的可回收内存
* 缓冲区（Buffer），对原始磁盘块的临时存储

#### 进程内存使用情况

* 虚拟内存，代码段+数据段+共享内存+已经申请的堆内存+已经换出的内存（Swap）。注意：已经申请，还未分配物理内存，也算在内。
* 常驻内存，实际使用的物理内存，不包括 Swap 和共享内存。一般换算成系统总内存的百分比。
* 共享内存，动态加载的链接库+程序代码段。
* Swap 内存。

#### 缺页异常

进程申请内存后，首次访问时系统才会通过缺页异常分配内存，有两种场景：

* 次缺页异常：可以直接从物理内存分配
* 主缺页异常：需要磁盘 I/O 介入（如 Swap）

#### Swap 使用情况

* 已用空间
* 剩余空间
* 换入和换出速度

### 指标 -&gt; 工具

![](../../.gitbook/assets/image%20%28296%29.png)

### 工具 -&gt; 指标

![](../../.gitbook/assets/image%20%28295%29.png)

### 分析思路

先运行几个覆盖面较大的性能工具，free, top, vmstat, pidstat 等：

1. free, top 查看系统整体内存情况
2. vmstat, pidstat 查看一段时间的趋势，判断内存问题类型
3. 详细分析

![](../../.gitbook/assets/image%20%28298%29.png)

### 优化思路

内存调优最重要的就是，保证应用程序的热点数据放到内存中，并尽量减少换页和交换。

* 禁止 Swap。若必须开启，则降低 swappiness 的值。
* 减少动态内存分配。使用内存池、大页（HugePage）等。
* 尽量使用 Cache、Buffer 来访问数据。
* 使用 cgroups 限制进程内存使用。
* 调整核心进程的 /proc/pid/oom\_adj，保证内存紧张时，核心进程不会被 OOM 杀死。

