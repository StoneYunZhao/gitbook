# Dynamic Programming

## 理论

动态规划适合解决**多阶段决策最优解**模型的问题，若一个最优问题的解决过程需要经历多个决策阶段，每个阶段对应一组状态，然后寻找一组决策序列，经过这组决策序列，能够产生最优值。

适合动态规划的问题必须满足 3 个特征：

* **最优子结构**，问题的最优解包含子问题的最优解，可以通过子问题的最优解推导出问题的最优解。或者说后面阶段的状态可以通过前面阶段的状态推导出来。
* **无后效性**：
  * 在推导后面阶段状态时，只关心前面阶段的状态值，不关心这个值是怎么一步步推导得来的。
  * 某阶段的状态一旦确定，就不受之后阶段的决策影响。
* **重复子问题**，不同的决策序列，达到某个相同阶段时，可能会产生重复的状态。

举例分析，一个 n\*n 的矩阵，存储正整数，从左上角到右下角移动，只能向右或向下移动，路径上经过的数字之和为路径的长度，求最短路径长度。

![](../../.gitbook/assets/image%20%28146%29.png)

* 总共要走 2\*\(n-1\) 步，每一步都需要做向右或向下的决策，符合多阶段决策最优解模型。
* 状态定义为 min\_dist\(i, j\)，表示 \(0, 0\) 到 \(1, 1\) 的最短路径长度，符合最优子结构：`min_dist(i, j) = w[i][j] + min(min_dist(i, j - 1), min_dist(i - 1, j))`
* 根据上面的公式，我们只关心上一步状态的值，符合无后效性。
* 到达位置 \(i, j\)，有多条路线，即符合重复子问题。

## 思路

解决动态规划问题一般有两种思路。

### 状态转移表法

一般能用动态规划解决的问题都可以用回溯解决。解决问题的步骤：

1. 回溯算法实现
2. 定义状态
3. 画递归树
4. 找重复子问题
5. 画状态转移表，状态表一般为二维数组，每个状态有行、列、值三个变量，如果问题比较复杂，状态表也可能是多维的。
6. 依据递推关系填表，从前往后，依据递推关系，分阶段填充状态表中的每个状态。
7. 将填表过程翻译成代码

{% hint style="info" %}
在第 4 步找到重复子问题后，除了使用状态转移表法，还可以使用**回溯加备忘录**的方式，效率是差不多的。
{% endhint %}

以上文的最短路径为例，先用回溯实现：

```java
private int minDist = Integer.MAX_VALUE;
private int[][] w = ...;
private int n = ...;

// 调用方式：minDistBacktracing(0, 0, 0);
public void minDistBT(int i, int j, int dist) {
  if (i == n && j == n) {
    if (dist < minDist) minDist = dist;
    return;
  }
  if (i < n) { // 往下走
    minDistBT(i + 1, j, dist+w[i][j]);
  }
  if (j < n) { // 往右走
    minDistBT(i, j+1, dist+w[i][j]);
  }
}
```

定义状态为 \(i, j, dist\)，表示到达 \(i, j\) 的路径长度为 dist，画递归树：

![](../../.gitbook/assets/image%20%28241%29.png)

从状态树中可以看出，尽管 \(i, j, dist\) 不存在重复，但是 \(i, j\) 有很多重复，我们只需要选出 \(i, j\) 中 dist 最小的节点，所以存在重复子问题。画状态转移表，并一步一步填充：

![](../../.gitbook/assets/image%20%2819%29.png)

![](../../.gitbook/assets/image%20%28236%29.png)

翻译成动态规划的代码：

```java
public int minDistDP(int[][] matrix, int n) {
  int[][] states = new int[n][n];
  int sum = 0;
  for (int j = 0; j < n; ++j) { // 初始化states的第一行数据
    sum += matrix[0][j];
    states[0][j] = sum;
  }
  sum = 0;
  for (int i = 0; i < n; ++i) { // 初始化states的第一列数据
    sum += matrix[i][0];
    states[i][0] = sum;
  }
  for (int i = 1; i < n; ++i) {
    for (int j = 1; j < n; ++j) {
      states[i][j] = 
            matrix[i][j] + Math.min(states[i][j-1], states[i-1][j]);
    }
  }
  return states[n-1][n-1];
}
```

### 状态转移方程法

1. 找最优子结构，分析某个问题是如何通过子问题来递归求解
2. 写状态方程，即递推公式
3. 将状态方程翻译成代码，一般有两种实现方式，一种是递归加备忘录，一种是递推迭代（其实与状态转移表的实现是一样的，只是思路不一样）。

还是以上文的最短路径为例，状态方程上面已经给出：

```text
min_dist(i, j) = w[i][j] + min(min_dist(i, j - 1), min_dist(i - 1, j))
```

然后用递归加备忘录的方式实现：

```java
private int[][] matrix = 
         {{1，3，5，9}, {2，1，3，4}，{5，2，6，7}，{6，8，4，3}};
private int n = 4;
private int[][] mem = new int[4][4];
public int minDist(int i, int j) { // 调用minDist(n-1, n-1);
  if (i == 0 && j == 0) return matrix[0][0];
  if (mem[i][j] > 0) return mem[i][j];
  int minLeft = Integer.MAX_VALUE;
  if (j-1 >= 0) {
    minLeft = minDist(i, j-1);
  }
  int minUp = Integer.MAX_VALUE;
  if (i-1 >= 0) {
    minUp = minDist(i-1, j);
  }
  
  int currMinDist = matrix[i][j] + Math.min(minLeft, minUp);
  mem[i][j] = currMinDist;
  return currMinDist;
}
```

## 案例

### 0-1 背包

见[回溯算法一节](back-tracking.md#01-bei-bao)的 0-1 背包问题，画递归树：

![](../../.gitbook/assets/image%20%2818%29.png)

定义状态 \(i, cw\)，表示在决策第 i 个物品是否放入背包时，当前背包重量为 cw。有很多重复子问题，画状态转移表，states\[n\]\[w + 1\] 每一行表示第 i 个物品决策完后，当前背包中的重量有哪些值：

![](../../.gitbook/assets/image%20%28260%29.png)

![](../../.gitbook/assets/image%20%2850%29.png)

翻译成代码：

```java
// weight:物品重量，n:物品个数，w:背包可承载重量
public int knapsack(int[] weight, int n, int w) {
  boolean[][] states = new boolean[n][w+1]; // 默认值false
  states[0][0] = true;  // 第一行的数据要特殊处理，可以利用哨兵优化
  if (weight[0] <= w) {
    states[0][weight[0]] = true;
  }
  for (int i = 1; i < n; ++i) { // 动态规划状态转移
    for (int j = 0; j <= w; ++j) {// 不把第i个物品放入背包
      if (states[i-1][j] == true) states[i][j] = states[i-1][j];
    }
    for (int j = 0; j <= w-weight[i]; ++j) {//把第i个物品放入背包
      if (states[i-1][j]==true) states[i][j+weight[i]] = true;
    }
  }
  for (int i = w; i >= 0; --i) { // 输出结果
    if (states[n-1][i] == true) return i;
  }
  return 0;
}
```

时间复杂度 O\(n\*w\)，空间复杂度 O\(n\*w\)，空间复杂度可以优化为 O\(w\)，用一个 w + 1 的一维数据记录状态：

```java
public static int knapsack2(int[] items, int n, int w) {
  boolean[] states = new boolean[w+1]; // 默认值false
  states[0] = true;  // 第一行的数据要特殊处理，可以利用哨兵优化
  if (items[0] <= w) {
    states[items[0]] = true;
  }
  for (int i = 1; i < n; ++i) { // 动态规划
    // 这里为什么要从后往前？
    for (int j = w-items[i]; j >= 0; --j) {//把第i个物品放入背包
      if (states[j]==true) states[j+items[i]] = true;
    }
  }
  for (int i = w; i >= 0; --i) { // 输出结果
    if (states[i] == true) return i;
  }
  return 0;
}
```

### 0-1 背包问题升级版

物品有价值，选择某些物品放入背包，在满足背包最大重量限制的条件下，背包中的总价值最大。

同样用回溯可以解决：

```java
private int maxV = Integer.MIN_VALUE; // 结果放到maxV中
private int[] items = {2，2，4，6，3};  // 物品的重量
private int[] value = {3，4，8，9，6}; // 物品的价值
private int n = 5; // 物品个数
private int w = 9; // 背包承受的最大重量
public void f(int i, int cw, int cv) { // 调用f(0, 0, 0)
  if (cw == w || i == n) { // cw==w表示装满了，i==n表示物品都考察完了
    if (cv > maxV) maxV = cv;
    return;
  }
  f(i+1, cw, cv); // 选择不装第i个物品
  if (cw + weight[i] <= w) {
    f(i+1,cw+weight[i], cv+value[i]); // 选择装第i个物品
  }
}
```

递归树如下：

![](../../.gitbook/assets/image%20%28176%29.png)

可以看出，\(2, 2, 4\) 和\(2, 2, 3\)，我们只需要选择前者，即对于相同的 \(i, cw\)，只需要保留 cv 最大的那个。states\[n\]\[w + 1\] 中保存的是当前状态的最大值：​

```java
public static int knapsack3(int[] weight, int[] value, int n, int w) {
  int[][] states = new int[n][w+1];
  for (int i = 0; i < n; ++i) { // 初始化states
    for (int j = 0; j < w+1; ++j) {
      states[i][j] = -1;
    }
  }
  states[0][0] = 0;
  if (weight[0] <= w) {
    states[0][weight[0]] = value[0];
  }
  for (int i = 1; i < n; ++i) { //动态规划，状态转移
    for (int j = 0; j <= w; ++j) { // 不选择第i个物品
      if (states[i-1][j] >= 0) states[i][j] = states[i-1][j];
    }
    for (int j = 0; j <= w-weight[i]; ++j) { // 选择第i个物品
      if (states[i-1][j] >= 0) {
        int v = states[i-1][j] + value[i];
        if (v > states[i][j+weight[i]]) {
          states[i][j+weight[i]] = v;
        }
      }
    }
  }
  // 找出最大值
  int maxvalue = -1;
  for (int j = 0; j <= w; ++j) {
    if (states[n-1][j] > maxvalue) maxvalue = states[n-1][j];
  }
  return maxvalue;
}
```

