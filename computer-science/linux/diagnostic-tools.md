# Diagnostic Tools

## 基础

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

## stress

系统压力测试工具：`apt install stress`，重要的参数有：

* -i, --io 模拟 N 个进程执行 sync\(\)
* -c,--cpu 模拟 N 个进程执行 sqrt\(\)
* -t,--timeout 执行 N 秒

## sysstat

系统压力测试工具：`apt install sysstat`

* mpstat：CPU 性能分析工具
  * -P ALL，输出所有 CPU 的统计
* pidstat：进程性能分析工具
  * -u，输出 CPU 使用率
  * -w，输出任务切换
    * cswch：每秒自愿上下文切换数。进程无法获取资源（IO、内存）导致上下文切换。
    * nvcswch：每秒非自愿上下文切换数。进程的时间片已到，系统强制调度。

### sar



## vmstat

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

## sysbench

多线程基准测试工具。

* --thread: 线程数
* --time: 执行多少秒

## perf

```bash
// 按方向键可定位对象，按回车可展开
$ perf top [-g]
Samples: 833  of event 'cpu-clock', Event count (approx.): 97742399
Overhead  Shared Object       Symbol
   7.28%  perf                [.] 0x00000000001f78a4
   4.72%  [kernel]            [k] vsnprintf
   4.32%  [kernel]            [k] module_get_kallsym
   3.65%  [kernel]            [k] _raw_spin_unlock_irqrestore
```

* Overhead: 比例
* Shard：动态共享对象
* Object：\[.\] 表示用户空间可以执行程序，\[k\] 表示内核空间。
* Symbol：函数名

还有`perf record [-g]`， `perf report [-g]` 等。

## ab

HTTP 服务性能测试工具。

```bash
# 并发10个请求，总共测试100个请求
$ ab -c 10 -n 100 http://192.168.0.10:10000/
```

## pstree

用树型显示所有进程之间的关系。

```bash
yum install psmisc
```

## execsnoop

专为短时进程设计的工具。它通过 ftrace 实时监控进程的 exec\(\) 行为，并输出短时进程的基本信息。

```bash
cd /usr/bin
wget https://raw.githubusercontent.com/brendangregg/perf-tools/master/execsnoop
chmod 755 execsnoop
```

