---
leetcode: LeetCode 42. Trapping Rain Water
difficulties: HARD
link: https://leetcode.cn/problems/trapping-rain-water
tags:
  - LeetCode
  - Hot100
  - 动态规划
  - 单调栈
  - 双指针
---

## 题目

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

## 题解

### C++

1. 动态规划：
```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int n = height.size();
        if (n == 0) {
            return 0;
        }
        vector<int> leftMax(n);
        leftMax[0] = height[0];
        for (int i = 1; i < n; ++i) {
            leftMax[i] = max(leftMax[i - 1], height[i]);
        }

        vector<int> rightMax(n);
        rightMax[n - 1] = height[n - 1];
        for (int i = n - 2; i >= 0; --i) {
            rightMax[i] = max(rightMax[i + 1], height[i]);
        }

        int ans = 0;
        for (int i = 0; i < n; ++i) {
            ans += min(leftMax[i], rightMax[i]) - height[i];
        } 
        return ans;
    }
};
```

2. 单调栈：
```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int n = height.size();
        int ans = 0;
        stack<int> stac;
        for (int i = 0; i < n; ++i) {
            while (!stac.empty() && height[i] > height[stac.top()]) {
                int top = stac.top();
                stac.pop();
                if (stac.empty()) {
                    break;
                }
                int left = stac.top();
                int currWidth = i - left - 1;
                int currHeight = min(height[left], height[i]) - height[top];
                ans += currWidth * currHeight;
            }
            stac.push(i);
        }
        return ans;
    }
};
```