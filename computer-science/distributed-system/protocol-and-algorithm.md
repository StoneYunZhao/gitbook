# Protocol & Algorithm

## 大纲

对于一个分布式算法，我们常从拜占庭容错、一致性、性能、可用性四个角度来分析。

![](../../.gitbook/assets/image%20%28328%29.png)

### 拜占庭容错

描述一个不可信场景，除了存在故障，还存在恶意行为。

大部分环境（企业内网）是可信的，系统具有故障容错能力就行了。

在不可信环境中，常见的拜占庭容错算法有 POW、PBFT 等。

### 一致性

* 强一致性：写操作完成后，任何后续的读操作都能读到更新后的值。又称为线性一致性、
* 弱一致性：写操作完成后，不能保证后续读操作能读到最新的值。
* 最终一致性：若某个对象没有写操作了，最终所有后续的读操作都能读到最新的值。

一致性的定义在不同理论中有不同的意义，很容易混淆，待补充。可以参考：

* [http://kaiyuan.me/2018/04/21/consistency-concept/](http://kaiyuan.me/2018/04/21/consistency-concept/)
* [https://segmentfault.com/a/1190000022248118](https://segmentfault.com/a/1190000022248118)

### 可用性

Gossip 只有可以节点也能提供服务；Paxos、ZAB、Raft、Quorum NWR、PBFT、POW 能容忍一定数量的节点故障；2PC、TCC 只有所有节点都健康时才能运行。

### 性能

Gossip 是 AP 系统，性能最高；Paxos、ZAB、Raft 都是领导者模型，写性能受限于领导者，读性能取决于一致性实现；2PC、TCC 需要预留和锁定资源，性能较低。

![](../../.gitbook/assets/image%20%28329%29.png)

