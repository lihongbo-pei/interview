# Algorithm

## 数据结构

### 红黑树

红黑树（Red Black Tree）是一种自平衡二叉查找树。由于其自平衡的特性，保证了最坏情形下在 O(logn) 时间复杂度内完成查找、增加、删除等操作，性能表现稳定。在 JDK 中，`TreeMap`、`TreeSet` 以及 JDK1.8 的 `HashMap` 底层都用到了红黑树。

### 为什么需要红黑树？

红黑树的诞生就是为了解决二叉查找树的缺陷。

二叉查找树是一种基于比较的数据结构，它的每个节点都有一个键值，而且左子节点的键值小于父节点的键值，右子节点的键值大于父节点的键值。这样的结构可以方便地进行查找、插入和删除操作，因为只需要比较节点的键值就可以确定目标节点的位置。但是，二叉查找树有一个很大的问题，就是它的形状取决于节点插入的顺序。如果节点是**按照升序或降序的方式插入的**，那么二叉查找树就会退化成一个线性结构，也就是一个链表。这样的情况下，二叉查找树的性能就会大大降低，时间复杂度就会从 O(logn) 变为 O(n)。

红黑树的诞生就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

### 红黑树特点

1. 每个节点非红即黑。黑色决定平衡，红色不决定平衡。这对应了 2-3 树中一个节点内可以存放 1~2 个节点。
2. 根节点总是黑色的。
3. 每个叶子节点都是黑色的空节点（NIL 节点）。这里指的是红黑树都会有一个空的叶子节点，是红黑树自己的规则。
4. 如果节点是红色的，则它的子节点必须是黑色的（反之不一定）。通常这条规则也叫不会有连续的红色节点。一个节点最多临时会有 3 个子节点，中间是黑色节点，左右是红色节点。
5. 从任意节点到它的叶子节点或空子节点的每条路径，必须包含相同数目的黑色节点（即相同的黑色高度）。每一层都只是有一个节点贡献了树高决定平衡性，也就是对应红黑树中的黑色节点。

正是这些特点才保证了红黑树的平衡，让红黑树的高度不会超过 2log(n+1)。

## 算法

### 快速排序 (Quick Sort)

快速排序用到了分治思想，同样的还有归并排序。乍看起来快速排序和归并排序非常相似，都是将问题变小，先排序子串，最后合并。不同的是快速排序在划分子问题的时候经过多一步处理，将划分的两组数据划分为一大一小，这样在最后合并的时候就不必像归并排序那样再进行比较。但也正因为如此，划分的不定性使得快速排序的时间复杂度并不稳定。

快速排序的基本思想：通过一趟排序将待排序列分隔成独立的两部分，其中一部分记录的元素均比另一部分的元素小，则可分别对这两部分子序列继续进行排序，以达到整个序列有序。

**算法步骤**

快速排序使用[分治法](https://zh.wikipedia.org/wiki/分治法)（Divide and conquer）策略来把一个序列分为较小和较大的 2 个子序列，然后递归地排序两个子序列。具体算法描述如下：

1. 从序列中**随机**挑出一个元素，做为 “基准”(`pivot`)；
2. 重新排列序列，将所有比基准值小的元素摆放在基准前面，所有比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个操作结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
3. 递归地把小于基准值元素的子序列和大于基准值元素的子序列进行快速排序。

**代码实现**

```java
import java.util.concurrent.ThreadLocalRandom;

class Solution {
    public int[] sortArray(int[] a) {
        quick(a, 0, a.length - 1);
        return a;
    }

    // 快速排序的核心递归函数
    void quick(int[] a, int left, int right) {
        if (left >= right) { // 递归终止条件：区间只有一个或没有元素
            return;
        }
        int p = partition(a, left, right); // 分区操作，返回分区点索引
        quick(a, left, p - 1); // 对左侧子数组递归排序
        quick(a, p + 1, right); // 对右侧子数组递归排序
    }

    // 分区函数：将数组分为两部分，小于基准值的在左，大于基准值的在右
    int partition(int[] a, int left, int right) {
        // 随机选择一个基准点，避免最坏情况（如数组接近有序）
        int idx = ThreadLocalRandom.current().nextInt(right - left + 1) + left;
        swap(a, left, idx); // 将基准点放在数组的最左端
        int pv = a[left]; // 基准值
        int i = left + 1; // 左指针，指向当前需要检查的元素
        int j = right; // 右指针，从右往左寻找比基准值小的元素

        while (i <= j) {
            // 左指针向右移动，直到找到一个大于等于基准值的元素
            while (i <= j && a[i] < pv) {
                i++;
            }
            // 右指针向左移动，直到找到一个小于等于基准值的元素
            while (i <= j && a[j] > pv) {
                j--;
            }
            // 如果左指针尚未越过右指针，交换两个不符合位置的元素
            if (i <= j) {
                swap(a, i, j);
                i++;
                j--;
            }
        }
        // 将基准值放到分区点位置，使得基准值左侧小于它，右侧大于它
        swap(a, j, left);
        return j;
    }

    // 交换数组中两个元素的位置
    void swap(int[] a, int i, int j) {
        int t = a[i];
        a[i] = a[j];
        a[j] = t;
    }
}
```

### 堆排序 (Heap Sort)

堆排序是指利用堆这种数据结构所设计的一种排序算法。堆是一个近似完全二叉树的结构，并同时满足**堆的性质**：即**子结点的值总是小于（或者大于）它的父节点**。

**算法步骤**

1. 将初始待排序列 $(R_1, R_2, \dots, R_n)$ 构建成大顶堆，此堆为初始的无序区；
2. 将堆顶元素 $R_1$ 与最后一个元素 $R_n$ 交换，此时得到新的无序区 $(R_1, R_2, \dots, R_{n-1})$ 和新的有序区 $R_n$, 且满足 $R_i \leqslant R_n (i \in 1, 2,\dots, n-1)$；
3. 由于交换后新的堆顶 $R_1$ 可能违反堆的性质，因此需要对当前无序区 $(R_1, R_2, \dots, R_{n-1})$ 调整为新堆，然后再次将 $R_1$ 与无序区最后一个元素交换，得到新的无序区 $(R_1, R_2, \dots, R_{n-2})$ 和新的有序区 $(R_{n-1}, R_n)$。不断重复此过程直到有序区的元素个数为 $n-1$，则整个排序过程完成。

**算法分析**

- **稳定性**：不稳定
- **时间复杂度**：最佳：$O(nlogn)$， 最差：$O(nlogn)$， 平均：$O(nlogn)$
- **空间复杂度**：$O(1)$



请实现有重复数字的升序数组的二分查找

给定一个 元素有序的（升序）整型数组 nums 和一个目标值 target ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1

示例1

输入

```
[1,2,4,4,5],4
```

输出

```
2
```

```java
		/**
     * 如果目标值存在返回下标，否则返回 -1
     * @param nums int整型一维数组
     * @param target int整型
     * @return int整型
     */
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                // 如果中间值等于目标值，返回中间值的下标
                return mid;
            } else if (nums[mid] < target) {
                // 如果中间值小于目标值，目标值在右半部分
                left = mid + 1;
            } else {
                // 如果中间值大于目标值，目标值在左半部分
                right = mid - 1;
            }
        }
        // 如果没有找到目标值，返回 -1
        return -1;
    }
```



给定一个仅包含数字![img](https://www.nowcoder.com/equation?tex=%5C%200-9) 的二叉树，每一条从根节点到叶子节点的路径都可以用一个数字表示。
例如根节点到叶子节点的一条路径是![img](https://www.nowcoder.com/equation?tex=1%5Cto%202%5Cto%203),那么这条路径就用![img](https://www.nowcoder.com/equation?tex=%5C%20123) 来代替。
找出根节点到叶子节点的所有路径表示的数字之和
例如：

![img](https://uploadfiles.nowcoder.com/images/20200807/999991351_1596786228797_BC85E8592A231E74E5338EBA1CFB2D20)

这颗二叉树一共有两条路径，
根节点到叶子节点的路径 ![img](https://www.nowcoder.com/equation?tex=1%5Cto%202) 用数字![img](https://www.nowcoder.com/equation?tex=%5C%2012) 代替
根节点到叶子节点的路径 ![img](https://www.nowcoder.com/equation?tex=1%5Cto%203) 用数字![img](https://www.nowcoder.com/equation?tex=%5C%2013) 代替
所以答案为![img](https://www.nowcoder.com/equation?tex=%5C%2012%2B13%3D25)

示例1

输入

```
{1,2,3}
```

输出

```
25
```

```java
import java.util.*;

/*
 * public class TreeNode {
 *   int val = 0;
 *   TreeNode left = null;
 *   TreeNode right = null;
 *   public TreeNode(int val) {
 *     this.val = val;
 *   }
 * }
 */

public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param root TreeNode类 
     * @return int整型
     */
    public int sumNumbers(TreeNode root) {
        // 如果根节点为空，直接返回0
        if (root == null) {
            return 0;
        }
        // 调用辅助方法，从根节点开始，初始路径数字为0
        return dfs(root, 0);
    }

    private int dfs(TreeNode node, int currentNumber) {
        // 如果当前节点为空，返回0
        if (node == null) {
            return 0;
        }
        // 更新当前路径的数字表示
        currentNumber = currentNumber * 10 + node.val;
        // 如果当前节点是叶子节点，返回当前路径的数字表示
        if (node.left == null && node.right == null) {
            return currentNumber;
        }
        // 递归计算左子树和右子树的路径和
        return dfs(node.left, currentNumber) + dfs(node.right, currentNumber);
    }
}
```



给定两个非0整数，求2数的最大公约数和最小公倍数

输入描述

输入两个整数：8 12

输出描述

输出 4 24

示例1

输入

8 12

输出

4 24
