---
leetcode: LeetCode 11. Container With Most Water
difficulties: MEDIUM
link: https://leetcode.cn/problems/container-with-most-water
tags:
  - LeetCode
  - 双指针
  - Hot100
---

## 题目

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水，返回容器可以储存的最大水量。


## 题解

### C++

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int ans = 0, left = 0, right = height.size() - 1;
        while (left < right) {
            int area = min(height[left], height[right]) * (right - left);
            ans = max(ans, area);
            if (height[left] <= height[right]) {
                left++;
            } else {
                right--;
            }
        }
        return ans;
    }
};
```