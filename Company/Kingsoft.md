# Kingsoft

Cwbc和XHRlyb生活在s市，这天他们打算一起出去旅游。旅行地图上有n个城市，它们之间通过n-1条道路联通。
Cwbc和XHRlyb第一天会在s市住宿，并游览与它距离不超过1的所有城市，之后的每天会选择一个城市住宿，然后游览与它距离不超过1的所有城市。
他们不想住在一个已经浏览过的城市，又想尽可能多的延长旅行时间。 XHRlyb想知道她与Cwbc最多能度过多少天的时光呢?
聪明的你在仔细阅读题目后，一定可以顺利的解决这个问题!
输入描述
第一行，两个正整数n和s，表示城市个数和第一天住宿的城市s。
接下来n-1行，每行两个整数x和y，表示城市x与城市y之间有一条双向道路，保证无环.
输出描述
第一行，一个非负整数表示答案。
补充说明
1sns500000,1s s,x,ysn.

给定一个非负整数N，返回 N! 结果的末尾为0的数量

```java
class Solution {
    public int trailingZeroes(int n) {
        int ans = 0;
        while (n != 0) {
            n /= 5;
            ans += n;
        }
        return ans;
    }
}
```



给定一个正整数n，请找出最少个数的完全平方数，使得这些完全平方数的和等于n。

完全平方指用一个整数乘以自己例如1\*1，2\*2，3\*3等，以此类推。若一个数能表示成某个整数的平方的形式，则称这个数为完全平方数，例如1，4，9和16都是完全平方数，但是2，3，5，8，11等不是。

输入描述 仅一行，输入一个正整数n

输出描述 按题目要求输出完全平方数之和为n的最少个数

示例1
输入 5
输出 2
说明 1+4=5

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        sc.close();

        int[] dp = new int[n + 1];
        // 初始化为最大值（n最多由n个1组成）
        for (int i = 1; i <= n; i++) {
            dp[i] = i;  // 最坏情况：i = 1² + 1² + ... + 1²（i次）
        }

        // 动态规划
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j * j <= i; j++) {
                dp[i] = Math.min(dp[i], dp[i - j * j] + 1);
            }
        }

        System.out.println(dp[n]);
    }
}

```

