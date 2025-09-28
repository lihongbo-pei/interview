# CMB

1.求解最长递增子序列（LIS）时，数组[10,9,2,5,3,7,101,18]的LIS长度是（4）

动态规划中的一个经典问题

2.表达式a+b*(c-d)/e的后缀形式是（ abcd-\*e/+ ）

遇到右括号出栈

8.在一棵度为4的树T中，若有15个度为4的结点，8个度为3的结点，2个度为2的结点，12个度为1的结点，则树T的叶结点个数是多少?

度数之和 = 边数E，即 E = 4×15+3×8+2×2+1×12+0×L

边数E = 节点数N - 1，节点数 N=15+8+2+12+L

10.分析慢查询日志发现，语句：

```sql
SELECT cust_id FROM Logs WHERE status ='UNDO' AND created_date >CURRENT_DATE - INTERVAL'1 days'
```

效率低下，优化此查询的最高效方案是? C

A.为created_date创建单列索引
B.将INTERVAL函数改为固定日期
C.为s(created_date,status)创建复合索引
D.使用子查询替代条件过滤

1.题干:银行风控系统需要检测账户间的循环交易(A-B-C-A形成环路)，以下代码使用深度优先搜索(DFS)检测交易关系图中是否存在环路。请补全缺失代码。	

问题：请补全下面程序中4处【】位置缺失的代码，以实现该功能需求，注意：填写后请保留“【】”框。

```java
import java.util.*;

// 账户节点定义
class AccountNode{
  String accountId;
  List<AccountNode> transactions = new ArrayList<>();
  
  public AccountNode(String id){
    this.accountId = id;
  }
}
  
  
// 环路检测器	
class CycleDetector{
  private Set(AccountNode> visited = new HashSet<>():
	private Set<AccountNode> recursionStack = new HashSet<>();
              
	public boolean hasCycle (AccountNode node)(
		if(recursionStack.contains(node)){  //空1：检测到递归栈中已存在该节点
			return true:
    }
		if (visited.contains(node)){
      return false;
    }
		recursionStack.add(node); // 空2：标记当前节点进入递归栈
		visited.add(node):
		for(AccountNode neighbor: node.transactions){
      if(hasCycle(neighbor)){ // 空3：递归检测邻居节点
				return true;
      }
    }
		recursionStack.remove(node); // 空4：当前节点退出递归栈
		return false	
}

// 使用示例	
public class BankTransactionAnalysis{
  public static void main(String[] args){
    AccountNode a = new AccountNode("A");
		AccountNode	b =	new	AccountNode("B");
		AccountNode c =	new	AccountNode("C");
    
		a.transactions.add (b);
		b.transactions.add(c);
		c.transactions.add(a); // 形成A→B→C→A环路	
		System.out.println(new CyeleDetector().hasCycle(a));  // 应输出true	
  }
}

```



2.题干：用多线程实现多个int数组的并发求和。关闭线程的时候，需要等待所有任务完成后才能退出。

问题：请补全下面程序中5处【】位置缺失的代码，以实现输出以下结果。注意：填写后请保留“【】”框。

第1组结果：15
第2组结果：40
第3组结果：65
第4组结果：90

```java
public class ParallelCalulator{
  private final ExecutorService executorService;
  public ParallelCalulator(int threadCount){
    this.executorService = Executors.newFixedThreadPool(threadCount); // 【】初始化线程池数量
  }
  public Future<Integer> sum(int[] array){
    return executorService.submit(() -> {
      int sum = 0;
      for(int i : array){
        sum += i; // 【】累加数组中的元素
      }
      return sum;
    })
  }
  
  public void shutdown() throws InterruptedException{
    executorService.shutdown();
    executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.NANOSECONDS); // 【】等待所有任务完成
  }
  
  public static void main(String[] args) throws Exception{
    ParallelCalulator calulator = new ParallelCalulator(4);
    int[][] arrays = new int[][]{{1,2,3,4,5}, {6,7,8,9,10}, {11,12,13,14,15}, {16,17,18,19,20}};
    List<Future<Integer>> futures = new ArrayList<>();
    for(int[] array : arrays){
      futures.add(calculator.sum(array)); // 【】并发求和
    }
    for(int i = 0; i < futures.size(); i++){
      System.out.println("第" + (i + 1) + "组结果：" + futures.get(i).get()); // 【】获取并打印结果
    }
    calulator.shutdown();
  }
}
```





[leetcode198 打家劫舍](https://leetcode.cn/problems/house-robber)

作为一名招商银行风险控制专家，日常需要负责优化ATM机网络的资金分配。每个ATM机内近期的日均现金流储备不同（用非负整数数组nums表示），但相邻ATM机采用同一套安全系统。若同时对相邻ATM机进行升级维护，会触发资金流动异常警报。

给定一个代表各ATM机日均现金流储备的非负整数数组，请计算在保证系统安全的前提下，通过选择最优ATM机组合进行升级维护，能够实现的最大资金调配额度。

请设计算法解决该**动态规划**问题，确保招商银行资金调配系统的安全性与效益最大化。

示例1:
输入：nums = [2,7,9,3,1]
输出：12
解释：选择第1台（额度=2）、第3台（额度=9）和第5台（额度=1），总调配额度为2+9+1=12。

示例2:
输入：nums = [1,2,3,1]
输出：4
解释：选择第1台（额度=1）和第3台（额度=3），总调配额度为1+3=4。

示例3:
输入：nums = [6,1,9,1]
输出：15
解释：选择第1台（额度=6）和第3台（额度=9），总调配额度为6+9=15。

```java
public static int atmUpgrade(int[] nums) {
    int n = nums.length;
    if (n == 0) return 0;
    if (n == 1) return nums[0];
    
    int[] dp = new int[n + 1];
    dp[0] = 0; 
    dp[1] = nums[0]; 
    
    for (int i = 2; i <= n; i++) {
      	// 状态转移方程
        dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i - 1]);
    }
    return dp[n];
}
```

