# Algorithm

## 数据结构

### 红黑树特点

1. 每个节点非红即黑。黑色决定平衡，红色不决定平衡。这对应了 2-3 树中一个节点内可以存放 1~2 个节点。
2. 根节点总是黑色的。
3. 每个叶子节点都是黑色的空节点（NIL 节点）。这里指的是红黑树都会有一个空的叶子节点，是红黑树自己的规则。
4. 如果节点是红色的，则它的子节点必须是黑色的（反之不一定）。通常这条规则也叫不会有连续的红色节点。一个节点最多临时会有 3 个子节点，中间是黑色节点，左右是红色节点。
5. 从任意节点到它的叶子节点或空子节点的每条路径，必须包含相同数目的黑色节点（即相同的黑色高度）。每一层都只是有一个节点贡献了树高决定平衡性，也就是对应红黑树中的黑色节点。

正是这些特点才保证了红黑树的平衡，让红黑树的高度不会超过 2log(n+1)。





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
