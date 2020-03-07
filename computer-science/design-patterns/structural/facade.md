# Facade

GoF 中的定义：Provide a unified interface to a set of interfaces in a subsystem. Facade Pattern defines a higher-level interface that makes the subsystem easier to use. 门面模式为子系统提供一组统一的接口，定义一组高层接口让子系统更易用。

适用场景：

* 解决易用性问题。比如 Linux 的 Shell 命令可以看做是门面，封装了系统调用。
* 解决性能问题。多个接口封装成一个接口，减少调用次数，尤其是远程调用。
* 解决分布式事务问题。

与适配器模式的区别？

* 适配器是做接口转换，解决的是原接口和目标接口不匹配的问题。 
* 门面模式做接口整合，解决的是多接口调用带来的问题。

