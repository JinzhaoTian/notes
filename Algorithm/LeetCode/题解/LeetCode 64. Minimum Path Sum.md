---
leetcode: LeetCode 64. Minimum Path Sum
difficulties: MEDIUM
link: https://leetcode.cn/problems/minimum-path-sum
tags:
  - LeetCode
  - 动态规划
  - Hot100
---

## 分析

给定一个包含非负整数的 `m x n` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。说明：每次只能向下或者向右移动一步。

### 暴力解

一眼看过去就想用 dfs 暴力搜，每个格子有两个分支，感觉是个 $O(2^n)$ 的算法。估计也可以加个记忆化搜索，当然我感觉这是个动态规划的模版题。

$$
dp[m][n]=grid[m][n] + min(dp[m][n-1], dp[m-1][n])
$$


### 动态规划

动态规划就是状态转移方程，加上初始化条件。


## 题解

### C++

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        if (grid.size() == 0 || grid[0].size() == 0)
            return 0;
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> dp = vector<vector<int>>(m, vector<int>(n));
        dp[0][0] = grid[0][0];
        for (int i = 1; i < m; i++){
            dp[i][0] = dp[i - 1][0] + grid[i][0];
        }
        for (int j = 1; j < n; j++){
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }
        for (int i = 1; i < m; i++){
            for (int j = 1; j < n; j++){
                dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[m - 1][n - 1];
    }
};
```