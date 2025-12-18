---
leetcode: LeetCode 44. Wildcard Matching
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/wildcard-matching
tags:
  - LeetCode
  - 动态规划
---

## 题目

给你一个输入字符串 (`s`) 和一个字符模式 (`p`) ，请你实现一个支持 `'?'` 和 `'*'` 匹配规则的通配符匹配：

- `'?'` 可以匹配任何单个字符。
- `'*'` 可以匹配任意字符序列（包括空字符序列）。

判定匹配成功的充要条件是：字符模式必须能够 **完全匹配** 输入字符串（而不是部分匹配）。

## 分析

通配符（wildcard）就是万用牌的意思。

字符串模式的动态规划一般抽象一个二维的 $dp$ 数组，假设字符串 $s$ 的长度为 $m$，字符模式 `p` 的长度为 $n$。

假设字符串 $s$ 遍历到 $i$，字符模式 `p` 遍历到 $j$，如果 $p[j]$ 是正常字符，且 $p[j] == s[i]$，那么他就取决于之前的（即 $dp[i-1][j-1]$）是否能匹配上，如果不相等，就为 `false`；如果  $p[j]$ 是 `'?'` ，那么能匹配上；如果 $p[j]$ 是 `'*'`，也能匹配上，但是由于 `'*'` 也可以匹配空序列，因此可以选择这个 `'*'` 参与匹配或者不参与匹配，不参与匹配 （即 $dp[i][j - 1]$），参与（即 $dp[i-1][j]$）就可以看当前字符串 $s[i]$ 之前的字符是否也能被 `'*'` 匹配
$$
dp[i][j] = \begin{cases}
				dp[i-1][j-1]\ \&\&\ (s[i] == p[j] ),\ p[i]\ is\ a-z \\
				dp[i-1][j-1],\ p[i]\ is\ ? \\
				dp[i][j-1]\ ||\ dp[i-1][j] ,\ p[i]\ is\ * \\
		   \end{cases}
$$


由于星号的匹配是不确定的，所以需要枚举所有的匹配项，可以用**动态规划**来解决。动态规划的关键在于写转移函数，状态初始化条件。

## 题解

### C++

```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int m = s.length(), n = p.length();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        dp[0][0] = 1;
        for (int i = 1; i <= n; ++i){
            if (p[i - 1] == '*'){
                dp[0][i] = 1;
            } else {
                break;
            }
        }

        for (int i = 1;i <= m; ++i){
            for (int j = 1;j <= n; ++j){
                if (p[j - 1] == '*') {
                    dp[i][j] = dp[i][j - 1] | dp[i - 1][j];
                } else if (p[j - 1] == '?' || s[i - 1] == p[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                }
            }
        }
        return dp[m][n];
    }
};
```

### Python

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        m, n = len(s), len(p)

        dp = [[False] * (n + 1) for _ in range(m + 1)]
        dp[0][0] = True
        for i in range(1, n + 1):
            if p[i - 1] == '*':
                dp[0][i] = True
            else:
                break
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if p[j - 1] == '*':
                    dp[i][j] = dp[i][j - 1] | dp[i - 1][j]
                elif p[j - 1] == '?' or s[i - 1] == p[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
                
        return dp[m][n]
```
