---
leetcode: LeetCode 41. First Missing Positive
difficulties: HARD
link: https://leetcode.cn/problems/first-missing-positive
tags:
  - LeetCode
  - Hot100
  - 哈希表
---

## 题目

给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

## 分析

一眼看过去可以直接用哈希表扫一遍然后统计，空间复杂度 `O(n)`，时间复杂度 `O(n)`。但是由于只是找一个数，因此可以直接改造当前的数组，变成一个哈希表。

首先就是将小于等于 0 的数处理掉，都变成 `n + 1` ，不影响结果；然后是处理 `0 < num <= n` 范围内的数，将 `num` 标记所属的位置下标代表的数，标记为负数。然后再遍历一次，找到第一个大于0的数的下标 `i`，就是第一个缺失的整数 `i + 1`。


## 题解

### C++

1. 哈希表
```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            if (nums[i] <= 0) {
                nums[i] = n + 1;
            }
        }

        for (int i = 0; i < n; ++i) {
            int num = abs(nums[i]);
            if (num <= n) {
                nums[num - 1] = -abs(nums[num - 1]);
            }
        }

        for (int i = 0; i < n; ++i) {
            if (nums[i] > 0) {
                return i + 1;
            }
        }

        return n + 1;
    }
};
```