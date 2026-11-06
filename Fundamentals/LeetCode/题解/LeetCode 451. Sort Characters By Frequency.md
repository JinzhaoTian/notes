---
leetcode: LeetCode 451. Sort Characters By Frequency
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/sort-characters-by-frequency
tags:
  - LeetCode
  - 排序
---




## 题解

### Python

```python
class Solution:
    def frequencySort(self, s: str) -> str:
        count = collections.Counter(s)
        _list = list(count.items())
        _list.sort(key = lambda x: x[1], reverse=True)
        res = ''
        for item in _list:
            res += item[0] * item[1]
        return res
```

