# Adapter

适配器模式将不兼容的接口转换为兼容的接口。

实现方式有类适配器和对象适配器两种，类适配器采用继承，对象适配器采用组合。两种实现有各自的场景：

* 若 Adaptee 接口不多，两种都可以。
* 若 Adaptee 接口较多，且与 ITarget 大部分相同，用类适配器比较合适。
* 若 Adaptee 接口较多，且与 ITarget 大部分不同，则用对象适配器合适。

```java
// 需要转换成的接口定义
public interface ITarget {
  void f1();
  void fc();
}

// 不兼容 ITarget 接口定义的接口
public class Adaptee {
  public void fa() { //... }
  public void fc() { //... }
}

// 类适配器: 基于继承
public class Adaptor extends Adaptee implements ITarget {
  public void f1() {
    super.fa();
  }
  
  // fc()不需要实现，直接继承自Adaptee，这是跟对象适配器最大的不同点
}

// 对象适配器：基于组合
public class Adaptor implements ITarget {
  private Adaptee adaptee;
  
  public Adaptor(Adaptee adaptee) {
    this.adaptee = adaptee;
  }
  
  public void f1() {
    adaptee.fa(); //委托给Adaptee
  }
  
  public void fc() {
    adaptee.fc();
  }
}
```

适配器模式的应用场景：

* 封装有缺陷的接口设计。比如依赖的外部系统的接口有缺陷（比如包含大量静态方法），我们可以对接口进行二次封装，抽象出更好的接口。
* 统一多个类的接口设计。系统若依赖多个外部系统，通过适配器模式，将他们的接口适配为统一个接口定义。比如依赖多个敏感词过滤器，需要轮流使用。
* 替换依赖的外部系统。比如阿里云的 oss 和华为云的 obs。
* 兼容老版本接口。比如 JDK1.0 有Enumeration 类，JDK2.0 开发更好的 Iterator 类，此时不能直接删除 Enumeration 类，因为项目中有很多类用到了这个，我们可以把 Enumeration 的实现改为 iterator。



