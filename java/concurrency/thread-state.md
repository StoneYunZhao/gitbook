# Thread

## 通用线程状态

线程是操作系统的概念，不同的开发语言都对齐进行了封装，Java 语言里的线程本质上就是操作系统的线程，它们是一一对应的。

通用的线程生命周期有五种状态：**初始状态、可运行状态、运行状态、休眠状态**和**终止状态**。

![](../../.gitbook/assets/image%20%2864%29.png)

* **初始状态：**指的是线程已经被创建，但是还不允许分配 CPU 执行。这个状态属于编程语言特有的，这里的创建仅仅在编程语言中创建，但是没有在操作系统层面创建。
* **可运行状态：**线程可以分配 CPU 执行，真正的操作系统线程已经被创建了。
* **运行状态：**被分配到 CPU 的线程的状态。
* **休眠状态：**运行状态的线程如果调用一个阻塞的 API（例如以阻塞方式读文件）或者等待某个事件（例如条件变量），那么线程的状态就会转换到此状态，同时**释放 CPU** 使用权，休眠状态的线程永远没有机会获得 CPU 使用权。
* **终止状态：**线程执行完或者出现异常。

这五种状态在不同的编程语言里面会有简化，如C 语言的 POSIX Threads 规范，就把初始状态和可运行状态合并了；Java 语言里则把可运行状态和运行状态合并了。

除了合并，也会有细化，Java 语言里就细化了休眠状态。

## Java 线程状态

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

**BLOCKED、WAITING、TIMED\_WAITING 可以理解为线程导致休眠状态的三种原因。**

### Java 线程状态转换

![&#x7EBF;&#x7A0B;&#x72B6;&#x6001;&#x7684;&#x53D8;&#x5316;](../../.gitbook/assets/image%20%2855%29.png)

{% hint style="warning" %}
线程调用阻塞式 API 时，不会进入 BLOCKED 状态，在操作系统层面是休眠状态，但是在 JVM 层面还是 RUNNABLE 状态。因为在 JVM 看来，等待 CPU 使用权（操作系统层面此时处于可执行状态）与等待 I/O（操作系统层面此时处于休眠状态）没有区别，都是在等待某个资源，所以都归入了 RUNNABLE 状态。
{% endhint %}

每个对象锁都对应一个等待队列和一个同步队列，详情见下一节。下图是一个线程状态转换的例子。

![&#x7B49;&#x5F85;&#x961F;&#x5217;&#x4E0E;&#x540C;&#x6B65;&#x961F;&#x5217;](../../.gitbook/assets/image%20%2854%29.png)

### 线程状态转换相关方法

1. **`Thread.sleep(long millis)`**：一定是**当前线程**调用此方法，当前线程进入 TIMED\_WAITING 状态，但**不释放**对象锁，millis 后线程自动苏醒进入就绪状态。是给其它线程执行机会的最佳方式。 
2. **`Thread.yield()`**：一定是**当前线程**调用此方法，当前线程放弃获取的 CPU 时间片，但**不释放**锁资源，由运行状态变为就绪状态，让 OS 再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证 yield\(\) 达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。`Thread.yield()`不会导致阻塞。该方法与 `sleep()`类似，只是不能由用户指定暂停多长时间。 
3. **`thread.join()`**：当前线程里调用**其它线程** t 的`join()`方法，当前线程进入 WAITING/TIMED\_WAITING 状态，当前线程**不释放**已经持有的对象锁。线程 t 执行完毕或者 millis 时间到，当前线程一般情况下进入 RUNNABLE 状态，也有可能进入 BLOCKED 状态（**因为`join()`是基于`wait()`实现的**）。 
4. **`obj.wait()`**：当前线程调用对象的`wait()`方法，当前线程**释放**对象锁，进入等待队列。依靠`notify()/notifyAll()`唤醒或者`wait(long timeout)` timeout 时间到自动唤醒。
5. **`obj.notify()`**：唤醒在此对象监视器上等待的单个线程，选择是任意性的。`notifyAll()`唤醒在此对象监视器上等待的所有线程。
6. **`LockSupport.park(), LockSupport.parkUntil(long deadlines)`**：当前线程进入 WAITING/TIMED\_WAITING 状态。对比`wait()`方法，不需要获得锁就可以让线程进入 WAITING/TIMED\_WAITING状态，需要通过`LockSupport.unpark(Thread thread)`唤醒。

### 终止线程

线程执行完 run\(\) 方法后，会自动转换到 TERMINATED 状态，当然如果执行 run\(\) 方法的时候异常抛出，也会导致线程终止。

有时候我们需要强制终止线程，`Thread.stop()` 可以做到，不过已经标记为 `@Deprecated`，所以不建议使用了。应该调用 `interrupt()` 方法。

`stop()` 会真的杀死线程，不给线程喘息的机会，如果线程持有 synchronized 隐式锁，也**不会释放**，很危险。类似的方法还有 `suspend()` 和 `resume()` 方法，这两个方法同样也都不建议使用了。

`interrupt()` 就好多了，仅仅是通知线程。

* **`interrupt()`**：给线程发一个中断信号。
* **`isInterrupted()`**：判断线程是否中断，不会清除中断状态。
* **`interrupted()`**：判断线程是否中断，会清除中断状态。
* **抛出中断异常**：会清除中断状态，表示可以接受下一个中断信号了。

```java
public class Thread implements Runnable {
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }
    
    public boolean isInterrupted() {
        return isInterrupted(false);
    }
    
    /**
     * Tests if some Thread has been interrupted.  The interrupted state
     * is reset or not based on the value of ClearInterrupted that is
     * passed.
     */
    private native boolean isInterrupted(boolean ClearInterrupted);
    
    /**
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public static native void sleep(long millis) throws InterruptedException;
}
```

### `join()`的本质

`join()`的本质是先调用`synchronized`方法获取线程的锁，然后调用`wait()`方法，当线程 terminate 的时候，会调用`notifyAll()`方法。

```java
public class Thread implements Runnable {
    /**
     * As a thread terminates the {@code this.notifyAll} method is invoked. 
     * It is recommended that applications 
     * not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     */
    public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
}
```

### Reference

* [https://blog.csdn.net/pange1991/article/details/53860651](https://blog.csdn.net/pange1991/article/details/53860651)
* [https://www.zhihu.com/question/56494969](https://www.zhihu.com/question/56494969)

## 线程的个数

### CPU 密集型

**对于 CPU 密集型的计算场景，理论上“线程的数量 =CPU 核数”就是最合适的**。不过在工程上，**线程的数量一般会设置为“CPU 核数 +1”**，这样的话，当线程因为偶尔的内存页失效或其他原因导致阻塞时，这个额外的线程可以顶上，从而保证 CPU 的利用率。

### IO 密集型

$$
线程数量 = CPU 核数 * (1 + I/O 耗时 ÷ CPU 耗时)
$$



