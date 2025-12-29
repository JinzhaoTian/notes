---
leetcode: LeetCode 283. Move Zeroes
difficulties: EASY
link: https://leetcode.cn/problems/move-zeroes
tags:
  - LeetCode
  - 双指针
  - Hot100
---

## 题目

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。



## 题解

### C++

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int n = nums.size(), left = 0, right = 0;
        while (right < n) {
            if (nums[right]) {
                swap(nums[left], nums[right]);
                left++;
            }
            right++;
        }
    }
};
```