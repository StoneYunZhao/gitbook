# Singleton

**定义：**单例模式确保一个类只有一个实例，并提供一个全局访问点。

**使用场景：**

* 解决资源访问冲突。比如日志都往一个文件写，写如文件的类就应该是单例。
* 表示全局唯一数据，比如配置、线程池、缓存、注册表。

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

## 小结

1. 饿汉式都不存在线程安全问题。
2. 饿汉式枚举的方式最简洁，懒汉式静态内部类的方式最简洁。
3. 上述实现存在多个类加载器问题。如果有两个及以上的类加载器都加载了这个类，那么还是会产生多个单件。解决办法是自行指定类加载器，并指定同一个类加载器。

