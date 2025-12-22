---
leetcode: LeetCode 96. Unique Binary Search Trees
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/unique-binary-search-trees
tags:
  - LeetCode
  - 二叉树
  - 动态规划
  - 数学
---

## 题目

给你一个整数 `n` ，求恰由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的**二叉搜索树**有多少种？返回满足题意的二叉搜索树的种数。


## 分析

可以自己分析情况，从0个节点，1个，2个，3个，直到4个节点，5个，就可以看出来规律了。

1. 可以隐隐约约感觉到是个递归的形式，因此可以往动态规划上靠，每多加一个数（假设此时为 `n + 1` 个数），都要把前面整过的形状再整一遍，然后添加新的数，将前面整过的形状依次枚举出来就是序列长度为 `n`、以 `i` 为根的不同二叉搜索树个数 $F(i,n), 1≤i≤n$ ，因此总数为，
$$
G(n)= \sum_{i=1}^n​F(i,n)
$$
2. 卡特兰数（Catalan数）：
$$
C_0​=1,C_{n+1}​= \frac{2(2n+1)}{n+2} C_n​
$$

## 题解

### C++

```cpp
class Solution {
public:
    int numTrees(int n) {
        vector<int> G(n + 1, 0);
        G[0] = 1;
        G[1] = 1;

        for (int i = 2; i <= n; ++i) {
            for (int j = 1; j <= i; ++j) {
                G[i] += G[j - 1] * G[i - j];
            }
        }

        return G[n];
    }
};
```

### Python

```python
class Solution:
    def numTrees(self, n):
        """
        :type n: int
        :rtype: int
        """
        G = [0]*(n+1)
        G[0], G[1] = 1, 1

        for i in range(2, n+1):
            for j in range(1, i+1):
                G[i] += G[j-1] * G[i-j]

        return G[n]

```
