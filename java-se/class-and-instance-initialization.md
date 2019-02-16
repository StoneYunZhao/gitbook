# Class & Instance Initialization

## 类初始化

### 触发条件

1. 一个类要创建实例需要初始化这个类。
2. 一个子类的初始化需要先初始化父类。
3. 调用类的静态方法会初始化这个类，所以main方法所在类需要先初始化。

### 初始化过程

1. 类的初始化即调用`<clinit>()`方法。
2. `<clinit>()`组成：

   * 静态变量的显示赋值。
   * 静态代码块。

   > 注意： 静态变量的显示赋值和静态代码块按照书写顺序执行。

3. `<clinit>()`方法只执行一次。

## 实例初始化

1. 实例初始化就是执行`<init>()`方法。
2. `<init>()`可能有多个，有几个构造函数就有几个。
3. `<init>()`方法组成：

   * super\(\)，一定是第一行。
   * 非静态变量显示赋值代码。
   * 非静态代码块。
   * 构造方法代码，一定是最后。

   > 注意： 非静态变量显示赋值代码和非静态代码块按照书写顺序执行。

4. `<init>()`方法每次调用构造方法都会执行。

## 例子

分析下面代码的执行结果：

```java
public class Father {
    private int i = test();
    private static int j = method();

    static {
        System.out.print(1);
    }

    public Father() {
        System.out.print(2);
    }

    {
        System.out.print(3);
    }

    public int test() {
        System.out.print(4);
        return 1;
    }

    public static int method(){
        System.out.print(5);
        return 1;
    }
}

public class Son extends Father {
    private int i = test();
    private static int j = method();

    static {
        System.out.print(6);
    }

    public Son() {
        System.out.print(7);
    }

    {
        System.out.print(8);
    }

    public int test() {
        System.out.print(9);
        return 1;
    }

    public static int method(){
        System.out.print(10);
        return 1;
    }

    public static void main(String[] args) {
        Son s1=new Son();
        System.out.println();
        Son s2 = new Son();
    }
}
```

结果：

```text
51106932987
932987
```

若main函数改为空函数，则结果为：

```text
51106
```
