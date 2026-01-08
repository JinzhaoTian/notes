---
leetcode: LeetCode 238. Product of Array Except Self
difficulties: MEDIUM
link: https://leetcode.cn/problems/product-of-array-except-self
tags:
  - LeetCode
  - Hot100
  - 前缀和
---
## 题目

给你一个整数数组 `nums`，返回数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除了 `nums[i]` 之外其余各元素的乘积 。


## 题解

### C++

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();

        vector<int> l(n, 0), r(n, 0);
        l[0] = 1;
        for (int i = 1; i < n; ++i) {
            l[i] = l[i - 1] * nums[i - 1];
        }

        r[n - 1] = 1;
        for (int i = n - 2; i >= 0; --i) {
            r[i] = r[i + 1] * nums[i + 1];
        }

        vector<int> ans(n);
        for (int i = 0; i < n; ++i) {
            ans[i] = l[i] * r[i];
        }

        return ans;
    }
};
```