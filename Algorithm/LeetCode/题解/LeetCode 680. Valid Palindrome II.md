---
leetcode: LeetCode 680. Valid Palindrome II
difficulties: EASY
link: https://leetcode-cn.com/problems/valid-palindrome-ii
tags:
  - LeetCode
  - 字符串
  - 双指针
---

## 题目

给你一个字符串 `s`，**最多** 可以从中删除一个字符。请你判断 `s` 是否能成为回文字符串：如果能，返回 `true` ；否则，返回 `false` 。

## 题解

### C++

```cpp
class Solution {
public:
    bool validPalindrome(string s) {
        int l = 0, r = s.length() - 1;
        while(l < r){
            if(s[l] == s[r]){
                l++;
                r--;
            } else {
                return isPalindrome(s.substr(l+1, r - l)) | isPalindrome(s.substr(l, r - l));
            }
        }
        return true;
    }

    bool isPalindrome(string ss){
        int l = 0, r = ss.length() - 1;
        while(l < r){
            if(ss[l] != ss[r]){
                return false;
            }
            l++;
            r--;
        }
        return true;
    }
};
```

### Python

```python
class Solution:
    def validPalindrome(self, s: str) -> bool:
        def isPalindrome(ss):
            l, r = 0, len(ss) - 1
            while l < r:
                if ss[l] != ss[r]:
                    return False
                l += 1
                r -= 1
            return True
        l, r = 0, len(s) - 1
        while l < r:
            if s[l] == s[r]:
                l += 1
                r -= 1
            else:
                return isPalindrome(s[l+1:r+1]) or isPalindrome(s[l:r])
        return True
```

