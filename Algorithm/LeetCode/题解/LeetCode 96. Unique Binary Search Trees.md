[Leetcode 96. Unique Binary Search Trees](https://leetcode-cn.com/problems/unique-binary-search-trees/)


可以自己分析情况，从0个节点，1个，2个，3个，直到4个节点，5个，就可以看出来规律了。

卡特兰数（Catalan数）

## 题解

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
