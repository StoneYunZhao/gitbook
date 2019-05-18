# Basic

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
* 构造方法。

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

## 继承

* 子类可以继承父类的非私有方法和属性。
* Java 只能单一继承。

### 抽象类

* 如果一个类中有一个方法被 abstract 修饰，那么这个类是抽象类，必须被 abstract 修饰。
* abstract 只能修饰类的方法，不能修饰变量。
* 被 abstract 修饰的方法，不能被 private、static、synchronized、native 修饰。
* 被 abstract 修饰的方法，实现后必须是相同或更低的访问级别。

### 接口

* 接口只能被 public 和 abstract 修饰，或者没有修饰符。当没有修饰符，只能被同包使用。接口默认被 abstract 修饰的，所以添不添加 abstract 修饰符都一样。
* 接口方法默认被 public、abstract 修饰，所以有没有上述两个修饰符都一样。
* 当接口方法被 default\(JDK1.8增加\)修饰时，表示有默认实现，需要用`{}`来实现 body。
* 当接口方法被 static\(JDK1.8增加\)修饰时，只能用接口类（不包括子类）访问方法，不能用接口实例访问。
* 接口中的成员变量默认是 public、static、final 的，所以有没有这三个修饰符都一样。必须赋初值。

### 抽象类与接口

* 接口不能有私有变量和方法，而抽象类可以。
* 接口可以看做是一种特殊的抽象类。
* 一个类可以实现多个接口，但是只能继承一个抽象类。
* 接口表示 has-a 的关系，而抽象类表示 is-a 的关系。
* 接口可以继承接口，抽象类可以实现接口，抽象类可以继承实体类。

## Object 的所有方法

```java
public class Object {
    private static native void registerNatives();
    static {
        registerNatives();
    }
    
    // 经常被覆盖的方法
    public boolean equals(Object obj) {
        return (this == obj);
    }
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    
    // native 方法
    public native int hashCode();
    protected native Object clone() throws CloneNotSupportedException;
    
    // final 方法
    public final native Class<?> getClass();
    public final native void notify();
    public final native void notifyAll();
    public final native void wait(long timeout) throws InterruptedException;
    public final void wait(long timeout, int nanos) throws InterruptedException;
    public final void wait() throws InterruptedException {
        wait(0);
    }
    
    // 不建议使用的方法
    protected void finalize() throws Throwable { }
}
```

### `getClass()`

`final` 方法不允许重写，返回**运行时**的 Class 对象。

### `hashCode()`

默认实现是将对象的内部地址转换成一个整数。可以被覆盖，覆盖的约定如下：

* 若对象没有被改变，则无论调用多少次 `hashCode()`，都会返回相同的整数值。
* 没有必要在不同的程序中保持相同的值。
* 若 `equals()`返回 `true`，则 `hashCode()`必须相等。
* 若 `equals()`返回 `false`，`hashCode()`可以相同；但是尽量不相同，可以提高 hash 表的性能。

### `equals()`

默认实现是比较两个对象的地址是否相等。可以被覆盖，约定如下：

* 自反性，`x.equals(x) == true`。
* 对称性，若`x.equals(y) == true`，则`y.equals(x) == true`。
* 传递性，若`x.equals(y) == true && y.equals(z) == true`，则`x.equals(z) == true`
* `x.equals(null) == false`。

### `clone()`

由于 `Object` 本身没有实现 `Cloneable` 接口，所以不重写 `clone()`方法会抛出异常。

返回对象的**浅拷贝**。

### `wait()`、`wait(long)`、`wait(long, int)`

* 让当前线程等待。
* 当前线程必须是调用`wait()`方法对象的监视器所有者，否则会发生`IllegalMonitorStateException`异常。
* `wait()`方法会将当前线程放置在对象的等待集中，并让当前线程放弃该对象监视器的所有权，即**放弃了锁**。
* 会被以下事件唤醒：
  * 其它线程调用此对象的`notify()`方法，并恰巧此线程被选中。
  * 其它线程调用此对象的`notifyAll()`方法。
  * 其它线程调用 `Thread.interrupt()`方法中断此线程。
  * 若设置了超时时间，时间超过后。
* 被唤醒后，线程在等待集中被移除，以常规方式与其它线程竞争，获取该对象的监视器所有权，一旦获得，则 `wait()`方法返回。

{% hint style="warning" %}
### 注意

在没有被通知、中断或超时的情况下，线程还可以唤醒一个所谓的虚假唤醒 \(spurious wakeup\)。虽然这种情况在实践中很少发生，但是应用程序必须通过以下方式防止其发生。
{% endhint %}

```java
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait(timeout);
        ... // Perform action appropriate to condition
}
```

若要成为某个对象监视器的所有者，可以有以下几种方式：

* 执行对象的同步实例方法。
* 使用 synchronized内置锁。
* 对于 Class 对象，执行该类的同步静态方法。

一个线程只能有一个对象的监视器。所以 wait、notify 方法**只能在同步代码块或同步方法中使用**。

{% hint style="info" %}
`wait()`释放了锁，而 `sleep()`仍然持有锁。
{% endhint %}

### `notify()`、`notifyAll()`

* 唤醒一个或全部在此对象监视器上等待的线程。
* 若是一个，则是随机选取一个。
* 直到当前线程放弃了对象上的锁后，被唤醒的线程才会继续竞争锁。即 notify 方法**不会立即释放锁**。
* 当前线程必须是该对象监视器的所有者，不然会抛出 IllegalMonitorStateException。

wait 和 notify 一般配套使用，见下面的例子：

```java
public class ThreadTest {
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        MyRunnable r = new MyRunnable();
        Thread t = new Thread(r);
        t.start();
        synchronized (r) {
            try {
                System.out.println("main thread 等待t线程执行完");
                r.wait();
                System.out.println("被notity唤醒，得以继续执行");
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
                System.out.println("main thread 本想等待，但被意外打断了");
            }
            System.out.println("线程t执行相加结果" + r.getTotal());
        }
    }
}
 
class MyRunnable implements Runnable {
    private int total;
    
    @Override
    public void run() {
        // TODO Auto-generated method stub
        synchronized (this) {
            System.out.println("Thread name is:" + Thread.currentThread().getName());
            for (int i = 0; i < 10; i++) {
                total += i;
            }
            notify();
            System.out.println("执行notif后同步代码块中依然可以继续执行直至完毕");
        }
        System.out.println("执行notif后且同步代码块外的代码执行时机取决于线程调度");
    }
    
    public int getTotal() {
        return total;
    }
}
```

```text
main thread 等待t线程执行完
Thread name is:Thread-0
执行notif后同步代码块中依然可以继续执行直至完毕
执行notif后且同步代码块外的代码执行时机取决于线程调度
被notity唤醒，得以继续执行
线程t执行相加结果45
```

