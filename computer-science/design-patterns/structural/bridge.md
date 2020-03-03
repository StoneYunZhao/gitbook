# Bridge

桥接模式，也叫桥梁模式。应用场景比较局限，在实际项目中并没有常见，简单了解即可。

**GoF 中的定义**：Decouple an abstraction from it's implementation so that the two can vary independently。将抽象和实现解耦，所以它们可以独立变化。

如下所示，如要缓存 Oracle 的数据库，改为 `oracle.jdbc.driver.OracleDriver`就行。

```java
Class.forName("com.mysql.jdbc.Driver");//加载及注册JDBC驱动程序
String url = "jdbc:mysql://localhost:3306/sample_db?user=root&password=your_password";
Connection con = DriverManager.getConnection(url);
...
```

```java
package com.mysql.jdbc;

public class Driver extends NonRegisteringDriver implements java.sql.Driver {
  static {
    try {
      java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
      throw new RuntimeException("Can't register driver!");
    }
  }
}
```

```java
public class DriverManager {
  static {
    loadInitialDrivers();
  }

  public static synchronized void registerDriver(java.sql.Driver driver) throws SQLException {
    if (driver != null) {
      registeredDrivers.addIfAbsent(new DriverInfo(driver));
    } else {
      throw new NullPointerException();
    }
  }
}
```

{% hint style="info" %}
这里的抽象不是抽象类或接口，而是抽象出来的一套类库，比如 JDBC；实现也不是接口的实现，而是实现的一套类库，比如 MySQL 的 Driver、Oracle 的 Driver。
{% endhint %}

很多书籍、资料中有**其它解释**：一个类存在两个（或多个）独立变化的维度，我们通过组合的方式，让这两个（或多个）维度可以独立进行扩展。类似于[组合由于继承](../principle.md#duo-yong-zu-he-shao-yong-ji-cheng)。

