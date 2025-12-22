---
leetcode: LeetCode 347. Top K Frequent Elements
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/top-k-frequent-elements
tags:
  - LeetCode
  - 排序
---

## 分析

寻找前k个出现频率最高的元素，这道题一看就知道用**桶排序**，但是如何设置桶和我想得不一样。

## 题解

### Python

这道题用Python写可以直接用库collections，显得异常方便。

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        count = collections.Counter(nums)
        return [item[0] for item in count.most_common(k)]
```

但是这道题的解法不能仅限于此。

重新回到桶排序，这次不仅仅存储频率，还要把对应的数存下来，所以要用一个字典而不是列表。

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        count = {}
        for item in nums:
            if item not in count:
                count[item] = 1
            else:
                count[item] += 1
        
        _list = list(count.items())
        _list.sort(key = lambda x: x[1], reverse=True)

        res = []
        for i in range(k):
            res.append(_list[i][0])
            
        return res
```


### C++

鉴于STL全都忘完了，注意一下几个数据结构，unordered_map，vector。

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        int maxCnt = 0;
        unordered_map<int, int> hash_map;
        for (auto num : nums) {
            hash_map[num] ++;
            maxCnt = max(maxCnt, hash_map[num]);
        }

        vector<vector<int>> bucket(maxCnt + 1);
        for (auto item : hash_map) {
            int num = item.first, idx = item.second;
            bucket[idx].push_back(num);
        }

        vector<int> res;
        for (int i = bucket.size() - 1; i >= 0; i--){
            if (k == 0) break;
            if (!bucket[i].size()) continue;
            for (int j = 0; j < bucket[i].size(); j++) {
                res.push_back(bucket[i][j]);
                k--;
            }
        }
        return res;
    }
};
```
