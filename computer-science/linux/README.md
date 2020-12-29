# Operating System

首先需要了解操作系统的基本原理：

* [基础](basic.md)
* [系统初始化](system-initialization.md)

然后需要用到实践中去，比如常见的性能工具：

![](../../.gitbook/assets/image%20%28277%29.png)

以及性能优化手段：

* 常用的[性能诊断工具](diagnostic-tools.md)
* [CPU](cpu-diagnosis.md)
  * [平均负载](cpu-diagnosis.md#ping-jun-fu-zai)
  * [CPU 上下文切换](cpu-diagnosis.md#shang-xia-wen-qie-huan)
  * [CPU 使用率](cpu-diagnosis.md#shi-yong-lv)。使用率过高的常见原因有：
    * 上下文切换
    * [iowait 过高](cpu-diagnosis.md#bu-ke-zhong-duan-zhuang-tai)，表现为进程长时间处于 D 状态，或较多进程处于 D 状态
    * [僵尸进程过多](cpu-diagnosis.md#jiang-shi-jin-cheng)
    * [软中断过多](cpu-diagnosis.md#ruan-zhong-duan)
  * [CPU 优化思路](cpu-diagnosis.md#cpu-xing-neng-you-hua)
* [内存](memory-diagnosis.md)
  * Linux 内存的一些[基本原理](memory-diagnosis.md#yuan-li)
  * [Buffer 与 Cache 的区别](memory-diagnosis.md#buffer-vs-cache)
  * [缓存命中率](memory-diagnosis.md#huan-cun-ming-zhong-lv)
  * [内存泄漏](memory-diagnosis.md#nei-cun-xie-lou)
  * [Swap](memory-diagnosis.md#swap)
  * [优化思路](memory-diagnosis.md#zong-jie)
* [磁盘](i-o-diagnosis.md)
  * [文件系统](i-o-diagnosis.md#wen-jian-xi-tong)
  * [磁盘](i-o-diagnosis.md#ci-pan)
  * [优化思路](i-o-diagnosis.md#you-hua-si-lu)
* [网络](network-diagnosis.md)
* [监控系统](monitor-system.md)建设

![](../../.gitbook/assets/image%20%28280%29.png)

