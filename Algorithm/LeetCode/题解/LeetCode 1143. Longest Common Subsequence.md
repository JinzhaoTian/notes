---
leetcode: LeetCode 1143. Longest Common Subsequence
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/longest-common-subsequence
tags:
  - LeetCode
  - 动态规划
---

谈起动态规划，就不得不提到最长公共子序列。

## 题目

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。

## 分析


### 动态规划

最长公共子序列问题是典型的二维动态规划问题，假设字符串 $text_1$ 和 $text_2$ 的长度分别为 $m$ 和 $n$，创建 $m+1$ 行 $n+1$ 列的二维数组 $dp$，其中 $dp[i][j]$ 表示 $text_1[0:i]$ 和 $text_2[0:j]$ 的最长公共子序列的长度。

只要抽象到上述的状态表达，那么后面的问题就是状态转移方程和初始化条件的问题了。假设 $text_1$ 遍历到位置 $i$ ，$text_2$ 遍历到位置 $j$ ，那么当前这两个字符相等，就是之前都没有这两个字符的时候的“最长公共子序列的值”（递归起来，即 $dp[i - 1][j - 1]$）加上一个字符；不相等的时候，要么为 $text_1$ 加上这个字符与  $text_2$ 不加上这个字符的“最长公共子序列的值”（即 $dp[i][j - 1]$），要么为 $text_1$ 不加上这个字符与  $text_2$ 加上这个字符的“最长公共子序列的值”（即 $dp[i - 1][j]$）。

那么，最终的答案就是
$$
dp[i][j] = \begin{cases}
					dp[i - 1][j - 1] + 1 \\
					max(dp[i - 1][j], dp[i][j - 1]) \\
			\end{cases}
$$

## 题解

### C++

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.length(), n = text2.length();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        for (int i = 1; i <= m; ++i){
            for (int j = 1; j <= n; ++j){
                if (text1[i - 1] == text2[j - 1]){
                    dp[i][j] = dp[i -1][j - 1] + 1;
                } else{
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
};
```