---
leetcode: LeetCode 46. Permutations
difficulties: MEDIUM
link: https://leetcode.cn/problems/permutations
tags:
  - LeetCode
  - Hot100
  - 回溯
---

## 题目

给定一个不含重复数字的数组 `nums` ，返回其所有可能的全排列。

## 题解

### C++

```cpp
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> ans;
        backtrack(ans, nums, 0, nums.size());
        return ans;
    }

    void backtrack(vector<vector<int>>& ans, vector<int>& output, int first, int len) {
        if (first == len) {
            ans.emplace_back(output);
            return;
        }
        for (int i = first; i < len; ++i) {
            swap(output[i], output[first]);
            backtrack(ans, output, first + 1, len);
            swap(output[i], output[first]);
        }
    }
};
```