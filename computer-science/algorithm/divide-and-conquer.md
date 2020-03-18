# Divide and Conquer

分治算法，将原始问题划分成 n 个规模较小，并且结构与原问题相似的子问题，递归解决这些子问题，然后合并其结果，就得到原问题的解。

分治算法与递归：分治算法是处理问题的一种思想，递归是一种编程技巧；分治算法一般都比较适合递归来实现。每一层递归都会涉及三种操作：分解、解决、合并。

适用条件：

* 原问题与分解的小问题有相同的模式。
* 原问题与子问题可以独立求解，没有相关性。
* 分解有终止条件。
* 合并操作的复杂度不能太高。

## 例题

* [LeetCode 50：求 x 的 n 次方。](https://github.com/StoneYunZhao/algorithm/blob/master/src/main/java/com/zhaoyun/leetcode/divide/LT50.java)

