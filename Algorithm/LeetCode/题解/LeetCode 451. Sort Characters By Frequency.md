[451. Sort Characters By Frequency](https://leetcode-cn.com/problems/sort-characters-by-frequency/)


Python:

```
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

