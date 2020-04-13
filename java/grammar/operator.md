# Operator

## 运算符优先级

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

## 位运算

| 运算符 | 定义 |
| :---: | :--- |
| & | 两个都为 1，结果为 1；其它为 0 |
| \| | 两个都为 0，结果为 0；其它为 1 |
| ^ | 两个相同为 0，不相同为 1 |
| ~ | 0 变 1，1变 0 |
| &lt;&lt; | 左移，高位丢弃，低位补 0 |
| &gt;&gt; | 右移，正数高位补 0，负数高位补 1，低位丢弃 |
| &gt;&gt;&gt; |  无符号右移，正负数高位都补 0，低位丢弃 |

### 位移运算

```java
public static void main(String[] args) {
    print(5);
    print(5 >> 1);
    print(5 >>> 1);
    print(-5);
    print(-5 >> 1);
    print(-5 >>> 1);
}
private static void print(int i) {
    System.out.println(i + ":" + Integer.toBinaryString(i));
}

// output
5:101
2:10
2:10
-5:11111111111111111111111111111011
-3:11111111111111111111111111111101
2147483645:1111111111111111111111111111101
```

### 异或运算

```java
// 异或运算的本质是：二进制下，不考虑进位的加法。

x ^ 0 = x
x ^ 1s = ~x // 1s = ~0
x ^ (~x) = 1s
x ^ x = 0
a ^ b = c => a ^ c = b, b ^ c = a
a ^ b ^ c = a ^ (b ^ c)
```

### 套路

```java
//// 常用

// 判断奇偶
x & 1 == 1 // x % 2 == 1
x & 1 == 0 // x % 2 == 0

// 把最低位的 1 变为 0
x & (x - 1) // 111 & 110 = 110; 110 & 101 = 100

// 补码编码
-x = ~x + 1;

// 得到最低位的 1，假设 x 为 byte
// 0000 0111 & 1111 1001 = 0000 0001
// 0000 0110 & 1111 1010 = 0000 0010
x & -x 

//// 不常用

x & (~0 << n) // 最右边的 n 位清 0
(x >> n) & 1 // 获取第 n 位值
x & (1 << (n - 1)) // 获取第 n 位的幂值
x | (1 << n) // 将第 n 位置为 1
x & (~(1 << n)) // 将第 n 为置为 0
x & ((1 << n) - 1) // 最高位到第 n 位清 0
x & (~(1 << (n + 1)) - 1) // 最低位到第 n 位清 0
```

### 例题

* [LeetCode 191：计算二进制数有几个 1。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/bit/LT191.java)
* [LeetCode 231：判断整数是否为 2 的 n 次方。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/bit/LT231.java)
* [LeetCode 338：计算 0 到 n 的每个数的二进制表示中 1 的个数。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/bit/LT338.java)

