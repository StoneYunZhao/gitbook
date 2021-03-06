# UML

统一建模语言（Unified Modeling Language）是一种用于说明、可视化、构建和编写一个正在开发的、面向对象的、软件密集系统的制品的开放方法。UML 2.2 定义了 14 种图：

* 结构性图：
  * 静态图
    * [**类图**](uml.md#lei-tu)\*\*\*\*
    * 对象图
    * 包图
  * 实现图
    * 组件图
    * 部署图
  * 剖面图
  * 复合结构图
* 行为式图：
  * 活动图
  * 状态图
  * 用例图
  * 交互性图
    * 通信图
    * 交互概述图
    * 时序图
    * 时间图

## 类图

描述了系统的类集合，类的属性和类之间的关系。

* 最上面是类名称，抽象类采用斜体表示，接口在上面添加 `<<interface>>` 表示
* 中间部分包含类的属性，类型在后面加 `:` 表示
* 底部部分包含类的方法，抽象方法采用斜体表示，返回值类型在后面加 `:` 表示

指定一个类成员（即任何属性或方法）的可见性有下列符号，必须摆在各成员的名字之前：

```text
+    公共
-    私有
#    保护（即对子类可见）
~    包（即对包内其它成员可见）
/    推导（即由其他属性推导得出，不需要直接给定其值）
_    静态（给整行加下划线，和访问修饰符同时存在）
```

{% hint style="info" %}
箭头方向一定是指向知道对方信息的一方。由于父类不知道子类的存在，但是子类知道父类的存在，所以是子类指向父类。
{% endhint %}

### 泛化（Generalization）

**is-a** 的关系，通常在程序里面泛化表现为继承于**非抽象类**。子类可视为其父类的特例。用带空心三角形箭头的实线表示。

```java
public class A { ... }
public class B extends A { ... }
```

### 实现（Realization）

一个 class 类实现 interface 接口（可以是多个）的功能，通常程序里面实现关系表现为继承**抽象类**。在 Java 中此类关系通过关键字 implements 明确标识。用带空心三角形箭头的虚线表示。

```java
public interface A {...}
public class B implements A { ... }
```

### 依赖（Dependency）

**use-a** 的关系，可以简单的理解为一个类 A 使用到了另一个类 B。被依赖的对象只是作为一种工具在使用，而并不持有对它的引用。表现在代码层面，**类 B 作为参数被类 A在 某个 method（方法）中使用**。用带燕尾箭头的虚线表示。

```java
// 聚合
public class A {
  private B b;
  public A(B b) {
    this.b = b;
  }
}
// 组合
public class A {
  private B b;
  public A() {
    this.b = new B();
  }
}
// 使用
public class A {
  public void func(B b) { ... }
}
```

### 关联（Association）

**has-a** 的关系，两个类之间、或类与接口之间一种强依赖关系，是一种长期的稳定的关系。在生命期问题上没有任何约定。被关联的对象还可以再被别的对象关联。在代码层面上，**被关联类以类属性的形式出现在关联类中**，也可能是关联类引用了一个类型为被关联类的全局变量。通常用一条实线表示，如果需要标明方向可以添加燕尾箭头。

```java
// 聚合
public class A {
  private B b;
  public A(B b) {
    this.b = b;
  }
}

// 组合
public class A {
  private B b;
  public A() {
    this.b = new B();
  }
}
```

### 聚合（Aggregate）

**owns-a** 的关系，整体与部分的一类特殊的关联关系。成分类可以不依靠聚合类而单独存在，可以具有各自的生命周期，部分可以属于多个整体对象，也可以为多个整体对象共享。图形以空心的菱形箭尾与实线来表示。

```java
// 销毁 A 不影响 B，B 可单独存在，如课程与学生
public class A {
  private B b;
  public A(B b) {
    this.b = b;
  }
}
```

### 组合（Composition）

**is a part of** 的关系，成分类必须依靠合成类而存在，整体与部分是不可分的，整体的生命周期结束也就意味着部分的生命周期结束。合成类别完全拥有成分类别，负责创建、销毁成分类别。图形以实心的菱形箭尾与实线表示。

```java
// B 不可单独存在，如鸟与翅膀
public class A {
  private B b;
  public A() {
    this.b = new B();
  }
}
```

![](../../.gitbook/assets/image%20%28240%29.png)

