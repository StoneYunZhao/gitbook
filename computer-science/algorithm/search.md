# Binary Search

**描述**：二分查找针对有序的数据集合，每次与区间中间元素对比，将待查找区间缩为一半，直到找到或区间被缩小为 0。

![](../../.gitbook/assets/image%20%2839%29.png)

**时间复杂度**：O\(logn\)，O\(logn\) 极其高效，**有时甚至比常量级更高效**。

**代码要点**：

* 循环退出条件：`low <= high`
* mid取值：`mid = (low + high) / 2`可能造成溢出，改为`low + ((high - low) >> 1)`
* `low = mid + 1, high = mid - 1`，而不是`low = mid, high = mid`，因为会造成死循环。

**局限性**：

* 依赖顺序表结构，因为需要下标**随机访问**。
* 需要**有序**数据。若不有序，需要插入、删除不频繁，一次排序多次查找的情况。
* 数据量小时没必要用二分，直接顺序查找。例外，数据比较耗时，如长度为300的字符串，那么此时就算数据量小也要二分。
* 数据量大不合适，因为需要连续内存。

{% hint style="info" %}
大部分情况，用二分查找解决的问题，都可以用散列表和二叉树解决，但是后者需要额外的内存空间。
{% endhint %}

二分查找**最简单**情况是有序数组中**不存在重复元素**，当有序数组中存在重复数据时，稍微复杂。二分查找变体：

**查找第一个值等于给定值的元素：**

```java
public static int search(int[] nums, int target) {
    if (nums == null) {
        return -1;
    }
    
    int low = 0, high = nums.length - 1;
    while (low <= high) {
        int mid = low + ((high - low) >> 1);
        if (nums[mid] > target) {
            high = mid - 1;
        } else if (nums[mid] < target) {
            low = mid + 1;
        } else {
            if (mid == 0 || nums[mid - 1] < target) {
                return mid;
            } else {
                high = mid - 1;
            }
        }
    }
    return -1;
}
```

**查找最后一个值等于给定值得元素：**

**查找第一个大于等于给定值的元素：**

**查找最后一个小于等于给定值的元素：**

{% hint style="info" %}
对于等值查询，散列表和二叉树都能解决二分查找的问题，但是对于“近似”查询（如上面的四种变体），二分查找的优势就明显了。
{% endhint %}

