# Class & Instance Initialization

## 类加载过程

![](../../.gitbook/assets/image%20%28164%29.png)

### 加载（Loading）

1. 通过全类名获取定义此类的二进制字节流。
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在内存中生成一个代表该类的 Class 对象，作为方法区这些数据的访问入口。

### 连接（Linking）

加载阶段和连接阶段的部分内容是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了。

#### 验证（Verification）

确保 Class 文件字节流中的信息符合虚拟机的要求，并且不会危害虚拟机自身的安全。

* **文件格式验证**：是否以魔数开头、版本号是否能被当前虚拟机处理等。
* **元数据验证**：语义分析，是否有父类、父类是否可以被继承、是否为抽象类等。
* **字节码验证**：最复杂的阶段，字节码指令不会跳转到方法体以外等。
* **符号引用验证**：将符号引用转化为直接引用，这个转换发生再解析阶段，此验证确保解析能够正常执行。

#### 准备（Preparing）

**准备阶段是正式为类变量分配内存并设置类变量初始值的阶段**，这些内存都将在**方法区**中分配。

{% hint style="warning" %}
* 仅包括类变量，被 static 修饰，不包括实例变量。
* 若被 static final 修饰（常量），则会在准备阶段直接赋值为代码中定义的值。
* 初始值是[数据类型的 0 值](../grammar/data-types.md#ji-ben-shu-ju-lei-xing)，而不是代码中定义的值。
{% endhint %}

#### 解析（Resolution）

虚拟机将常量池内的**符号引用替换为直接引用**的过程。主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行。

* **符号引用（Symbolic References）**：一组符号描述所引用的目标，任何形式的字面量，只需能无歧义地定位到目标。与内存布局无关。
* **直接引用（Direct References）**：直接指向目标的指针、相对偏移量、间接定位到目标的句柄等。与内存布局有关。

### 初始化（Initialization）

类的加载时机虚拟机并没有规定，但是规定以下情况必须对类进行初始化：

1. 遇到`new`、`getstatic`、`putstatic`、`invokestatic` 这四条字节码指令时。即 new 一个对象、读取类的静态变量（**若静态变量被 final 修饰、在编译期把结果放入常量池除外**）、设置类的静态变量、调用类的静态方法（包括 main 方法）。
2. 使用 java.lang.reflect 对类进行反射调用时。
3. 一个子类的初始化需要先初始化父类。
4. 当使用动态动态语言时，如果一个 MethodHandle 实例的最后解析结构为 REF\_getStatic、REF\_putStatic、REF\_invokeStatic、的方法句柄，并且这个句柄没有初始化，则需要先触发器初始化。

类的初始化即调用`<clinit>()`方法，带锁，线程安全。**仅执行一次**。`<clinit>()`组成：

* 静态变量的显示赋值。
* 静态代码块。

> 注意： 静态变量的显示赋值和静态代码块按照书写顺序执行。

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

## 案例

### 类加载与实例加载顺序

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

### 常量引用

```java
public class ConstantReference {
    public static void main(String[] args) {
        System.out.println(ConstClass.HELLO);
    }

    private static class ConstClass{
        private static final String HELLO = "hello";

        static {
            System.out.println("ConstClass init.");
        }
    }
}
```

上面代码只会输出 hello，因为在编译阶段 ”hello“ 会存储在 ConstantReference 类的常量池中。

若把 HELLO 的 final 修饰符去掉，则会先打印 ”ConstClass init.“

### 父类静态变量引用

```java
public class SuperStaticReference {

    public static void main(String[] args) {
        System.out.println(SubClass.value);
    }

    private static class SuperClass {
        static {
            System.out.println("SuperClass init!");
        }

        static int value = 123;
    }

    private static class SubClass extends SuperClass {
        static {
            System.out.println("SubClass init!");
        }
    }
}

// output
SuperClass init!
123
```

上面代码没有输出 ”SubClass init!“，对于静态字段，只有直接定义这个字段的类才会被初始化。

### 数组引用

```java
public class ArrayReference {
    public static void main(String[] args) {
        SuperClass[] a = new SuperClass[10];
    }

    private static class SuperClass {
        static {
            System.out.println("SuperClass init!");
        }
    }
}
```

上面代码不会输出 ”SuperClass init!“。

