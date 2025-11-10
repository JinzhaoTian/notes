[680. Valid Palindrome II](https://leetcode-cn.com/problems/valid-palindrome-ii/)

Python: 踉踉跄跄半天写出来，原理都明白，但是就是用代码表达不好，没什么可担心的，放心写就是了。

```
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

C++:

```
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
