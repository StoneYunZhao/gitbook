# CPU Diagnosis

## 平均负载

我们经常会用 uptime、top 等工具查询 cpu 的平均负载，那到底什么是平均负载呢？

uptime 命令显示的分别是当前时间，系统运行时间，1、3、15 分钟的平均负载。

```bash
➜  ~ uptime
 17:44:09 up  1:58,  3 users,  load average: 0.00, 0.00, 0.00
```

通过 man uptime 可以看到平均负载的定义：**System load averages is the average number of processes that are either in a runnable or uninterruptable state. A process in a runnable state is either using the CPU or waiting to use the CPU. A process in uninterruptable state is wait- ing for some I/O access, eg waiting for disk.** 

简单来说，即单位时间内系统处于可运行状态或不可中断状态的平均进程数。如平均负载为 2 时，系统有 2 个 CPU，意味着 CPU 刚好被完全占用。三个时间间隔的平均负载反映了趋势。所以**平均负载高并不能反映问题出在哪里，可能是 CPU 使用过高，也可能是 IO 等待过长**。

* 可运行状态（ps 命令中看到的 R 状态）：
  * 正在使用 CPU
  * 等待使用 CPU
* 不可中断状态（ps 命令中看到的 D 状态）：等待某些硬件设备的 IO 响应。

**经验：当平均负载高于 CPU 个数的 70% 的时候，就需要关注性能问题了。**

{% hint style="info" %}
获取 CPU 个数的方法

* `cat /proc/cpuinfo`
* `lscpu`
* `cpuid`
{% endhint %}

{% hint style="success" %}
**平均负载与 CPU 使用率**  
两者并不一定是完全对应的，从定义可以看出，平均负载是指活跃的进程个数，活跃的进程还包括了等待 CPU 和等待 IO 的进程。

* CPU 密集型：两个是一致的
* IO 密集型：等待 IO 会使平均负载升高，但 CPU 使用率不一定高
* 大量的进程调用会使平均负载升高，CPU 使用率也会升高
{% endhint %}

### 压力测试

* stress：系统压力测试工具
* sysstat：性能分析工具
  * mpstat：CPU 性能分析工具
  * pidstat：进程性能分析工具

```bash
## CPU 密集型

# 模拟一个 CPU 使用 100%
stress --cpu 1 --timeout 10

# 查看 uptime 命令输出的变化
watch -d uptime

# 每隔 5 秒输出一组所有 CPU 的使用率
mpstat -P ALL 5

# 查看哪个进程占用的 CPU
pidstat -u 5 1

## IO 密集型, 不停地执行 sync
stress -i 1 --timeout 600

## 大量进程
stress -c 8 --timeout 600
```

