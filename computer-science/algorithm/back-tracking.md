# Back Tracking

回溯的处理思想，类似于枚举。我们枚举所有的解，找到满足期望的解。为了有规律地枚举所有的解，避免重复和遗漏，把问题求解的过程分为多个阶段。每个阶段，会有多个选择，随意选择一个，若发现这个选择不满足条件，则回到上一步，做另外一个选择。

比如深度优先遍历即使用的回溯思想。

回溯算法一般适用用递归来实现。剪枝操作是提高回溯效率的一种技巧。

## 案例

* [LeetCode 20：生成有效括号组合。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/backtracking/LT22.java)
* [LeetCode 51：N 皇后问题，输出所有解。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/backtracking/LT51.java)
* [LeetCode 52：N 皇后问题，输出解的个数。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/backtracking/LT52.java)
* [LeetCode 37：数独的解。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/backtracking/LT37.java)

### 0-1 背包

问题：一个背包总承重 W kg，有 n 个重量不等的物体，选择几件物品装到背包中，使背包中的总重量最大。

注意：此问题与[贪心算法中的背包问题](greedy-algorithm.md#bei-bao-wen-ti)区别在于这里的物体不能分割，只能选择装或不装，所以叫做 0-1 问题。此类问题用[动态规划](dynamic-programming.md)效率更高。

思路：用回溯思想的话，若有 n 个物体，把问题分为 n 个阶段，每个阶段对应是否选择物品。可以用一点剪枝技巧，当已经选择的物品超过 W 后，就停止后面的阶段。

```java
private int maxW = Integer.MIN_VALUE; //存储背包中物品总重量的最大值
private int[] weight = {2，2，4，6，3}; // 每个物品的重量
private int n = 5; // 物品个数
private int w = 9; // 背包重量
// cw表示当前已经装进去的物品的重量和；i表示考察到哪个物品了；
// 调用方式：f(0, 0)
public void f(int i, int cw) {
  if (cw == w || i == n) { // cw==w表示装满了;i==n表示已经考察完所有的物品
    if (cw > maxW) maxW = cw;
    return;
  }
  f(i+1, cw); // 不选择此阶段的物品
  if (cw + weight[i] <= w) {// 已经超过可以背包承受的重量的时候，就不要再装了
    f(i+1,cw + weight[i]); // 选择此阶段的物品
  }
}
```

### 正则

