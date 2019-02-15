# Singleton

**定义：**单例模式确保一个类只有一个实例，并提供一个全局访问点。

**使用场景：**线程池、缓存、注册表。

## 饿汉式

不管是否需要这个对象都会创建。

### 直接实例化

```java
public class Singleton {
	public static final Singleton1 INSTANCE = new Singleton();
	private Singleton(){}
}
```

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

### 线程安全

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

### 静态内部类

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

