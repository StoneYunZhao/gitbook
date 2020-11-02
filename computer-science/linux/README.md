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
* 内存
* I/O
* 网络

![](../../.gitbook/assets/image%20%28280%29.png)

