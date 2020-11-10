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
uptime

// output: 当前时间，系统运行时间，1、3、15 分钟的平均负载
17:44:09 up  1:58,  3 users,  load average: 0.00, 0.00, 0.00
```

观察某个命令输出的变化：

```bash
watch -d uptime
// -d, --differences 高亮变化的部分
```

## /proc

/proc 是 linux 提供的一种特殊的文件系统，是用户与内核交互的接口。可以从 /proc 中查询运行状态和配置选项，也可以修改 /proc 来修改内核的配置。

```bash
/proc/softirqs # 软中断
/proc/interrupts # 硬中断

/proc/stat # CPU 和任务统计信息 
/proc/[pid]/stat # 进程的 CPU 和任务统计信息

/proc/meminfo # 内存信息
```

```bash
# 配置 pages_min 值，控制 kswapd0 的行为
/proc/sys/vm/min_free_kbytes

# 配置 oom_score
/proc/[pid]/oom_adj 

# 清理文件页、目录项、Inodes 等各种缓存
echo 3 > /proc/sys/vm/drop_caches 
```

## free

```bash
➜  ~ free
              total        used        free      shared  buff/cache   available
Mem:         810492      323488      125568        1848      361436      360564
Swap:       2097148        5408     2091740
```

* total: 
* used: 包含了共享内存
* free: 
* shared: 
* buff/cache: 并不是所有的缓存都可以回收。
* available: 新进程可用的内存大小。不仅包含未使用内存，还包括了可回收的缓存。

## ps

线程名字有中括号，表示无法获取它们的命令行参数，一般都是内核线程。

## top

常用查看系统负载、进程状态的工具。

S（Status） 列表示进程状态：

* R：Running or Runnable，正在运行或正在等待运行。
* D：Disk Sleep，不可中断睡眠（Uninterruptible Sleep），一般表示正在和硬件交互，且交互过程不允许被其它进程中断或打断。
* Z：Zombie，僵尸进程，进程实际上已经结束，但是父进程还没有回收它的资源。
* S：Interruptible Sleep，可中断睡眠，进程等待某个事件而被系统挂起，唤醒后会进入 R 状态。
* I：Idle，空闲状态，不可中断的内核线程。D 会导致平均负载升高，但是 I 不会。

内存相关的列：

* VIRT：进程的虚拟内存大小。只要申请过，就算没有分配物理内存，也会计算在内。
* RES：常驻内存大小，实际使用的物理内存，不包括 swap 和共享内存。
* SHR：共享内存，不一定全部共享，程序代码段、非共享的动态链接库也计算在 SHR 内；不过真正共享的内存，共享的动态链接库自然也计算在内。
* %MEM：进程使用的物理内存占系统内存的百分比。

## stress

系统压力测试工具：`apt install stress`，重要的参数有：

* -i, --io 模拟 N 个进程执行 sync\(\)
* -c,--cpu 模拟 N 个进程执行 sqrt\(\)
* -t,--timeout 执行 N 秒

## dd

转换和复制磁盘和文件的工具。

```bash
# 从随机设备读取，写入文件
dd if=/dev/urandom of=/tmp/file bs=1M count=500

# 从随机设备数据，直接写入磁盘
dd if=/dev/urandom of=/dev/sdb1 bs=1M count=2048

# 从文件读取，写入空设备
dd if=/tmp/file of=/dev/null

# 从磁盘读取，写入空设备
dd if=/dev/sda1 of=/dev/null bs=1M count=1024
```

{% hint style="info" %}
如果把 dd 当做测试文件系统性能的工具，由于缓存的存在，会导致测试结果严重失真。
{% endhint %}

## sysstat

系统压力测试工具：`apt install sysstat`

### mpstat

CPU 性能分析工具

* -P ALL，输出所有 CPU 的统计

### pidstat

进程性能分析工具

* -u，输出 CPU 使用率
* -d，输出 I/O 统计
* -w，输出任务切换
  * cswch：每秒自愿上下文切换数。进程无法获取资源（IO、内存）导致上下文切换。
  * nvcswch：每秒非自愿上下文切换数。进程的时间片已到，系统强制调度。
* -p，指定某个进程

### sar

系统活动报告工具，可以实时查看系统当前的活动，也可以配置保存和报告历史统计数据。

```bash
# -n DEV, 报告网络设备
sar -n DEV 1
```

## dstat

新的性能工具，吸收了 vmstat、iostat、ifstat 等工具的优点。

```bash
yum install dstat

dstat 1 10
```

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

## strace

跟踪进程的系统调用。

```bash
strace -p ${pid}
```

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

pstress -aps ${pid}
```

## pgrep

根据进程名查找进程号。

## execsnoop

专为短时进程设计的工具。它通过 ftrace 实时监控进程的 exec\(\) 行为，并输出短时进程的基本信息。

```bash
cd /usr/bin
wget https://raw.githubusercontent.com/brendangregg/perf-tools/master/execsnoop
chmod 755 execsnoop
```

## hping3

可以构造 TCP/IP 协议数据包的工具。

```bash
yum install -y epel-release
yum install -y hping3

brew install hping
```

```bash
# -S tcp 协议的 SYN
# -p 目标端口哦
# 发送间隔
hping3 -S -p 80 -i u100 10.93.245.152
```

## tcpdump

常用的网络抓包工具。

```bash
yum install -y tcpdump
```

```bash
# -i 网卡
tcpdump -i ens33 -n tcp port 80
```

## bcc

基于 Linux 内核的 eBPF 机制。

### cachestat

提供整个操作系统缓存的读写命中情况。

```bash
$ cachestat 1 3
   TOTAL   MISSES     HITS  DIRTIES   BUFFERS_MB  CACHED_MB
       2        0        2        1           17        279
       2        0        2        1           17        279
       2        0        2        1           17        279 
```

* TOTAL: 总 IO 次数。一次代表一页，为 4KB。
* MISSES: 未命中缓存次数
* HITS: 命中缓存次数
* DIRTIES: 新增到缓存中的脏页数
* BUFFERS\_MB: Buffer的大小
* CACHED\_MB: Cache 的大小

### cachetop

提供每个进程的缓存命中情况。

```bash
$ cachetop
11:58:50 Buffers MB: 258 / Cached MB: 347 / Sort: HITS / Order: ascending
PID      UID      CMD              HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
   13029 root     python                  1        0        0     100.0%       0.0%
```

{% hint style="info" %}
不会吧 Direct IO 算进来。
{% endhint %}

### memleak

可以跟踪系统或指定进程的内存分配、释放请求，然后定期输出一个未释放内存和相应调用栈的汇总情况。

## pcstat

Go 开发的。查看文件在内存中的缓存大小及缓存比例。

```bash
$ pcstat /bin/ls
+---------+----------------+------------+-----------+---------+
| Name    | Size (bytes)   | Pages      | Cached    | Percent |
|---------+----------------+------------+-----------+---------|
| /bin/ls | 133792         | 33         | 0         | 000.000 |
+---------+----------------+------------+-----------+---------+
```

