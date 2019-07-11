## House Robber

### 问题

```
You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.

Example 1:

Input: [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and then rob house 3 (money = 3).
             Total amount you can rob = 1 + 3 = 4.
Example 2:

Input: [2,7,9,3,1]
Output: 12
Explanation: Rob house 1 (money = 2), rob house 3 (money = 9) and rob house 5 (money = 1).
             Total amount you can rob = 2 + 9 + 1 = 12.

```

### 思路

该问题具备以下特征：

* 给定约束条件下的最优解问题，其中，约束条件是不能选择相邻的两个数。
* 当前问题依赖于子问题的最优解

可以考虑采用dp实现：

* 初始化dp数组result, 这里仅需要一维数组
* 求递推公式：result[i] = max(result[i-1], result[i-2) + a[i]) 

```

 public int rob(int[] nums) {
        if (nums.length == 0) return 0;
        if (nums.length == 1) return nums[0];

        //dp array init
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);

        //dp status transfer, 递推公式
        for (int i = 2; i < nums.length; i++){
            dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
        }

        //最后一个节点即为最优值
        return dp[dp.length - 1];
 }


```

### 动态规划

常见问题：

* 背包/旅行问题
* 最长公共子串
* 最长公共子序列
