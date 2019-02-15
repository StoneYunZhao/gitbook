# Operator

## 自增与自减

分析下面代码的结果：

```java
public class Test {
	
	public static void main(String[] args) {
		int i = 1;
		i = i++;
		int j = i++;
		int k = i + ++i * i++;
		System.out.println("i=" + i);
		System.out.println("j=" + j);
		System.out.println("k=" + k);
	}
}
```

结果：

```text
i=4
j=1
k=11
```

解释：

1. 下面为字节码，自增自减没有入栈出栈，所以L1结束后，`i=1` 。
2. 赋值运算符`=`最后进行，所以L2结束后，`j=1` 。
3. `=`号右边，从左到右依次入栈，但是实际计算，看运算符优先级，所以L3后，`k=11` 。

```java
   L0
    LINENUMBER 6 L0
    ICONST_1
    ISTORE 1
   L1
    LINENUMBER 7 L1
    ILOAD 1
    IINC 1 1
    ISTORE 1
   L2
    LINENUMBER 8 L2
    ILOAD 1
    IINC 1 1
    ISTORE 2
   L3
    LINENUMBER 9 L3
    ILOAD 1
    IINC 1 1
    ILOAD 1
    ILOAD 1
    IINC 1 1
    IMUL
    IADD
    ISTORE 3
```

