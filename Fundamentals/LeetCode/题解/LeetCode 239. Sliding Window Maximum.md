---
leetcode: LeetCode 239. Sliding Window Maximum
difficulties: HARD
link: https://leetcode.cn/problems/sliding-window-maximum
tags:
  - LeetCode
  - Hot100
  - 优先队列
---

## 题目

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。返回滑动窗口中的最大值。

## 题解

### C++

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        priority_queue<pair<int, int>> q;
        for (int i = 0; i < k; ++i) {
            q.push({nums[i], i});
        }

        vector<int> ans = { q.top().first };
        for (int i = k; i < nums.size(); ++i) {
            q.push({nums[i], i});
            while (q.top().second <= i - k) {
                q.pop();
            }
            ans.push_back(q.top().first);
        }
        return ans;
    }
};
```