# ThreadLocal

再次重复一遍，并发问题出现的根源是多个线程同时读写同一共享变量。通过 [Immutable](immutable.md) 的方式可以只读不写，还有另一种方式就是：**避免共享**。

比如**局部变量是线程安全的**，因为每个线程都有自己的一份，没有共享，即**线程封闭**。Java 提供 ThreadLocal 也能做到避免共享，即每个线程都有自己的副本。

## 使用方法

我们知道 SimpleDateFormat 不是线程安全的，如果要在并发场景下使用它，有一个办法就是通过 ThreadLocal。

如果是同一个线程调用 get 方法，则会返回相同的实例，若是不同的线程调用 get 方法，则会返回不同的实例。

```java
// java.lang.Thread.java
public class SafeDateFormat {
    private static final ThreadLocal<SimpleDateFormat> th = 
            ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    public static DateFormat get() {
        return th.get();
    }
}
```

## 原理实现

需要实现 ThreadLocal 的逻辑，我们有两个思路：

* 在 ThreadLocal 中持有一个 Map，key 是线程 ID，value 是我们需要的值。
* 在 Thread 中持有一个 Map，key 是 ThreadLocal，value 是我们需要的值。

Java 使用的是第二种思路，理由是：

* **数据亲缘性**：ThreadLocal 只是一个工具类，内部不应该存储与线程相关的数据，与线程相关的数据应该放在 Thread 里面。
* **不容易产生内存泄漏**：ThreadLocal 的生命周期往往比线程长，若数据放在 ThreadLocal中，就算 Thread 已经销毁了，相关的数据也不会被回收；若数据放在 Thread 中，Thread 对象可以被回收，那么 ThreadLocalMap 也可以被回收了。

下面是 Java 1.11 精简后的代码：

{% tabs %}
{% tab title="" %}
```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    
    public Thread() {
        // inheritableThreadLocals 的逻辑
        // 若创建线程的线程存在 inheritableThreadLocals，则通过复制继承
        Thread parent = currentThread();
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="" %}
```java
// java.lang.ThreadLocal.java
public class ThreadLocal<T> {
    static class ThreadLocalMap {
    
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;
            
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        
        private Entry[] table;
        
        private Entry getEntry(ThreadLocal<?> key) {}
        private void set(ThreadLocal<?> key, Object value) {}
    }
    
    public T get() {
        ThreadLocalMap map = getMap(Thread.currentThread());
        ...
    }
    
    public void set(T value) {
        ThreadLocalMap map = getMap(Thread.currentThread());
        ...
    }
    
    public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null) {
             m.remove(this);
         }
     }
    
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
}
```
{% endtab %}
{% endtabs %}

## 注意事项

### 内存泄漏

上文提到 Java 设计成 Thread 持有 ThreadLocal 数据的原因之一是不容易产生内存泄漏，但是在某些情况下还是会有内存泄漏的问题。

比如在线程池中使用 ThreadLocal。线程池中的线程存活时间很长，往往与程序的生命周期一致，所以 Thread 持有的 ThreadLocalMap 一直不会被回收。尽管 ThreadLocalMap 中的 Entry 对 ThreadLocal 是弱引用的，只要 ThreadLocal 自己的生命周期结束就可以被回收，但是 Entry 中的 Value 是强引用，所以尽管 Value 的生命周期结束了，也不会被回收，导致内存泄漏。

**解决方案**：手动释放，比如通过 try-finally。

```java
ThreadLocal tl;

tl.set(obj);
try {
    ...
} finally {
    tl.remove();
}
```

### InheritableThreadLocal

ThreadLocal 创建的线程变量，子线程是无法继承的，也就是说你在线程 A 中创建了 ThreadLocal V，然后又在线程 A 中创建了线程 B，但是在线程 B 中无法访问到 V。

Java 提供了 InheritableThreadLocal 来支持这种需求。

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    protected T childValue(T parentValue) {
        return parentValue;
    }

    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

