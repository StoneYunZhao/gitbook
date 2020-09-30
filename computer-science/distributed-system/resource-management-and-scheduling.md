# Resource Management

## 集中式结构

一台或多台服务器组成中央服务器组，系统内所有的业务先由中央服务器处理。多个服务节点与中央服务器组相连，将自己的信息汇报给中央服务器。

优点：部署简单。

![](../../.gitbook/assets/image%20%28274%29.png)

### Borg

![](../../.gitbook/assets/image%20%28273%29.png)

### Kubernetes

![](../../.gitbook/assets/image%20%28275%29.png)

### Mesos

![](../../.gitbook/assets/image%20%28272%29.png)

## 非集中式结构

集中式结构存在单点性能瓶颈和单点故障等问题。非集中式可以做到超大规模。

以 Akka、Redis、Cassandra 为例。

![](../../.gitbook/assets/image%20%28281%29.png)

