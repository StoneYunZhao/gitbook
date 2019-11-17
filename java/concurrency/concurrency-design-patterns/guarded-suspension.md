# Multitheading If

## Guarded Suspension

为了效率，现在异步调用使用越来越广泛。异步调用需要解决两个问题：

1. 线程 T1 发送异步请求后，怎么等待结果返回？
2. 多个线程发送异步请求，多个结果返回后，怎么对应到原来的请求？

### 常规的 Guarded Suspension 模式

对于问题 1，假设服务 A 异步调用服务 B，服务 A 发送请求的线程是 T1，但是接受服务 B 返回结果的线程是 T2，那么 T1 怎么等待返回结果呢？

![](../../../.gitbook/assets/image%20%28153%29.png)

使用 Guarded Suspension 模式可以解决上述问题，使用一个 GuardedObject 对象，内部有一个成员变量，成员方法 get\(Predicate&lt;T&gt; p\) 中的 p 就是 T1 用来检验结果是否已经返回，成员方法  onChanged\(\) 就是 T2 获取到返回结果后调用。

![](../../../.gitbook/assets/image%20%2874%29.png)

```java
public final class GuardedObject<T> {
    private String obj;

    private final Lock lock = new ReentrantLock();
    private final Condition done = lock.newCondition();

    private static final int TIMEOUT = 1;

    public String get(Predicate<String> p) {
        lock.lock();
        try {
            while (!p.test(obj)) {
                done.await(TIMEOUT, TimeUnit.SECONDS);
            }
        } catch (InterruptedException e) {
            throw new RuntimeException();
        } finally {
            lock.unlock();
        }
        return obj;
    }

    public void onChanged(String obj) {
        lock.lock();
        try {
            this.obj = obj;
            done.signalAll();
        } finally {
            lock.unlock();
        }
    }
}

/**
 * 服务 A 发送请求给服务 B，然后等待请求返回
 */
public final class Sender {
    public void send() {
        String req = "request";
        System.out.println("Service A send request to service B: " + req);

        GuardedObject go = new GuardedObject<>();
        go.get(Objects::nonNull);
    }
}

/**
 * 服务 A 另外一个线程收到服务 B 的返回后，把结果反映到 GuardedObject
 */
public final class Receiver {
    public void handle(String resp) {
        GuardedObject go = ???
        go.onChanged(resp);
    }
}
```

### 扩展 Guarded Suspension 模式

Guarded Suspension 模式有一个问题，由于 Sender 会发送多条个请求，在 Receiver 中，怎么对应上 Sender 中的 GuardedObject 呢？我们可以维护一个请求 Id 与 GuardedObject 的关系表，即扩展的 Guarded Suspension 实现。

```java
public final class GuardedObject {
    private String obj;

    private final Lock lock = new ReentrantLock();
    private final Condition done = lock.newCondition();

    private static final int TIMEOUT = 1;

    private final static Map<String, GuardedObject> gos = new ConcurrentHashMap<>();

    public static GuardedObject create(String key) {
        GuardedObject go = new GuardedObject();
        gos.put(key, go);
        return go;
    }

    public static  void handle(String key, String obj) {
        GuardedObject go = gos.remove(key);
        if (go != null) {
            go.onChanged(obj);
        }
    }

    public String get(Predicate<String> p) {
        lock.lock();
        try {
            while (!p.test(obj)) {
                done.await(TIMEOUT, TimeUnit.SECONDS);
            }
        } catch (InterruptedException e) {
            throw new RuntimeException();
        } finally {
            lock.unlock();
        }
        return obj;
    }

    private void onChanged(String obj) {
        lock.lock();
        try {
            this.obj = obj;
            done.signalAll();
        } finally {
            lock.unlock();
        }
    }
}

/**
 * 服务 A 发送请求给服务 B，然后等待请求返回
 */
public final class Sender {
    public void send() {
        String req = "request", id ="id";
        System.out.println("Service A send request to service B: " + req);

        GuardedObject go = GuardedObject.create(id);
        go.get(Objects::nonNull);
    }
}

/**
 * 服务 A 另外一个线程收到服务 B 的返回后，把结果反映到 GuardedObject
 */
public final class Receiver {
    public void handle(String id, String resp) {
        GuardedObject.handle(id, resp);
    }
}
```

## Balking

Guarded Suspension 模式本质是有一个状态变量，一个线程会判断这个状态变量是否已经满足要求，若没有满足，则等待直到满足再进行接下来的逻辑。

Balking 模式的本质有点相似，同样有一个状态变量，一个线程会判断这个状态变量是否已经满足要求，若没有满足，则直接返回；若满足，则执行接下来的逻辑。与 Guarded Suspension 模式的差别就在于状态变量没有满足要求时，是直接返回还是等待直到满足要求。

所以 Guarded Suspension 模式通常也被称为“多线程版本的 if”，只是这个 if 是需要等待的，而且很执着。Balking 也是多线程版本的 if，这个 if 不需要等待。

以下是一个自动存盘的逻辑，采用了 Balking 模式：

```java
public final class AutoSave {
    private boolean changed = false;

    private ScheduledExecutorService ses = Executors.newSingleThreadScheduledExecutor();

    private void startAutoSave() {
        ses.scheduleWithFixedDelay(this::autoSave, 5, 5, TimeUnit.SECONDS);
    }

    private void autoSave() {
        synchronized (this) {
            if (!changed) {
                return;
            }
            changed = false;
        }

        execSave();
    }

    private void execSave() {
        System.out.println("save");
    }

    public void edit() {
        System.out.println("edit");

        change();
    }

    private void change() {
        synchronized (this) {
            changed = true;
        }
    }
}
```

[单例模式的双重检测方式](../../../computer-science/design-patterns/singleton.md#xian-cheng-an-quan)本质也是采用了 Balking 模式。

