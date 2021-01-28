# Protocol & Algorithm

## 大纲

![](../../.gitbook/assets/image%20%28330%29.png)

对于一个分布式算法，我们常从拜占庭容错、一致性、性能、可用性四个角度来分析。

![](../../.gitbook/assets/image%20%28328%29.png)

### 拜占庭容错

描述一个不可信场景，除了存在故障，还存在恶意行为。

大部分环境（企业内网）是可信的，系统具有故障容错能力就行了。

在不可信环境中，常见的拜占庭容错算法有 POW、PBFT 等。

### 一致性

* 强一致性\(Strong consistency\)：写操作完成后，任何后续的读操作都能读到更新后的值。包括：
  * 线性一致性\(Linearizability consistency\)、原子一致性：CAP 中的 C 即指这个，如 ZK 的写操作，etcd 的读写都是。
  * 顺序一致性\(Sequential consistency\)：如 ZK 的整体（read+write）
* 弱一致性\(Weak consistency\)：写操作完成后，不能保证后续读操作能读到最新的值。包括：
  * 因果一致性\(Causal consistency\)
  * 最终一致性\(Eventual consitency\)：若某个对象没有写操作了，最终所有后续的读操作都能读到最新的值。

一致性的定义在不同理论中有不同的意义，很容易混淆，待补充。可以参考：

* [http://kaiyuan.me/2018/04/21/consistency-concept/](http://kaiyuan.me/2018/04/21/consistency-concept/)
* [https://segmentfault.com/a/1190000022248118](https://segmentfault.com/a/1190000022248118)

### 可用性

Gossip 只有一个节点也能提供服务；Paxos、ZAB、Raft、Quorum NWR、PBFT、POW 能容忍一定数量的节点故障；2PC、TCC 只有所有节点都健康时才能运行。

### 性能

Gossip 是 AP 系统，性能最高；Paxos、ZAB、Raft 都是领导者模型，写性能受限于领导者，读性能取决于一致性实现；2PC、TCC 需要预留和锁定资源，性能较低。

## 拜占庭将军问题

描述的是最复杂的分布式故障场景，除了存在故障行为，还存在恶意行为。常用的算法有 PBFT 算法和 PoW 算法。

可以通过 2 忠 1 叛来举例。

### 口信消息型

兰伯特论文中 [The Byzantine Generals Problem](https://www.microsoft.com/en-us/research/publication/byzantine-generals-problem/) 提到解法。

算法前提：叛将人数 m 是已知的。而且需要 m+1 轮递归循环。

算法结论：若叛将人数为 m，则总将军数不能少于 3m + 1。

所以 2 忠 1 叛问题，必须再增加一名忠将才能解决。

### 签名消息型

消息特性：

* 消息无法伪造，且对消息的任何更改都能被发现。
* 所有人都能验证消息的真伪。

基于上述特性，兰伯特论文提到，任何伪造消息都能被发现，且无论多少忠将多少叛将，忠将总能达成一致的作战消息，即 n 位将军能容忍 n-2 位叛将。也需要 m+1 轮协商。此问题是解决忠将如何达成共识的问题，不关心共识是什么，比较理论化。

{% hint style="info" %}
消息签名一般通过非对称加密方式实现。比如 A 向 B 发送消息，A 存有私钥，B 存有公钥。A 把消息计算 hash 值\(MD5\)，再通过私钥加密，把消息和加密的 hash 都发送过去，B 通过公钥解密 hash，同时也计算消息的 hash，比较两个 hash 值即可。
{% endhint %}

## CAP 理论



