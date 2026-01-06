---
leetcode: LeetCode 560. Subarray Sum Equals K
difficulties: MEDIUM
link: https://leetcode.cn/problems/subarray-sum-equals-k
tags:
  - LeetCode
  - Hot100
  - 前缀和
  - 哈希表
---

## 题目

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回该数组中和为 `k` 的子数组的个数。

## 题解

### C++

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> mp;
        mp[0] = 1;
        int cnt = 0, pre = 0;
        for (int i = 0; i < nums.size(); ++i) {
            pre += nums[i];
            if (mp.find(pre - k) != mp.end()) {
                cnt += mp[pre - k];
            }
            mp[pre]++;
        }
        return cnt;
    }
};
```