---
leetcode: LeetCode 189. Rotate Array
difficulties: MEDIUM
link: https://leetcode.cn/problems/rotate-array
tags:
  - LeetCode
  - Hot100
  - 数组
  - 双指针
---

## 题目

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

## 题解

### C++

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        reverse(nums, 0, n - 1);
        reverse(nums, 0, k % n - 1);
        reverse(nums, k % n, n - 1);
    }

    void reverse(vector<int>& nums, int start, int end) {
        while (start < end){
            swap(nums[start], nums[end]);
            start += 1;
            end -= 1;
        }
    }
};
```