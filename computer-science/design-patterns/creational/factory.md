# Factory

## 简单工厂

### 介绍

由一个工厂对象决定创建出哪一种产品类的实例。不属于 GoF（**或者把简单工厂看做工厂方法的一种特例**）。

**适用场景**：工厂类负责创建的对象种类比较少。

**优点**：

* 客户端可以免除直接创建产品对象的责任，而仅仅“消费”产品。
* 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可。

**缺点**：

* 违背开闭原则。一旦添加新产品就不得不修改工厂逻辑，在产品类型较多时，有可能造成工厂逻辑过于复杂，不利于系统的扩展和维护。
* 若使用静态工厂方法，造成工厂角色无法形成基于继承的等级结构。

### 类图

![](../../../.gitbook/assets/image%20%2876%29.png)

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

![](../../../.gitbook/assets/image%20%2856%29.png)

## 工厂方法

### 介绍

定义：工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法把实例化推迟到子类。

### 类图

![](../../../.gitbook/assets/image%20%2887%29.png)

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

![](../../../.gitbook/assets/image%20%2859%29.png)

### 源码示例

Connection 对应 Factory，Statement 对应 ProductA，PreparedStatement 对应 ProductB。

```java
// java.sql
public interface Connection  extends Wrapper, AutoCloseable {
    Statement createStatement();
    PreparedStatement prepareStatement(String sql);
}
```

## DI 框架

### 与工厂模式的区别

* DI 底层是基于工厂模式的。DI 框架相当于一个大工厂，在启动时或使用时创建对象。
* 工厂模式仅负责某个类或某组相关类的创建，而 DI 负责整个应用中的类创建。
* DI 除了负责对象创建，还有其它能力，如配置解析等。

### 核心功能

* 配置解析。工厂类创建的对象是事先确定好的，相当于硬编码，而 DI 作为框架，事先并不知道需要创建哪些类，所以需要一中方式，将需要创建的类写在配置中。
* 对象创建。利用反射在程序运行中动态地创建类。
* 生命周期管理。比如 spring 的 scope（prototype，singleton）；lazy-init；init-method，destroy-method 等。

### 简单实现

参考 [Spring 的容器](../../../java/spring.md#rong-qi)。

**ApplicationContext**

```java
public interface ApplicationContext {
  Object getBean(String beanId);
}

// 负责组装 BeansFactory 和 BeanConfigParser
public class ClassPathXmlApplicationContext implements ApplicationContext {
  private BeansFactory beansFactory;
  private BeanConfigParser beanConfigParser;

  public ClassPathXmlApplicationContext(String configLocation) {
    this.beansFactory = new BeansFactory();
    this.beanConfigParser = new XmlBeanConfigParser();
    loadBeanDefinitions(configLocation);
  }

  private void loadBeanDefinitions(String configLocation) {
    InputStream in = this.getClass().getResourceAsStream("/" + configLocation);
    List<BeanDefinition> beanDefinitions = beanConfigParser.parse(in);
    beansFactory.addBeanDefinitions(beanDefinitions);
  }

  @Override
  public Object getBean(String beanId) {
    return beansFactory.getBean(beanId);
  }
}
```

**BeanConfigParser**

```java
public interface BeanConfigParser {
  List<BeanDefinition> parse(InputStream inputStream);
}

public class XmlBeanConfigParser implements BeanConfigParser {
  @Override
  public List<BeanDefinition> parse(InputStream inputStream) {
    // TODO:...
  }
}
```

**BeanDefinition**

```java
public class BeanDefinition {
  private String id;
  private String className;
  private List<ConstructorArg> constructorArgs = new ArrayList<>();
  private Scope scope = Scope.SINGLETON;
  private boolean lazyInit = false;
  // 省略必要的getter/setter/constructors
 
  public boolean isSingleton() {
    return scope.equals(Scope.SINGLETON);
  }


  public static enum Scope {
    SINGLETON,
    PROTOTYPE
  }
  
  public static class ConstructorArg {
    private boolean isRef;
    private Class type;
    private Object arg;
    // 省略必要的getter/setter/constructors
  }
}
```

**BeansFactory**

```java
public class BeansFactory {
  private ConcurrentHashMap<String, Object> singletonObjects = new ConcurrentHashMap<>();
  private ConcurrentHashMap<String, BeanDefinition> beanDefinitions = new ConcurrentHashMap<>();

  public void addBeanDefinitions(List<BeanDefinition> beanDefinitionList) {
    for (BeanDefinition beanDefinition : beanDefinitionList) {
      this.beanDefinitions.putIfAbsent(beanDefinition.getId(), beanDefinition);
    }

    for (BeanDefinition beanDefinition : beanDefinitionList) {
      if (beanDefinition.isLazyInit() == false && beanDefinition.isSingleton()) {
        createBean(beanDefinition);
      }
    }
  }

  public Object getBean(String beanId) {
    BeanDefinition beanDefinition = beanDefinitions.get(beanId);
    if (beanDefinition == null) {
      throw new NoSuchBeanDefinitionException("Bean is not defined: " + beanId);
    }
    return createBean(beanDefinition);
  }

  @VisibleForTesting
  protected Object createBean(BeanDefinition beanDefinition) {
    // TODO:...
  }
}
```

## 总结

* 所有的工厂都是用来封装对象的创建。
* 工厂方法使用继承；抽象工厂使用组合。
* 所有的工厂模式都是通过减少应用程序和具体类之间的依赖促进松耦合。

### 简单工厂与工厂方法

* 若对象的创建比较复杂，不只是简单的 new 就可以，而是要组合其它类对象，做各种初始化操作，那么使用工厂方法更佳。
* 创建对象简单，则使用简单工厂更佳。

