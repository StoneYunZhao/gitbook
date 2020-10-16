# CPU Diagnosis

## 工具

### 基础

查看cpu：

```bash
cat /proc/cpuinfo
lscpu
cpuid
```

查看系统负载：

```bash
top

uptime

// output: 当前时间，系统运行时间，1、3、15 分钟的平均负载
17:44:09 up  1:58,  3 users,  load average: 0.00, 0.00, 0.00
```

观察某个命令输出的变化：

```bash
watch -d uptime
// -d, --differences 高亮变化的部分
```

### stress

系统压力测试工具：`apt install stress`，重要的参数有：

* -i, --io 模拟 N 个进程执行 sync\(\)
* -c,--cpu 模拟 N 个进程执行 sqrt\(\)
* -t,--timeout 执行 N 秒

### sysstat

系统压力测试工具：`apt install sysstat`

* mpstat：CPU 性能分析工具
  * -P ALL，输出所有 CPU 的统计
* pidstat：进程性能分析工具
  * -u，输出 CPU 使用率
  * -w，输出任务切换
    * cswch：每秒自愿上下文切换数。进程无法获取资源（IO、内存）导致上下文切换。
    * nvcswch：每秒非自愿上下文切换数。进程的时间片已到，系统强制调度。

### vmstat

系统性能分析工具，主要用于分析系统内存使用情况、CPU 上下文切换、中断次数等。

```bash
vmstat [options] [delay [count]]

// 每隔 5 秒输出一次
➜  ~ vmstat 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 1719840  17412 151560    0    0     1     0   24   31  0  0 100  0  0
 0  0      0 1719808  17420 151560    0    0     0     6   59   66  0  0 100  0  0
```

* cs\(context switch\)：每秒上下文切换次数。
* in\(interrupt\)：每秒中断次数。
* r\(runnable\)：正在运行和等待 CPU 的进程数。
* b\(uninterruptible\)：不可中断睡眠状态的进程数。

### sysbench

多线程基准测试工具。

* --thread: 线程数
* --time: 执行多少秒

## 平均负载

我们经常会用 uptime、top 等工具查询 cpu 的平均负载（Load Average），那到底什么是平均负载呢？

通过 man uptime 可以看到平均负载的定义：**System load averages is the average number of processes that are either in a runnable or uninterruptable state. A process in a runnable state is either using the CPU or waiting to use the CPU. A process in uninterruptable state is wait- ing for some I/O access, eg waiting for disk.** 

简单来说，即单位时间内系统处于可运行状态或不可中断状态的平均进程数。如平均负载为 2 时，系统有 2 个 CPU，意味着 CPU 刚好被完全占用。三个时间间隔的平均负载反映了趋势。所以**平均负载高并不能反映问题出在哪里，可能是 CPU 使用过高，也可能是 IO 等待过长**。

* 可运行状态（ps 命令中看到的 R 状态）：
  * 正在使用 CPU
  * 等待使用 CPU
* 不可中断状态（ps 命令中看到的 D 状态）：等待某些硬件设备的 IO 响应。

**经验：当平均负载高于 CPU 个数的 70% 的时候，就需要关注性能问题了。**

{% hint style="success" %}
**平均负载与 CPU 使用率**  
两者并不一定是完全对应的，从定义可以看出，平均负载是指活跃的进程个数，活跃的进程还包括了等待 CPU 和等待 IO 的进程。

* CPU 密集型：两个是一致的
* IO 密集型：等待 IO 会使平均负载升高，但 CPU 使用率不一定高
* 大量的进程调用会使平均负载升高，CPU 使用率也会升高
{% endhint %}

### 压力测试

```bash
# CPU 密集型, 模拟一个 CPU 使用 100%
stress -c 1 -t 10

# IO 密集型, 不停地执行 sync
stress -i 1 -t 600

# 大量进程
stress -c 8 -t 600

# 查看 uptime 命令输出的变化
watch -d uptime

# 每隔 5 秒输出一组所有 CPU 的使用率
mpstat -P ALL 5

# 查看哪个进程占用的 CPU
pidstat -u 5 1
```

## 上下文切换

每个任务运行前，CPU 都需要知道从哪里加载、从哪个位置开始运行，所以系统需要先设置好 CPU 寄存器和程序计数器（Program Counter, PC）。

寄存器是 CPU 内置的容量小、速度极快的内存；PC 用于存储正在执行的指令位置、或下一条指令位置。这两者是 CPU 在运行任何任务前必须依赖的环境，所以也叫 CPU 上下文。

CPU 上下文切换就是把前一个任务的上下文（寄存器和 PC）保存到内核中，加载新任务的上下文到寄存器和 PC。

根据任务和不同，可以分为**进程上下文切换、线程上下文切换、中断上下文切换**。

### 系统调用

Linux 把进程的运行空间分为内核空间（最高权限，可以访问所有资源）和用户空间（必须通过系统调用陷入到内核中才能访问内存等硬件设备），分别对应 Ring0 和 Ring3。

![](../../.gitbook/assets/image%20%28285%29.png)

系统调用时，CPU 寄存器中用户态的指令位置需要先保存起来，更新为内核态指令的位置；系统调用结束后，又恢复为用户态。所以**一次系统调用会发生两次 CPU 上下文切换**。

{% hint style="info" %}
系统调用不需要涉及虚拟内存等资源，也不会切换进程，所以系统调用通常称为特权模式切换，而不是上下文切换。
{% endhint %}

### 进程上下文切换

进程是由内核来管理和调度的，进程切换只能发生在内核态。所以进程切换不仅包括用户态资源（虚拟内存、栈、全局变量），还包括内核态资源（内核堆栈、寄存器）。

Linux 为每个 CPU 维护一个队列，将活跃进程（正在运行或正在等待 CPU）按照优先级和等待 CPU 的时长排序。进程调度的时机发生在：

* 一个进程执行结束。
* 分时操作系统中进程的时间片用完了
* 进程通过 sleep 将自己主动挂起
* 有更高优先级的进程
* 硬件中断

### 线程上下文切换

**线程是调度的基本单位，而进程是资源拥有的基本单位**。内核中的任务调度对象都是线程，而进程只是给线程提供了虚拟内存、全局变量等资源。所以：

* 进程只有一个线程时，进程就等于线程。
* 进程有多个线程时，线程会共享相同的虚拟内存和全局变量等资源，这些资源在上下文切换时不需要修改。
* 线程也有自己的私有数据，如栈和寄存器，这些在上下文切换是需要保存的。

由此可见，若两个线程不属于同一进程，则上下文切换与进程切换一样；若属于同一进程，切换时只需要切换线程的私有数据。**这也是多线程替代多进程的一个优势**。

### 中断上下文切换

为了快速响应硬件事件，中断会打断进程的正常执行，转而调度中断处理程序。中断不涉及进程的用户态，所以中断上下文切换不需要保存和恢复用户态资源。对于同一 CPU， 中断处理比进程有更高的优先级。

### 案例

```bash
# 1. 查看空闲状态时的状态
vmstat 1 1

# 2. 模拟8个线程
sysbench --threads=10 --max-time=300 threads run

# 3. 查看此时的上下文切换
vmstat 1

# 4. 查看是哪个进程导致的
pidstat -w -u 1

# 5. 查看线程上下文切换
pidstat -wt 1

# 6. 查看中断类型
watch -d cat /proc/interrupts
```

### 总结

上下文切换 1 万以内都算正常，若超过 1 万次、或出现量级增长，就可能出现了性能问题。不过这是经验数据，具体还要看配置等实际场景。

* 自愿上下文切换变多，说明进程都在等待资源，可能发生了 IO 等其它问题。
* 非自愿上下文切换变多，说明都在争抢 CPU，确实 CPU 出现了瓶颈。
* 中断次数变多，需要查看 /proc/interrupts 具体分析。

