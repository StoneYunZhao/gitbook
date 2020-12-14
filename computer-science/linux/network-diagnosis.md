# Network Diagnosis

## 原理

首先需要了解[网络协议相关知识](../network-protocol/)，Linux 按照 TCP/IP 模型实现了网络协议栈。网络接口配置的最大传输单元（MTU）规定了 IP 包的大小，Linux 默认为 1500。若超过 MTU，则会在网络层分片。

![](../../.gitbook/assets/image%20%28310%29.png)

网卡发送和接受网络包的基本设备，网卡内核有驱动程序注册到系统。在网络收发过程中，内核通过中断与网卡交互。网卡硬中断只处理最核心的数据读取和发送，协议栈的大部分逻辑都由软中断处理。

### 网络包接收流程

1. 网络帧到达网卡，网卡通过 DMA 把网络包放到收包队列（环形缓冲区）；然后通过硬中断，告诉中断处理程序已收到网络包。
2. 网卡中断处理程序为网络帧分配内核数据结构（sk\_buff），并将其 copy 到 sk\_buff 缓冲区中（双向链表）；然后通过软中断通知内核收到新的网络帧。
3. 内核协议栈从缓冲区取出网络帧，从上到下逐层处理网络帧。
   1. 链路层：检查报文合法性，确认上层协议类型（IPv4、IPv6），去掉帧头帧尾交给网络层。
   2. 取出 IP 头，确认交给上层处理还是转发；若是上层处理，确认上层协议类型（TCP、UDP），去掉 IP 头交给传输层。
   3. 取出 TCP 头或 UDP 头，根据四元组（源 IP、源端口、目的 IP、目的端口）找到对应 Socket，把数据 Copy 到 Socket 的接收缓冲区。
   4. 应用程序使用 Socket 接口，读取新接收到的数据。

![](../../.gitbook/assets/image%20%28312%29.png)

### 网络包发送流程

发送流程与接收相反，见上图的右边部分。

## 性能指标

* **带宽**：链路最大传输速率，bit/s
* **吞吐量**：单位时间内成功传输的数据量，bit/s。吞吐量 / 带宽 = 使用率。
* **延时**：网络请求发出到远端响应所需时间。
* **PPS**：Packet Per Second。通常用于评估网络的转发能力，硬件交换机通常可以接近理论最大值。Linux 服务器易受网络包大小的影响。
* 网络可用性：
* 并发连接数：
* 丢包率：
* 重传率：

可用 ifconfig、ip 查看网络配置；用 netstat、ss 查看套接字、网络栈、网络接口、路由表、协议栈统计信息；sar 查看网络吞吐和PPS 信息；ethtool 查看网络驱动和硬件信息；ping 查看连通性和延时信息。

## C10K 到 C10M

C 是 Client 的缩写。C10K 就是单机同时处理 10k 个请求。解决 **C10K** 问题比较简单的，主要从如下几个方便入手：

* IO 模型优化：异步、非阻塞的方式。详见[多客户端连接](../network-protocol/transport-layer.md#duo-ke-hu-duan-lian-jie)。
* 工作模型优化：
  * 主进程+多个 worker 子进程。如 nginx。注意惊群问题，即多个进程同时被唤醒。nginx 通过在每个 worker 进程中增加全局锁来解决。可用线程替代子进程。另外参考 [reactor 模型](../../java/tuning/programming.md#nio)。
  * 监听相同端口的多进程模型。开启 SO\_REUSEPORT 选项，由内核负责将请求负载均衡到这些监听进程中去。由于内核确保只有一个进程被唤醒，所以避免了惊群问题。

通过 IO 多路复用、请求处理优化等方式，解决 C10K 很简单。epoll 配合线程池，再加上 CPU、内存、网络接口性能优化和容量提升，**C100K** 也容易解决。

但是 **C1000K** 没这么简单，除了使用 epoll，还需要从应用程序到内核、再到 CPU、内存等硬件优化来实现。比如多队列网卡、中断负载均衡、CPU 绑定、RPS/RFS、将网络包的处理卸载到网络设备（TSO/GSO、LRO/GRO、VXLAN OFFLOAD）等。

那么 **C10M** 呢？C1000K 中，各种软件、硬件优化已经做到头了。Linux 内核协议栈做了太多工作，如硬中断、软中断、各层网络协议栈、应用程序，路径太长。所以可以跳过内核协议栈把网络包直接发送到应用程序。常见的实现方式有 DPDK 和 XDP。

### DPDK

用户态网络标准，直接由用户态进程通过轮询的方式来处理网络接收。还有大页、CPU 绑定、内存对齐、流水线并发等机制优化网络包处理效率。

![](../../.gitbook/assets/image%20%28311%29.png)

### XDP

eXpress Data Path，Linux 提供的高性能网络数据路径。允许网络包在进入内核协议栈之前就进行处理。XDP 与 bcc-tools 都是基于 eBPF 机制实现的。XDP 应用程序一般是专业的网络程序，如 IDS（入侵检测系统）、DDoS 防御、cilium 容器网络插件等。

![](../../.gitbook/assets/image%20%28309%29.png)

## 性能测试

Linux 基于 TCP/IP 协议栈，所以评估网络性能需要明确属于协议栈的哪一层。

### 转发性能

网络接口层和网络层主要负责网络包的封装、寻址、路由、发送、接受。这两层中最重要的指标是 PPS，特别是 64B 小包的处理能力。

可用内核自带高性能网络测试工具 pktgen 测试转发性能。但是该命令不能直接找到，它是作为内核线程来运行，需要加载 pktgen 内核模块，再通过 /proc 交互。具体使用方法这里不介绍。

### TCP/UDP 性能

可用 iperf、netperf 测试 TCP、UDP 性能。详见 [iperf 使用介绍](diagnostic-tools.md#iperf)。

### HTTP 性能

可用 ab、webbench 等测试 HTTP 性能。详见 [ab 使用介绍](diagnostic-tools.md#ab)。

### 应用负载性能

使用 iperf、ab 等工具测试了 TCP、HTTP 等性能数据并不等同于应用程序的实际性能。因为实际用户请求大不相同，带有不同的业务逻辑。所以需要模拟用户请求来做测试。可用 wrk、TCPCopy、Jmeter、LoadRunner 等工具实现。

## DDoS

DoS（Denail of Service）拒绝服务请求，利用大量的合理请求占用过多的资源，从而使目标服务无法响应正常请求。

DDoS（Distributed Denail of Service）在 DoS 的基础上，采用分布式架构，利用多台主机同时攻击目标机器。可以分为以下几类：

* 耗尽带宽。
* 耗尽操作系统资源。
* 消耗应用程序的运行资源。

DDoS 只能缓解，不能测地解决。比如购买专业的流量清洗设备、网络防火墙，内核调优、DPDK、XDP 等方式，在应用程序中利用各级缓存、WAF、CDN 等方式。

## NAT

先了解相关[基础知识](../network-protocol/network-layer.md#nat-wang-guan)。Linux 提供 Netfilter 框架可以对数据包修改和过滤。iptables、ip6tables、ebtables 就是基于 Netfilter，提供了易用的命令行接口。

![&#x6570;&#x636E;&#x5305;&#x901A;&#x8FC7; Netfilter &#x65F6;&#x7684;&#x6D41;&#x5411;](../../.gitbook/assets/image%20%28316%29.png)

图中，绿色背景方框表示表（table），用于管理链。分为四种类型：filter、nat、mangle（修改分组元数据）、raw（原始数据包）。白色背景方框表示链（chain），管理具体的 iptables 规则。每个表可以包含多个链，有内置链，也可创建自定义链，如：

* filter 表：内置 INPUT、OUTPUT、FORWARD 链。
* nat 表：内置三种链。
  * PREROUTING：路由判断前所执行的规则，如 DNAT。
  * POSTROUTING：路由判断后所执行的规则，如 SNAT 或 MASQUERADE。
  * OUTPUT：类似 PREROUTING，仅处理本机发送出去的包。

灰色的 conntrack 表示连接跟踪模块，通过内核中的连接跟踪表（Hash），记录网络连接状态，是 iptables 状态过滤（-m state）和 NAT 的实现基础。

```bash
# SNAT

## 配置一个子网
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -j MASQUERADE

## 配置一个具体 IP
iptables -t nat -A POSTROUTING -s 192.168.0.2 -j SNAT --to-source 100.100.100.100

# DNAT
iptables -t nat -A PREROUTING -d 100.100.100.100 -j DNAT --to-destination 192.168.0.2

# 双向地址转换
iptables -t nat -A POSTROUTING -s 192.168.0.2 -j SNAT --to-source 100.100.100.100
iptables -t nat -A PREROUTING -d 100.100.100.100 -j DNAT --to-destination 192.168.0.2

# 需要开启 linux 的 IP 转发功能
sysctl net.ipv4.ip_forward
sysctl -w net.ipv4.ip_forward=1
cat /etc/sysctl.conf | grep ip_forward # 持久化
```

## 总结

### 指标 -&gt; 工具

![](../../.gitbook/assets/image%20%28320%29.png)

### 工具 -&gt; 指标

![](../../.gitbook/assets/image%20%28317%29.png)

### 优化思路

#### 应用程序

* epoll 取代 select 和 poll。
* 使用 AIO
* 主进程 + 多个 worker 子进程的工作模型。
* 监听相同端口的多进程模型。
* 长连接取代短连接。
* 使用缓存降低 IO 次数。
* 使用 proto buffer 等序列化方式压缩网络传输数据量。
* 使用 DNS 缓存等减少 DNS 延迟。

#### 套接字

套接字可以屏蔽 Linux 内核中不同协议的差别，提供统一的访问接口，每个套接字都有读写缓冲区。

* 读缓冲区：缓存远端发来的数据，若满，则不能接受新数据。
* 写缓冲区：缓存要发送出去的数据，若满，则写操作阻塞。

![](../../.gitbook/assets/image%20%28318%29.png)

另外可以优化套接字的配置选项：

* TCP\_NODEPLAY，仅有 Nagle 算法。
* TCP\_CORK，小包聚合成大包一起发送。
* SO\_SNDBUF, SORCVBUF。

#### 传输层

**TCP**

![](../../.gitbook/assets/image%20%28319%29.png)

**UDP**

* 增大缓冲区
* 增大端口号范围
* 根据 MTU 调整 UDP 数据包大小

#### 网络层

网络层负责网络包的封装、寻址、路由，包括 IP、ICMP 等。

路由和转发的角度：

* `net.ipv4.ip_forward`
* `ip.ipv4.ip_default_ttl`
* `ip.ipv4.conf.eth0.rp_filter`

从分片的角度主要调整 MTU 大小。

从 ICMP 的角度：

* `net.ipv4.icmp_echo_ignore_all`
* `net.ipv4.icmp_echo_ignore_broadcasts`

#### 链路层

链路层负责网络包在物理网络中的传输。

从中断的角度：

* 配置网卡硬中断的亲和性。
* 开启 RPS、RFS。

利用网卡的能力：

* TSO、UFO
* GSO
* LRO
* GRO
* RSS
* VXLAN 卸载

网络接口本身：

* 网络接口多队列
* 网络接口缓冲区
* 配置 QoS

