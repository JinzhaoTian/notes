---
leetcode: LeetCode 345. Reverse Vowels of a String
difficulties: EASY
link: https://leetcode-cn.com/problems/reverse-vowels-of-a-string
tags:
  - LeetCode
  - 双指针
  - 字符串
---

## 题目

给你一个字符串 `s` ，仅反转字符串中的所有元音字母，并返回结果字符串。

## 分析

注意大小写。


## 题解

### C++

```cpp
class Solution {
public:
    string reverseVowels(string s) {
        int l = 0, r = s.length() - 1;
        while(l < r){
            while((l < r) && (!isVowels(s[l]))) l++;
            while((l < r) && (!isVowels(s[r]))) r--;
            swap(s[l], s[r]);
            l++;
            r--;
        }
        return s;
    }
    
    bool isVowels(char a) {
        char v[10] = {'a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U'};
        for(int i = 0; i < 10; i++){
            if (a == v[i])
                return true;
        }
        return false;
    }
};
```


### Python

```python
class Solution:
    def reverseVowels(self, s: str) -> str:
        vowels = 'aeiouAEIOU'
        s = list(s)
        l ,r = 0, len(s) - 1
        while l < r:
            while l < r and s[l] not in vowels:
                l += 1
            while l < r and s[r] not in vowels:
                r -= 1
            s[l], s[r] = s[r], s[l]
            l += 1
            r -= 1
        return ''.join(s)
```

