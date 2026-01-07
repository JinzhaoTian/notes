---
leetcode: LeetCode 53. Maximum Subarray
difficulties: MEDIUM
link: https://leetcode.cn/problems/maximum-subarray
tags:
  - LeetCode
  - Hot100
  - 动态规划
---

## 题目

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

## 分析

感觉是一个动态规划的模版题，关键是状态怎么转移的。因为要求子数组连续，当遍历到 `i` 时，此时的最大和 `dp[i]`，要么就和前面的连续起来，即 `dp[i - 1] + nums[i]`；要么就是不连续了，自己单独，即 `nums[i]` 。

## 题解

### C++

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int pre = 0, maxAns = nums[0];
        for (const auto &x: nums) {
            pre = max(pre + x, x);
            maxAns = max(maxAns, pre);
        }
        return maxAns;
    }
};
```