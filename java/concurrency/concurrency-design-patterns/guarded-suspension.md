# Guarded Suspension

为了效率，现在异步调用使用越来越广泛。异步调用需要解决两个问题：

1. 线程 T1 发送异步请求后，怎么等待结果返回？
2. 多个线程发送异步请求，多个结果返回后，怎么对应到原来的请求？

## Guarded Suspension 模式

对于问题 1，假设服务 A 异步调用服务 B，服务 A 发送请求的线程是 T1，但是接受服务 B 返回结果的线程是 T2，那么 T1 怎么等待返回结果呢？

![](../../../.gitbook/assets/image%20%28139%29.png)

使用 Guarded Suspension 模式可以解决上述问题，使用一个 GuardedObject 对象，内部有一个成员变量，成员方法 get\(Predicate&lt;T&gt; p\) 中的 p 就是 T1 用来检验结果是否已经返回，成员方法  onChanged\(\) 就是 T2 获取到返回结果后调用。

![](../../../.gitbook/assets/image%20%2870%29.png)

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

## 扩展 Guarded Suspension 模式

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

