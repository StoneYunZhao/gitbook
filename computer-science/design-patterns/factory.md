# Factory

## 简单工厂

### 介绍

由一个工厂对象决定创建出哪一种产品类的实例。不属于 GoF。

适用场景：工厂类负责创建的对象种类比较少。

优点：

* 客户端可以免除直接创建产品对象的责任，而仅仅“消费”产品。
* 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可。

缺点：

* 一旦添加新产品就不得不修改工厂逻辑，在产品类型较多时，有可能造成工厂逻辑过于复杂，不利于系统的扩展和维护。
* 若使用静态工厂方法，造成工厂角色无法形成基于继承的等级结构。

### 类图

![](../../.gitbook/assets/image%20%2838%29.png)

### 源码示例

在 JDK 中 Calendar 类的 getInstance\(\) 方法使用了简单工厂模式。

```java
// JDK 1.11; java.util
public abstract class Calendar implements Serializable, Cloneable, Comparable<Calendar> {
    public static Calendar getInstance(Locale aLocale) {
        return createCalendar(defaultTimeZone(aLocale), aLocale);
    }

    private static Calendar createCalendar(TimeZone zone, Locale aLocale) {
        CalendarProvider provider =
            LocaleProviderAdapter.getAdapter(CalendarProvider.class, aLocale)
                                 .getCalendarProvider();
        if (provider != null) {
            try {
                return provider.getInstance(zone, aLocale);
            } catch (IllegalArgumentException iae) {}
        }

        Calendar cal = null;

        if (aLocale.hasExtensions()) {
            String caltype = aLocale.getUnicodeLocaleType("ca");
            if (caltype != null) {
                switch (caltype) {
                case "buddhist":
                cal = new BuddhistCalendar(zone, aLocale);
                    break;
                case "japanese":
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                    break;
                case "gregory":
                    cal = new GregorianCalendar(zone, aLocale);
                    break;
                }
            }
        }
        if (cal == null) {
            if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") {
                cal = new BuddhistCalendar(zone, aLocale);
            } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja"
                       && aLocale.getCountry() == "JP") {
                cal = new JapaneseImperialCalendar(zone, aLocale);
            } else {
                cal = new GregorianCalendar(zone, aLocale);
            }
        }
        return cal;
    }
}
```

![](../../.gitbook/assets/image%20%2825%29.png)

## 工厂方法

### 介绍

定义：工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法把实例化推迟到子类。

### 类图

![](../../.gitbook/assets/image%20%2842%29.png)

工厂方法一般是一个框架，即 Creator 类中还有其它方法，这些方法在抽象类中已经实现，而且方法中的逻辑使用了抽象方法 factoryMethod\(String\)。

### 源码示例 `Collection`

Collection 对应 Creator，ArrayList 对应 ConcreteCreator，Iterator 对应 Product，`Itr` 对应 ConcreteProduct。

```java
// java.util

public interface Collection<E> extends Iterable<E> {
    Iterator<E> iterator();
}

public class ArrayList<E> extends AbstractList<E> {
    public Iterator<E> iterator() {
        return new Itr();
    }
    
    private class Itr implements Iterator<E> {}
}
```

### 源码示例 ILogFactory

ILoggerFactory 对应 Creator，LoggerContext 对应 ConcreteCreator，Logger 对应 Product。

```java
// org.slf4j

public interface ILoggerFactory {
  public Logger getLogger(String name);
}

public class LoggerContext extends ContextBase implements ILoggerFactory {

  public final Logger getLogger(final String name) {
    if (Logger.ROOT_LOGGER_NAME.equalsIgnoreCase(name)) {
      return root;
    }

    int i = 0;
    Logger logger = root;
    Logger childLogger = (Logger) loggerCache.get(name);
    if (childLogger != null) {
      return childLogger;
    }
    
    ...
}
```

## 抽象工厂

### 介绍

定义：抽象工厂模式提供一个接口，用于创建相关或者依赖对象的家族，而不需要明确指定具体类。

{% hint style="info" %}
**抽象工厂**的方法经常以**工厂方法**的方式**实现**。
{% endhint %}

### 类图

![](../../.gitbook/assets/image%20%2827%29.png)

### 源码示例

Connection 对应 Factory，Statement 对应 ProductA，PreparedStatement 对应 ProductB。

```java
// java.sql
public interface Connection  extends Wrapper, AutoCloseable {
    Statement createStatement();
    PreparedStatement prepareStatement(String sql);
}
```

## 总结

* 所有的工厂都是用来封装对象的创建。
* 工厂方法使用继承；抽象工厂使用组合。
* 所有的工厂模式都是通过减少应用程序和具体类之间的依赖促进松耦合。

