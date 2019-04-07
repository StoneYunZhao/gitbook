# Thread State

`java.lang.Thread.State` 定义了 Java 线程的6种状态。

```java
public class Thread implements Runnable {
     /**
     * These states are virtual machine states which do not reflect
     * any operating system thread states.
     */
    public enum State {
        NEW,
        /**
         * A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
    }
}
```

* **NEW**：new 了一个 Thread 对象，但未调用 `start()`方法。
* **RUNNABLE**：调用了 Thread 的 `start()`方法后进入的状态，可以被 CPU 调度。
* **BLOCKED**：线程阻塞在进入`synchronized`关键字修饰的方法或代码块时的状态。
* **WAITING**：不会被分配 CPU 执行时间，它们要等待被**显式**地唤醒，否则会处于无限期等待的状态。
* **TIMED\_WAITING**：不会被分配 CPU 执行时间，需要被**显示**地唤醒，或达到一定时间后自动唤醒。
* **TERMINATED**：线程已经执行完毕，不可逆转。若再调用`start()`方法，会抛出`java.lang.IllegalThreadStateException`异常。

{% hint style="danger" %}
1. 有人觉得 Java 线程状态中还少了个 running 状态，但这其实是把两个不同层面的状态混淆了。
2.  Java 线程没有 running 的概念，它的 RUNNABLE 包含了 操作系统的 ready 和 running 两种状态。
3. 注意看 State 的注释，Java 线程状态是虚拟机层面的，不会反映操作系统的线程状态。
4. Java 不需要 running 状态是因为没有必要去区分，操作系统的 CPU 时间片段一般很小，10-20ms 左右，Java 去区分是没有意义的。
{% endhint %}

