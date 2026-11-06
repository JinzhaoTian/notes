---
leetcode: LeetCode 3. Longest Substring Without Repeating Characters
difficulties: MEDIUM
link: https://leetcode.cn/problems/longest-substring-without-repeating-characters
tags:
  - LeetCode
  - Hot100
  - 字符串
  - 滑动窗口
---

## 题目

给定一个字符串 `s` ，请你找出其中不含有重复字符的最长子串的长度。

## 题解

### C++

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<char> occ;
        int n = s.size();
        int r = -1, ans = 0;
        for (int l = 0; l < n; ++l){
            if (l != 0) {
                occ.erase(s[l - 1]);
            }
            while (r + 1 < n && !occ.count(s[r + 1])) {
                occ.insert(s[r + 1]);
                r++;
            }
            ans = max(ans, r - l + 1);
        }
        return ans;
    }
};
```