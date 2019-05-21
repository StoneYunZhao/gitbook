# Network Protocol

《计算机组成与系统结构》、[《数据结构与算法》](../algorithm/)、《操作系统》、《计算机网络》、《编译原理》是大学计算机的核心课程。

新技术层出不穷，网络协议是你到了 45 岁之后任然有价值的知识。

协议的三要素：

* 语法：一段内容要符合一定的规则和格式。如括号要成对。
* 语义：一段内容要代表某种意义。如数字减去数字是有意义的。
* 顺序。

只有通过网络协议，才能使一大片机器互相协作，共同完成一件事。

| 层级 | 协议 |
| :--- | :--- |
| [应用层](application-layer.md) | [DHCP,](application-layer.md#dhcp) HTTP, HTTPS, RTMP, P2P, DNS, GTP, RPC |
| [传输层](transport-layer.md) | [UDP](transport-layer.md#udp), [TCP](transport-layer.md#tcp) |
| [网络层](network-layer.md) | [ICMP](network-layer.md#icmp), [IP](network-layer.md#ip), [OSPF](network-layer.md#ospf), [BGP](network-layer.md#bgp), IPSec, GRE |
| [链路层](data-link-layer.md) | [ARP](data-link-layer.md#arp), [VLAN](data-link-layer.md#vlan), [STP](data-link-layer.md#stp) |
| [物理层](pysical-layer.md) | [网络跳线](pysical-layer.md#8p-8-c), [集线器](pysical-layer.md#hub) |

只要是在网络上跑的包，都是完整的。可以有下层没有上层，绝对不可能有上层没有下层。比如对于 TCP 协议，无论是三次握手、重试，只要想发出去包，就一定有 IP 和 MAC 层。

