---
leetcode: LeetCode 76. Minimum Window Substring
difficulties: HARD
link: https://leetcode.cn/problems/minimum-window-substring
tags:
  - LeetCode
  - Hot100
  - 字符串
  - 滑动窗口
  - 哈希表
---

## 题目

给定两个字符串 `s` 和 `t`，长度分别是 `m` 和 `n`，返回 `s` 中的最短窗口子串，使得该子串包含 `t` 中的每一个字符（包括重复字符）。如果没有这样的子串，返回空字符串 `""`。

## 题解

### C++

1. 哈希表：超时
```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char, int> ori, cnt;
        for (int i = 0; i < t.size(); ++i) {
            ori[t[i]]++;
        }

        int l = 0, r = 0;
        int minLen = INT_MAX, ansL = -1;

        while (r < s.size()) {
            if (ori.find(s[r]) != ori.end()) {
                cnt[s[r]] ++;
            }
            while (check(ori, cnt) && l <= r) {
                if (r - l + 1 < minLen) {
                    minLen = r - l + 1;
                    ansL = l;
                }
                if (ori.find(s[l]) != ori.end()) {
                    cnt[s[l]]--;
                }
                l++;
            }
            r++;
        }

        return ansL == -1 ? "" : s.substr(ansL, minLen);
    }

    bool check(const unordered_map<char, int> ori, const unordered_map<char, int> cnt){
        for (auto it = ori.begin(); it != ori.end(); it++) {
            auto found = cnt.find(it->first);
            if (found == cnt.end() || found->second < it->second)
                return false;
        }
        return true;
    }
};
```

2. 哈希表：改成全局变量就过了
```cpp
class Solution {
public:
    unordered_map<char, int> ori, cnt;

    string minWindow(string s, string t) {
        
        for (int i = 0; i < t.size(); ++i) {
            ori[t[i]]++;
        }

        int l = 0, r = 0;
        int minLen = INT_MAX, ansL = -1;

        while (r < s.size()) {
            if (ori.find(s[r]) != ori.end()) {
                cnt[s[r]] ++;
            }
            while (check() && l <= r) {
                if (r - l + 1 < minLen) {
                    minLen = r - l + 1;
                    ansL = l;
                }
                if (ori.find(s[l]) != ori.end()) {
                    cnt[s[l]]--;
                }
                l++;
            }
            r++;
        }

        return ansL == -1 ? "" : s.substr(ansL, minLen);
    }

    bool check() {
        for (auto it = ori.begin(); it != ori.end(); it++) {
            auto found = cnt.find(it->first);
            if (found == cnt.end() || found->second < it->second)
                return false;
        }
        return true;
    }
};
```