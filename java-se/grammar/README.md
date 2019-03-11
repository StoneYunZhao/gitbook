# Grammar

## 访问修饰符

Java 访问修饰符有四个，权限由小到大依次为：

* **private：**可以修饰数据成员、构造方法、方法成员，不可修饰类（外部类或顶层类，不包括内部类）。被 private 修饰后，只能在定义它们的类中访问。
* **default：**类、数据成员、构造方法、方法成员都能使用，即不写修饰符。默认权限即同包权限，只能在定义它们的类中以及同包的类中使用。
* **protected：**可以修饰数据成员、构造方法、方法成员，不可修饰类（外部类或顶层类，不包括内部类）。可以被定义它们的类、同包类、子类访问。
* **public：**类、数据成员、构造方法、方法成员都能使用。可以在任何地方访问。

## 变量

### 类变量

1. 有static修饰。
2. 存储在方法区。

### 成员变量

1. 存储在堆中。
2. 修饰符有：public、protected、private、final、volatile、transient。

### 局部变量

1. 存储在栈中。
2. 在方法体、代码块、形参中。
3. 修饰符有：final。

## 方法

### 重写

#### 哪些方法不能被重写

* final方法。
* static方法。
* private方法。

## 内部类

* **顶层类**（Top-level）：类的定义代码不嵌套在其它类中。
* 内部类（Inner Class）：
  * **静态内部类**（Static Inner Class）：
    * 不依赖外部类实例而被实例化。
    * 不能访问外部类的普通成员，只能访问外部类的静态成员和静态方法（包括私有）。
  * **成员内部类**（Member Inner Class）：
    * 可以自由引用外部类的属性和方法，无论是静态还是非静态。
  * **局部内部类**（Local Inner Class）：
    * 定义在一个代码块中的类，作用范围为所在代码块。
    * 与局部变量一致，不能被public、protected、private、static 修饰。
    * 只能访问final 类型的局部变量。
  * **匿名内部类**（Anonymous Inner Class）：
    * 没有构造方法。
    * 必须继承（extend）其它类或接口。

{% hint style="info" %}
* 非静态内部类只能在外部类实例化后，才能被实例化；**不能有静态成员**。
* 静态内部类、成员内部类可以看做是类的静态成员和非静态成员，支持的访问修饰符一致。
* 局部内部类可以看做局部变量，支持的访问修饰符一致。
* 外部类可以看做是顶层类，所以与文件名一致的顶层类可以被 public 修饰，而其它则不行。
{% endhint %}

```java
/**
 * 外部类
 * 可以被 public、abstract、final 修饰
 * 不能被 protect、private、static 修饰
 */
public class Outter {
    // 静态内部类
    // 可以被 public、protected、private、final、abstract 修饰
    static class StaticInner{}

    // 成员内部类
    // 可以被 public、protected、private、final、abstract 修饰
    class MemberInner{} 

    public void f() {
        // 局部内部类
        // 不能被 public、protected、private、static、abstract 修饰
        // 可以被 final 修饰
        class LocalInner{} 
    }

    public static void g() {
        // 局部静态内部类
        // 不能被 public、protected、private、static、abstract 修饰
        // 可以被 final 修饰
        class LocalInner{} 
    }

    public void p() {
        // 匿名内部类
        new Runnable() { 
            @Override
            public void run() {
                System.out.println(1);
            }
        };
    }
}

// 顶层类
// 不能被 public、protected、private、static 修饰
// 可以被 final、abstract 其中之一修饰
class TopLevel{}
```

