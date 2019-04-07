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

每个对象锁都对应一个等待队列和一个同步队列，详情见下一节。

![&#x7B49;&#x5F85;&#x961F;&#x5217;&#x4E0E;&#x540C;&#x6B65;&#x961F;&#x5217;](../../.gitbook/assets/image%20%2827%29.png)

![&#x7EBF;&#x7A0B;&#x72B6;&#x6001;&#x7684;&#x53D8;&#x5316;](../../.gitbook/assets/image%20%2828%29.png)

1. Thread.sleep\(long millis\)，一定是当前线程调用此方法，当前线程进入TIMED\_WAITING状态，但不释放对象锁，millis后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。 
2. Thread.yield\(\)，一定是当前线程调用此方法，当前线程放弃获取的CPU时间片，但不释放锁资源，由运行状态变为就绪状态，让OS再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证yield\(\)达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield\(\)不会导致阻塞。该方法与sleep\(\)类似，只是不能由用户指定暂停多长时间。 
3. thread.join\(\)/thread.join\(long millis\)，当前线程里调用其它线程t的join方法，当前线程进入WAITING/TIMED\_WAITING状态，当前线程不会释放已经持有的对象锁。线程t执行完毕或者millis时间到，当前线程一般情况下进入RUNNABLE状态，也有可能进入BLOCKED状态（因为join是基于wait实现的）。 
4. obj.wait\(\)，当前线程调用对象的wait\(\)方法，当前线程释放对象锁，进入等待队列。依靠notify\(\)/notifyAll\(\)唤醒或者wait\(long timeout\) timeout时间到自动唤醒。
5. obj.notify\(\)唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll\(\)唤醒在此对象监视器上等待的所有线程。
6. LockSupport.park\(\)/LockSupport.parkNanos\(long nanos\),LockSupport.parkUntil\(long deadlines\), 当前线程进入WAITING/TIMED\_WAITING状态。对比wait方法,不需要获得锁就可以让线程进入WAITING/TIMED\_WAITING状态，需要通过LockSupport.unpark\(Thread thread\)唤醒。

* [https://blog.csdn.net/pange1991/article/details/53860651](https://blog.csdn.net/pange1991/article/details/53860651)
* [https://www.zhihu.com/question/56494969](https://www.zhihu.com/question/56494969)





