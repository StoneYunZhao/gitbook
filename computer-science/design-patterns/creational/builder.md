# Builder

Builder 模式，中文为建造者、构建者、生成器模式。

适用场景：一个类的属性很多，避免构造函数参数列表过长。尽管可以用 set 方法解决，但是遇到如下情况，则必须用建造者模式。

* 有些属性是必填的，这些属性又比较多，若放在构造函数则太长。
* 类的属性之间有一定的依赖关系或约束条件。
* 希望创建好的类不可变，创建好后不能再修改内部属性。这样就不能暴露 set 方法。

使用建造者模式，可以避免对象存在无效状态。比如下面代码，若使用建造者模式，可以避免无效状态。

```java
Rectangle r = new Rectange(); // r is invalid
r.setWidth(2); // r is invalid
r.setHeight(3); // r is valid
```

{% hint style="info" %}
工厂模式是用于创建不同但是相关类型的对象；而建造者模式是用于创建一种类型复杂的对象。
{% endhint %}

```java
public class ResourcePoolConfig {
  private int maxIdle;
  private int minIdle;

  private ResourcePoolConfig(Builder builder) {
    this.maxIdle = builder.maxIdle;
    this.minIdle = builder.minIdle;
  }

  // 将 Builder 类设计成了 ResourcePoolConfig 的内部类。
  // 也可以设计成独立的非内部类 ResourcePoolConfigBuilder。
  public static class Builder {
    private static final int DEFAULT_MAX_IDLE = 8;
    private static final int DEFAULT_MIN_IDLE = 0;

    private int maxIdle = DEFAULT_MAX_IDLE;
    private int minIdle = DEFAULT_MIN_IDLE;

    public ResourcePoolConfig build() {
      // 校验逻辑放到这里来做，包括必填项校验、依赖关系校验、约束条件校验等
      if (minIdle > maxIdle) {
        throw new IllegalArgumentException("...");
      }

      return new ResourcePoolConfig(this);
    }

    public Builder setMaxIdle(int maxIdle) {
      if (maxIdle < 0) {
        throw new IllegalArgumentException("...");
      }
      this.maxIdle = maxIdle;
      return this;
    }

    public Builder setMinIdle(int minIdle) {
      if (minIdle < 0) {
        throw new IllegalArgumentException("...");
      }
      this.minIdle = minIdle;
      return this;
    }
  }
}

// 这段代码会抛出IllegalArgumentException，因为minIdle>maxIdle
ResourcePoolConfig config = new ResourcePoolConfig.Builder()
        .setMaxIdle(10)
        .setMinIdle(12)
        .build();
```

