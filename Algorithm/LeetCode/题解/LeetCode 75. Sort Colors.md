---
leetcode: LeetCode 75. Sort Colors
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/sort-colors
tags:
  - LeetCode
---


## 题解

### Python

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nums.sort()
```

但是题目要求不能使用库函数：_“You must solve this problem without using the library's sort function.”_，而且最好想一个常数空间，一次遍历的算法。所以要深入探究一下：

可以使用双指针法：

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        p0, p1 = 0, 0
        for i in range(len(nums)):
            if nums[i] == 1:
                nums[i], nums[p1] = nums[p1], nums[i]
                p1 += 1
            elif nums[i] == 0:
                nums[i], nums[p0] = nums[p0], nums[i]
                if p0 < p1:
                    nums[i], nums[p1] = nums[p1], nums[i]
                p0 += 1
                p1 += 1
```
