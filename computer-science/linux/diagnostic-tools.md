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

/proc/slabinfo # 查看 slab 信息

/proc/zoneinfo # NUMA 架构中每个 node 的信息

/proc/diskstats # 查看磁盘统计信息
```

```bash
# 配置 pages_min 值，控制 kswapd0 的行为
/proc/sys/vm/min_free_kbytes

# 配置 oom_score
/proc/[pid]/oom_adj 

# 清理文件页、目录项、Inodes 等各种缓存
echo 3 > /proc/sys/vm/drop_caches 

# NUMA 架构下，节点回收内存的方式，具体查看内存一节。
/proc/sys/vm/zone_reclaim_mode

# 配置系统使用 swap 的积极程度，具体查看内存一节。
/proc/sys/vm/swappiness
```

## ps

线程名字有中括号，表示无法获取它们的命令行参数，一般都是内核线程。

```bash
ps -ef
# -T 展示线程

# 查看所有内核线程
ps -f --ppid 2 -p 2
ps -ef | grep "\[.*\]"
```

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
# -n 查看网络统计信息。DEV（网络接口）、EDEV（网络接口错误）、TCP、UDP、ICMP
sar -n DEV 1

# -r 显示内存使用情况，-S 显示 Swap 使用情况
sar -r -S 1
# kbcommit, %commit: 对内存需求的预估
# kbactive: 活跃内存
# kbinact: 非活跃内存
```

* rxpck/s, txpck/s：PPS，包/秒。
* rxKB/s, txKB/s：吞吐量。
* rxcmp/s, tcxmp/s：压缩包数据包数。
* %ifutil：网络接口使用率。半双工模式下为 \(rxkB/s+txkB/s\)/Bandwidth，而全双工模式下为 max\(rxkB/s, txkB/s\)/Bandwidth。Bandwidth 可用ethtool 查看。

### iostat

数据来自 /proc/diskstats。

```bash
iostat -d -x 1
# -d 展示设备使用率
# -x 展示扩展统计信息
```

![](../../.gitbook/assets/image%20%28306%29.png)

* r/s + w/s 就是 IOPS
* rKB/s + wKB/s 就是吞吐量
* r\_await + w\_await 就是响应时间

{% hint style="info" %}
饱和度一般无法直接获取，可通过与基础测试结果（fio）比较来评估饱和度。
{% endhint %}

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
yum install -y strace

strace -p ${pid} -f
# -f 跟踪多线程。
# -T 系统调用时长
# -tt 跟踪时间
# -e expr 跟踪特定的系统调用
```

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

## numactl

查看、控制 NUMA 中的 node 信息。

```bash
numactl --hardware
```

## bcc

基于 Linux 内核的 eBPF 机制。

### CentOS Install

CentOS 7

#### 升级内核

```bash
# 升级系统
yum update -y

# 安装ELRepo
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

# 安装新内核
yum remove -y kernel-headers kernel-tools kernel-tools-libs
yum --enablerepo="elrepo-kernel" install -y kernel-ml kernel-ml-devel kernel-ml-headers kernel-ml-tools kernel-ml-tools-libs kernel-ml-tools-libs-devel

# 更新Grub后重启
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-set-default 0
reboot

# 重启后确认内核版本已升级为4.20.0-1.el7.elrepo.x86_64
uname -r
```

#### 安装 bcc-tools

```bash
# 安装bcc-tools
yum install -y bcc-tools

# 配置PATH路径
export PATH=$PATH:/usr/share/bcc/tools

# 验证安装成功
cachestat 
```

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

### filetop

跟踪内核中文件的读写情况。

### opensnoop

动态跟踪内核中 open 系统调用。

## 内存相关

### free

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

### slabtop

查看 slab 信息。

### pcstat

Go 开发的。查看文件在内存中的缓存大小及缓存比例。

```bash
$ pcstat /bin/ls
+---------+----------------+------------+-----------+---------+
| Name    | Size (bytes)   | Pages      | Cached    | Percent |
|---------+----------------+------------+-----------+---------|
| /bin/ls | 133792         | 33         | 0         | 000.000 |
+---------+----------------+------------+-----------+---------+
```

## 磁盘相关

### dd

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

### df

查看文件系统的磁盘使用情况。

```bash
df -h

# 查看索引节点（inode）
df -i
```

### iotop

```bash
yum install -y iotop
```

### lsof

查看进程打开的文件列表，文件包括普通文件、目录、块设备、动态库、网络套接字等。

```bash
yum install -y lsof

lsof -p ${pid}
```

## 网络相关

### net-tools

```bash
yum install -y net-tools
```

#### ifconfig

```bash
ifconfig [$interface]
```

* RUNNING：同 ip 的 LOWER\_UP。

### iproute

```bash
yum install -y iproute
```

net-tools 的下一代，建议使用这个。

#### ip

```bash
ip -s addr show dev $interface
# -s, -stats：显示一些统计信息
```

* LOWER\_UP：物理网络连通，网卡已经连到交换机或路由器。
* MTU：默认 1500。
* errors：校验错误、帧同步错误等。
* dropped：丢弃的数据包数。如内存不足。
* overruns：超限的数据包数。如网络 IO 过快，Ring Buffer 来不及处理。
* carrier：发生 carrier 错误的数据包数。如双工模式不匹配、物理电缆出问题等。
* collisions：碰撞数据包数。

### netstat

查看套接字、网络栈、网络接口以及路由表的信息。

```bash
netstat -nlp
# -n 显示数字地址和端口，而不是名字
# -p 显示进程信息
# -l 只显示监听的套接字

netstat -s
# -s, 协议的收发汇总情况及错误

netstat -i
# -i, 查看网卡，比如丢包等信息
```

### ss

与 netstat 类似，但是性能更好。

```bash
ss -nplt
# -n 显示数字地址和端口，而不是名字
# -p 显示进程
# -l 只显示监听的套接字
# -t 只显示 tcp 套接字

ss -s
# -s 统计信息
```

接收队列（Recv-Q）和发送队列（Send-Q）通常应该是 0。若不是 0 时，说明有网络包的堆积发生。

若 Socket 处于 Established 状态时：

* Recv-Q：Socket 缓冲区还未被应用程序取走的字节数（接收队列长度）。
* Send-Q：还未被远端主机确认的字节数（发送队列长度）。

若 Socket 处于 Listen 状态：

* Recv-Q：全连接队列的长度。
* Send-Q：全连接队列的最大长度。

{% hint style="info" %}
全连接指完成了三次 TCP 握手。需要被 accept\(\) 系统调用取走。半连接指还未完成 TCP 三次握手，服务器收到客户端 SYN 后，会把连接放到半连接队列中，再向客户端发送 SYN+ACK。
{% endhint %}

### ethtool

```bash
ethtool $interface
```

### ping

测试主机连通性和延时。

```bash
ping -c3 $ip
```

### tcpdump

常用的网络抓包工具。基于 libpcap。

```bash
yum install -y tcpdump
```

```bash
tcpdump [option] [filter-expression]

# -i 网卡
tcpdump -i ens33 -n tcp port 80

tcpdump -i any -nn udp port 53 or host 35.190.27.188
# -i any 所有网卡
# -nn 不解析抓包中的域名、协议、端口号
# udp port 53 仅显示 udp 协议的 53 端口
# host 35.190.27.188 只显示特定 IP（包括源地址和目的地址）
```

基本输出格式：`时间戳 协议 源地址.源端口 > 目的地址.目的端口 网络包详细信息`。可以通过选项增加其它字段。

![](../../.gitbook/assets/image%20%28313%29.png)

![](../../.gitbook/assets/image%20%28314%29.png)

### Wireshark

抓包和分析工具，提供图形界面和汇总分析工具。

```bash
yum install -y wireshark
```

tcpdump 通过 -w 选项保持抓包文件，可通过 wireshark 图形界面打开，以更加友好的方式展示。

比如 Statistics -&gt; Flow Graph 可以查看整个 TCP 的流程。推荐书籍 《Wireshark网络分析就这么简单》、《Wireshark网络分析的艺术》

### hping3

可以构造 TCP/IP 协议数据包的工具。

```bash
yum install -y epel-release
yum install -y hping3

brew install hping
```

```bash
# -S tcp 协议的 SYN
# -p 目标端口哦
# -i u100 发送间隔为 100 微秒
hping3 -S -p 80 -i u100 10.93.245.152
```

### traceroute

```bash
yum install -y "traceroute"

traceroute --tcp -p 80 -n baidu.com
# -n 不对结果中的 IP 进行反向域名解析
```

### bind-utils

```bash
yum install -y bind-utils
```

#### nslookup

```bash
nslookup $domain
# -debug 开启调试模式

# 通过 ip 查找域名
nslookup -type=PTR 35.190.27.188 8.8.8.8
```

#### dig

```bash
dig +trace +nodnssec baidu.com
# +trace 开启跟踪查询
# +nodnssec 禁止 DNS 安全扩展
```

dig trace 的输出主要有四部分：

* 一：查到根域名服务器 \(.\) 的 NS 记录。
* 二：从 NS 中选一个，查询顶级域名 com. 的 NS 记录。
* 三：从 com. 的 NS 记录中选一个，查找二级域名 baidu.com. 的 NS 记录。
* 四：从 baidu.com. 的 NS 记录中选一个，查询 baidu.com. 的 A 记录。

### dnsmasq

最常用的 DNS 缓存服务器，还可作为 DHCP 服务器使用。

```bash
/etc/init.d/dnsmasq start

echo nameserver 127.0.0.1 > /etc/resolv.conf
```

## 测试相关

### sysbench

多线程基准测试工具。

* --thread: 线程数
* --time: 执行多少秒

```bash
apt-get install sysbench

sysbench --num-threads=8 --max-requests=100000 --test=cpu run

sysbench --test=memory --memory-oper=read --memory-access-mode=seq run
sysbench --test=memory --memory-oper=write --memory-access-mode=seq run
sysbench --test=memory --memory-oper=read --memory-access-mode=rnd run
sysbench --test=memory --memory-oper=write --memory-access-mode=rnd run
```

### stress

系统压力测试工具：`apt install stress`，重要的参数有：

* -i, --io 模拟 N 个进程执行 sync\(\)
* -c,--cpu 模拟 N 个进程执行 sqrt\(\)
* -t,--timeout 执行 N 秒

### ab

HTTP 服务性能测试工具。

```bash
yum install -y httpd-tools

# 并发 10 个请求，总共测试 10000 个请求
$ ab -c 10 -n 10000 http://192.168.0.10:10000/
```

测试结果分为请求汇总、连接时间汇总、请求延时汇总三部分。

### fio

磁盘性能测试工具。

```bash
yum install -y fio


# 随机读
fio -name=randread -direct=1 -iodepth=64 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 随机写
fio -name=randwrite -direct=1 -iodepth=64 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 顺序读
fio -name=read -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 顺序写
fio -name=write -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb 

# direct，是否跳过系统缓存
# iodepth，使用异步 io 时，同时发出 io 请求的上限
# rw，io 模式
# ioengine，io 引擎
# bs，io 大小
# filename，文件或磁盘路径
```

* slat，从 io 提交到实际执行 io 的时长。
* clat，从 io 提交到完成 io 的时长。
* lat，fio 创建 io 到 io 完成的时长。
* bw，吞吐量。
* iops，每秒 io 次数。

{% hint style="info" %}
对于同步 IO，clat = 0；对于异步 IO，lat ≈ slat + clat。
{% endhint %}

```bash
### fio 还支持重放

# 使用blktrace跟踪磁盘I/O，注意指定应用程序正在操作的磁盘
$ blktrace /dev/sdb

# 查看blktrace记录的结果
$ ls
sdb.blktrace.0  sdb.blktrace.1

# 将结果转化为二进制文件
$ blkparse sdb -d sdb.bin

# 使用fio重放日志
$ fio --name=replay --filename=/dev/sdb --direct=1 --read_iolog=sdb.bin 
```

### iperf

TCP、UDP 性能测试工具。

```bash
yum install -y iperf3

## 服务端（被测试端）启动服务
iperf3 -s -i 1 -p 10000
# -s 启动服务端
# -i 汇报时间间隔
# -p 监听端口

## 客户端运行测试
iperf3 -c 192.168.0.30 -b 1G -t 15 -P 2 -p 10000
# -c 启动客户端，后面为目标服务器 IP
# -b 目标带宽
# -t 测试时间
# -P 并发数
# -p 服务端端口
```

SUM 行就是测试汇总结果。包括测试时间、数据传输量、带宽等。又分为 sender 和 receiver 两行。

### netperf

TCP、UDP 性能测试工具。

### wrk

网络性能测试工具。

```bash
wrk --latency -c 100 -t 2 --timeout 2 http://192.168.0.30/
```

