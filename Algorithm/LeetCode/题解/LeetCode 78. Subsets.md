---
leetcode: LeetCode 78. Subsets
difficulties: MEDIUM
link: https://leetcode.cn/problems/subsets
tags:
  - LeetCode
  - Hot100
  - 回溯
---

## 题目

给你一个整数数组 `nums` ，数组中的元素互不相同。返回该数组所有可能的子集（幂集）。


## 题解

### C++

```cpp
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> ans;
        int n = nums.size();
        for (int i = 0; i < (1 << n); ++i) {
            vector<int> t;
            for (int j = 0; j < n; ++j) {
                if (i & (1 << j)) {
                    t.push_back(nums[j]);
                }
            }
            ans.push_back(t);
        }
        return ans;
    }
};
```