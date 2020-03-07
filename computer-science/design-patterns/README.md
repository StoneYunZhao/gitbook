# Design Patterns

**设计模式**（design pattern）是对软件设计中普遍存在（反复出现）的各种问题，所提出的解决方案。

有四位在软件开发领域很有名的人出版了一本书《设计模式：可复用面向对象软件的基础》，一共收录了 23 种设计模式。这四个人常称为 GoF（Gang of Four）。23 种设计模式分类：

* 创建型
  * [简单工厂](creational/factory.md#jian-dan-gong-chang)（不属于 GoF）
  * [工厂方法](creational/factory.md#gong-chang-fang-fa)
  * [抽象工厂](creational/factory.md#chou-xiang-gong-chang)
  * [生成器](creational/builder.md)
  * [原型](creational/prototype.md)
  * [单例](creational/singleton.md)
* 结构型
  * [适配器](structural/adapter.md)
  * [桥接](structural/bridge.md)
  * [组合](structural/composite.md)
  * [装饰](structural/decorator.md)
  * [外观](structural/facade.md)
  * [享元](structural/flyweight.md)
  * [代理](structural/proxy.md)
* 行为型
  * [责任链](behavioral/chain-of-responsibility.md)
  * 命令
  * 解释器
  * [迭代器](behavioral/iterator.md)
  * 中介者
  * 备忘录
  * [观察者](behavioral/observer.md)
  * 状态
  * 策略
  * 模板方法
  * 访问者

学习设计模式需要先了解[ UML](uml.md) 和一些[基本原则](principle.md)。

当然除了 GoF 收录的 23 种设计模式，还有许多其它设计模式。比如[并发领域的一些设计模式](../../java/concurrency/concurrency-design-patterns/)。

学习设计模式有很好好处：

* 应对面试
* 告别烂代码
* 提高复杂代码的设计和开发能力
* 读源码、学框架事半功倍
* 为职场发展做铺垫

平时我们评价代码质量的好坏有很多词语，我们最常用的标准有：

* 可维护性（maintainability）
* 可读性（readability）
* 可扩展性（extensibility）
* 灵活性（flexibility）
* 简洁性（simplicity）
* 可复用性（reusability）
* 可测试性（testability）

**面向对象**由于具有丰富的特性，所以是设计原则、设计模式的编码实现基础。  
**设计原则**是知道我们代码设计的一些经验总结。  
**设计模式**是针对软件开发中经常遇到的一些设计问题，总结出来的一套解决方案或者设计思路。比设计原则更加具体、可执行。  
**编程规范**主要解决的是代码的可读性问题。  
**重构**作为保持代码质量不下降的有效手段，利用的就是面向对象、设计原则、设计模式、编码规范这些理论。

![](../../.gitbook/assets/image%20%28171%29.png)

