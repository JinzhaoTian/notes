[Leetcode 44. Wildcard Matching](https://leetcode-cn.com/problems/wildcard-matching/)

通配符(wildcard)就是万用牌的意思。

- ***** 表示匹配任意长度的任意字符
- **?** 可以匹配任意一个小写字母

由于星号的匹配是不确定的，所以需要枚举所有的匹配项，可以用**动态规划**来解决。动态规划的关键在于写转移函数，状态初始化条件。

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
