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

![](../../.gitbook/assets/image%20%28150%29.png)

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

![](../../.gitbook/assets/image%20%28246%29.png)

从状态树中可以看出，尽管 \(i, j, dist\) 不存在重复，但是 \(i, j\) 有很多重复，我们只需要选出 \(i, j\) 中 dist 最小的节点，所以存在重复子问题。画状态转移表，并一步一步填充：

![](../../.gitbook/assets/image%20%2819%29.png)

![](../../.gitbook/assets/image%20%28241%29.png)

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

* [LeetCode 70：楼梯的走法数。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/dp/LT70.java)
* [LeetCode 120：三角形的数组，从顶到底的最短路径。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/dp/LT120.java)
* [LeetCode 121：买卖股票最大利润，只能买卖一次。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/dp/LT121.java)
* [LeetCode 122：买卖股票最大利润，可以买卖无数次。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/greedy/LT122.java)
* LeetCode 123：买卖股票最大利润，可以买卖两次。
* [LeetCode 188：买卖股票最大利润，可以买卖 k 次。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/dp/LT188.java)
* LeetCode 309：买卖股票最大利润，可以买卖无数次，卖了之后隔一天才能买。
* LeetCode 714：买卖股票最大利润，可以买卖无数次，有交易费。
* [LeetCode 300：最长上升子序列。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/dp/LT300.java)
* [LeetCode 322：最少的硬币凑成总金额。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/dp/LT322.java)

### 0-1 背包

见[回溯算法一节](back-tracking.md#01-bei-bao)的 0-1 背包问题，画递归树：

![](../../.gitbook/assets/image%20%2818%29.png)

定义状态 \(i, cw\)，表示在决策第 i 个物品是否放入背包时，当前背包重量为 cw。有很多重复子问题，画状态转移表，states\[n\]\[w + 1\] 每一行表示第 i 个物品决策完后，当前背包中的重量有哪些值：

![](../../.gitbook/assets/image%20%28266%29.png)

![](../../.gitbook/assets/image%20%2852%29.png)

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

![](../../.gitbook/assets/image%20%28181%29.png)

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

### 莱文斯距离

[LeetCode 72。](https://leetcode-cn.com/problems/edit-distance/)

**编辑距离**：将一个字符串转换为另一个字符串，需要的最少编辑操作（增加、删除、替换）次数。编辑距离越小，说明两个字符串越相似。

莱文斯距离允许增加、删除、替换三种操作，最长公共子串只允许增加、删除两种操作。

![](../../.gitbook/assets/image%20%28134%29.png)

用回溯处理莱文斯距离，两个字符串 a、b，编辑 a 以得到 b。当 a\[i\] 和 b\[j\] 匹配时，递归匹配 a\[i + 1\] 和 b\[j + 1\]；若不匹配，则：

* 删除 a\[i\]，递归匹配 a\[i + 1\] 和 b\[j\]
* 在 a\[i\] 前增加一个字符 b\[j\]，递归匹配 a\[i\] 和 b\[j + 1\]
* 把 a\[i\] 替换成 b\[j\]，递归匹配 a\[i + 1\] 和 b\[j + 1\]

```java
// 回溯实现
private char[] a = "mitcmu".toCharArray();
private char[] b = "mtacnu".toCharArray();
private int n = 6;
private int m = 6;
private int minDist = Integer.MAX_VALUE; // 存储结果
// 调用方式 lwstBT(0, 0, 0);
public lwstBT(int i, int j, int edist) {
  if (i == n || j == m) {
    if (i < n) edist += (n-i);
    if (j < m) edist += (m - j);
    if (edist < minDist) minDist = edist;
    return;
  }
  if (a[i] == b[j]) { // 两个字符匹配
    lwstBT(i+1, j+1, edist);
  } else { // 两个字符不匹配
    lwstBT(i + 1, j, edist + 1); // 删除a[i]
    lwstBT(i, j + 1, edist + 1); // a[i]前添加一个字符
    lwstBT(i + 1, j + 1, edist + 1); // 将 a[i] 替换成 b[j]
  }
}
```

定义状态 \(i, j, min\_edist\)，表示处理到 a\[i\], b\[j\] 时，已经编辑的最小次数，状态转移方程：

```text
如果：a[i]!=b[j]，那么：min_edist(i, j)就等于：
min(min_edist(i-1,j)+1, min_edist(i,j-1)+1, min_edist(i-1,j-1)+1)

如果：a[i]==b[j]，那么：min_edist(i, j)就等于：
min(min_edist(i-1,j)+1, min_edist(i,j-1)+1，min_edist(i-1,j-1))     
```

状态表的填充过程：

![](../../.gitbook/assets/image%20%2838%29.png)

转换为代码：

```java
public int lwstDP(char[] a, int n, char[] b, int m) {
  int[][] minDist = new int[n][m];
  for (int j = 0; j < m; ++j) { // 初始化第0行:a[0..0]与b[0..j]的编辑距离
    if (a[0] == b[j]) minDist[0][j] = j;
    else if (j != 0) minDist[0][j] = minDist[0][j-1]+1;
    else minDist[0][j] = 1;
  }
  for (int i = 0; i < n; ++i) { // 初始化第0列:a[0..i]与b[0..0]的编辑距离
    if (a[i] == b[0]) minDist[i][0] = i;
    else if (i != 0) minDist[i][0] = minDist[i-1][0]+1;
    else minDist[i][0] = 1;
  }
  for (int i = 1; i < n; ++i) { // 按行填表
    for (int j = 1; j < m; ++j) {
      if (a[i] == b[j]) minDist[i][j] = min(
          minDist[i-1][j]+1, minDist[i][j-1]+1, minDist[i-1][j-1]);
      else minDist[i][j] = min(
          minDist[i-1][j]+1, minDist[i][j-1]+1, minDist[i-1][j-1]+1);
    }
  }
  return minDist[n-1][m-1];
}

private int min(int x, int y, int z) {
  int minv = Integer.MAX_VALUE;
  if (x < minv) minv = x;
  if (y < minv) minv = y;
  if (z < minv) minv = z;
  return minv;
}
```

### 最长公共子串

只允许增加和删除，状态 \(i, j, max\_lcs\) 表示 a\[0:i\], b\[0:j\] 的最长公共子串，状态转移方程：

```text
如果：a[i]==b[j]，那么：max_lcs(i, j)就等于：
max(max_lcs(i-1,j-1)+1, max_lcs(i-1, j), max_lcs(i, j-1))；

如果：a[i]!=b[j]，那么：max_lcs(i, j)就等于：
max(max_lcs(i-1,j-1), max_lcs(i-1, j), max_lcs(i, j-1))；
```

```java
public int lcs(char[] a, int n, char[] b, int m) {
  int[][] maxlcs = new int[n][m];
  for (int j = 0; j < m; ++j) {//初始化第0行：a[0..0]与b[0..j]的maxlcs
    if (a[0] == b[j]) maxlcs[0][j] = 1;
    else if (j != 0) maxlcs[0][j] = maxlcs[0][j-1];
    else maxlcs[0][j] = 0;
  }
  for (int i = 0; i < n; ++i) {//初始化第0列：a[0..i]与b[0..0]的maxlcs
    if (a[i] == b[0]) maxlcs[i][0] = 1;
    else if (i != 0) maxlcs[i][0] = maxlcs[i-1][0];
    else maxlcs[i][0] = 0;
  }
  for (int i = 1; i < n; ++i) { // 填表
    for (int j = 1; j < m; ++j) {
      if (a[i] == b[j]) maxlcs[i][j] = max(
          maxlcs[i-1][j], maxlcs[i][j-1], maxlcs[i-1][j-1]+1);
      else maxlcs[i][j] = max(
          maxlcs[i-1][j], maxlcs[i][j-1], maxlcs[i-1][j-1]);
    }
  }
  return maxlcs[n-1][m-1];
}

private int max(int x, int y, int z) {
  int maxv = Integer.MIN_VALUE;
  if (x > maxv) maxv = x;
  if (y > maxv) maxv = y;
  if (z > maxv) maxv = z;
  return maxv;
}
```

