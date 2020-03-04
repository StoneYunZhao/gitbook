# Decorator

装饰器模式通过组合给原始类添加增强功能，并且支持多个装饰器嵌套使用。如下所示，装饰器类与原始类需要继承相同的接口，典型的例子是 JDK 的 InputStream：

```java
public interface IA {
  void f();
}

public class A impelements IA {
  public void f() { //... }
}

public class ADecorator impements IA {
  private IA a;
  public ADecorator(IA a) {
    this.a = a;
  }
  
  public void f() {
    // 功能增强代码
    a.f();
    // 功能增强代码
  }
}
```

{% hint style="info" %}
可以看出，代理模式与装饰器模式的代码结构非常类似。区别是代理类附加的是跟原始类无关的功能，装饰器类附加的是跟原始类相关的功能。
{% endhint %}

