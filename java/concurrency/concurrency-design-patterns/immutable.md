# Immutable

## 思路

并发问题出现的根源是多个线程同时读写同一共享变量。若只有读，没有写，则不会存在并发问题。

所以解决并发问题最简单的办法就是**使共享变量只读**。可以使用不变性让对象不可写，即对象创建之后，状态不再发生变化。

## 方案

实现一个不可变类的方法：

* 将 class 声明为 final。
* 所有成员变量声明为 final，并且只允许存在只读方法。
* 对于 getter 或其它可能会返回内部状态的方法，使用 copy-on-write 的方式。
* 构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值。

{% hint style="warning" %}
* 就算对象属性是 final 的，但是若属性是引用类型，那么属性的属性还是可能被修改，所以需要上面的 3、4点。
{% endhint %}

## 案例

Java 中的不可变类有[ String ](../../grammar/data-types.md#string)和 Long、Integer、Double 等基础类的[包装类](../../grammar/data-types.md#bao-zhuang-lei)。

String 有字符串替换操作，看上去与不可变性矛盾，实际上它是通过创建一个新的字符串来达到替换的目的，并没有修改原对象的属性。

由此可见，不可变类需要提供类似修改的功能，方式是创建一个新的不可变对象。

但是这样可能会造成对象创建太多了，可以利用[享元模式](../../../computer-science/design-patterns/structural/flyweight.md)来减少对象的创建。

