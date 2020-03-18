# Singleton

## **Preface**

### 概念

**定义：**单例模式确保一个类只有一个实例，并提供一个全局访问点。

唯一的作用范围：

* 单例模式：进程间不唯一，进程内唯一，线程间唯一。
* 线程单例：线程内唯一，线程间不唯一。
* 集群单例：进程间唯一。

**使用场景：**

* 解决资源访问冲突。比如日志都往一个文件写，写如文件的类就应该是单例。
* 表示全局唯一数据，比如配置、线程池、缓存、注册表。

### 缺点

* 对 OOP 特性支持不友好，对抽象、继承、多态支持不友好。
* 会隐藏类之间的依赖关系。
* 对扩展性不友好。比如连接池若设计为单例，若以后需要增加一个慢 sql 连接池，就需要两个实例了。
* 对测试性不友好。
* 不支持有参数的构造函数。

### 替代方案

* 静态方法。但是并不能解决上节的缺点。
* 将单例生成的对象作为参数传递给函数。可以解决第二点缺点。
* 工厂模式。
* IOC 容器。
* 程序员自己保证不要创建两个对象。

## 饿汉式

不管是否需要这个对象都会创建。

### 直接实例化

```java
public class Singleton {
	public static final Singleton1 INSTANCE = new Singleton();
	private Singleton(){}
}
```

{% hint style="warning" %}
若该类实现了 Serializable 接口，就可以通过[序列化](../../../java/class-libraries/java-io.md#dui-xiang-xu-lie-hua)的方式破坏单例模式。可以添加 readResolve\(\) 方法，自定义返回对象的策略。
{% endhint %}

### 枚举

最简洁的方式。

```java
public enum Singleton {
	INSTANCE
}
```

### 静态代码块

适合于初始化比较复杂的情况。

```java
public class Singleton {
	public static final Singleton INSTANCE;
	private String info;
	
	static{
		try {
			Properties pro = new Properties();
			pro.load(Singleton.class.getClassLoader().getResourceAsStream("single.properties"));
			INSTANCE = new Singleton(pro.getProperty("info"));
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}
	
	private Singleton(){}

	public String getInfo() {
		return info;
	}
}
```

## 懒汉式

延迟创建对象。

### 线程不安全

适用于单线程。

```java
public class Singleton {
	private static Singleton instance;
	private Singleton(){
		
	}
	public static Singleton getInstance(){
		if(instance == null){
			instance = new Singleton();
		}
		return instance;
	}
}
```

### 线程安全 - 双重检测

适用于多线程。

```java
public class Singleton {
	private static Singleton instance;
	private Singleton(){
		
	}
	public static Singleton getInstance(){
		if(instance == null){
			synchronized (Singleton.class) {
				if(instance == null){
				    instance = new Singleton();
				}
			}
		}
		return instance;
	}
}
```

{% hint style="info" %}
网上有人说 instance 应该加上 volatile 修饰，因为由于指令重排序，导致对象被 new 出来，赋值给 instance 后，但是还没有初始化就被使用了。实际上，这个**只有在很低版本的 JVM 才有这个问题**，高版本已经解决这个问题了，把 new 操作和初始化做成原子操作。
{% endhint %}

### 线程安全 - 静态内部类

适用于多线程。

**原理：**静态内部类不会自动随着外部类的加载和初始化而初始化。

```java
public class Singleton {
	private Singleton(){}
	private static class Inner{
		private static final Singleton INSTANCE = new Singleton();
	}
	
	public static Singleton getInstance(){
		return Inner.INSTANCE;
	}
}
```

## 线程单例

通过一个线程安全的 Map，key 是线程 ID，value 是对象。其实 Java 提供的 ThreadLocal 可以轻松实现线程单例，不过 ThreadLocal 底层也是用 Map 实现的。

```java
public class Singleton {
  private static final ConcurrentHashMap instances = 
    new ConcurrentHashMap<>();

	private Singleton(){}
	
	public static Singleton getInstance(){
		Long currentThreadId = Thread.currentThread().getId(); 
		instances.putIfAbsent(currentThreadId, new IdGenerator()); 
		return instances.get(currentThreadId);
	}
}
```

## 集群单例

需要把单例序列化到外部存储，进程在使用这个单例时需要先从外部存储读取到内存，并反序列化为对象，使用完成后再序列化回外部存储。是从外部存储读取、写入时需要加锁。

```java
public class Singleton {
  private static SharedObjectStorage storage = new FileSharedObjectStorage();
  private static DistributedLock lock = new DistributedLock();
  private static Singleton instance;

	private Singleton(){}
	
	public synchronized static Singleton getInstance() {
	  if (instance == null) { 
	    lock.lock(); 
	    instance = storage.load(Singleton.class); 
	  } 
	  return instance; 
	}
	
	public synchroinzed void freeInstance() { 
	  storage.save(this, Singleton.class); 
	  instance = null; //释放对象 lock.unlock(); 
	}
}
```

## 多例模式

多例指的是一个类可以创建多个对象，但是个数有限，比如 3 个。

```java
public class Singleton {
  private static final int SERVER_COUNT = 3;
  private static final Map<Long, Singleton> instances = new HashMap<>();

  static {
    instances.put(1L, new instances("a"));
    instances.put(2L, new instances("b"));
    instances.put(3L, new instances("c"));
  }
  
  private String name;

  private instances(String name) {
    this.name = name;
  }

  public Singleton getInstance(long serverNo) {
    return instances.get(serverNo);
  }

  public Singleton getRandomInstance() {
    Random r = new Random();
    int no = r.nextInt(SERVER_COUNT)+1;
    return instances.get(no);
  }
}
```

这个与工厂模式有点类似，多例模式创建的都是同一个类的对象，工厂模式创建的是不同的子类对象。也类似于享元模式。

枚举也相当于多例模式。

## 小结

1. 饿汉式都不存在线程安全问题。
2. 饿汉式枚举的方式最简洁，懒汉式静态内部类的方式最简洁。
3. 上述实现存在多个类加载器问题。如果有两个及以上的类加载器都加载了这个类，那么还是会产生多个单件。解决办法是自行指定类加载器，并指定同一个类加载器。

