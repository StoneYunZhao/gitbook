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

![](../../.gitbook/assets/image%20%2896%29.png)

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

![](../../.gitbook/assets/image%20%2822%29.png)

## 工厂方法

## 抽象工厂

