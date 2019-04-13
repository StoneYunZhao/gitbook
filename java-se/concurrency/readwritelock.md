# ReadWriteLock

在生产中经常有读多写少的场景，比如缓存。针对这种情况，Java 提供了一个接口`ReadWriteLock`。

* 允许多个线程同时读共享变量。
* 只允许一个线程写共享变量。
* 当一个线程在写共享变量时，不允许其它线程读和写操作。

### 缓存实现

```java
class Cache<K,V> {
  final Map<K, V> m = new HashMap<>();
  final ReadWriteLock rwl = new ReentrantReadWriteLock();
  // 读锁
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  // 读缓存
  V get(K key) {
    r.lock();
    try { return m.get(key); }
    finally { r.unlock(); }
  }
  // 写缓存
  V put(String key, Data v) {
    w.lock();
    try { return m.put(key, v); }
    finally { w.unlock(); }
  }
}
```

### 缓存数据按需加载

缓存中的数据是需要初始化的，若数据量小，可以直接一次性全部加载；若数据量大，则要按需加载（懒加载），即当查询的时候再加载。

```java
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         
    try {
      v = m.get(key); 
    } finally{
      r.unlock();     
    }
    // 缓存中存在，返回
    if(v != null) {   
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); 
      if(v == null){  
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }
```

### 读写锁升级与降级

把上面的代码改成如下是否可行？

```java
// 读缓存
r.lock();         
try {
  v = m.get(key); 
  if (v == null) {
    w.lock();
    try {
      // 再次验证并更新缓存
      // 省略详细代码
    } finally{
      w.unlock();
    }
  }
} finally{
  r.unlock();     
}
```

答案是不行，会导致写锁永远等待。

* **锁的升级**：先获取了读锁，再获取写锁。这是**不允许**的。
* **锁的降级**：先获取了写锁，再获取读锁。这是**允许**的。

### 总结

* 读写锁也支持公平和非公平模式。
* 只有**写锁支持条件变量**，读锁不支持。

