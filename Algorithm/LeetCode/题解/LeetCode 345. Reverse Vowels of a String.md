[345. Reverse Vowels of a String](https://leetcode-cn.com/problems/reverse-vowels-of-a-string/)


注意大小写。

Python:

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

C++: 太久没写C++了，感觉语法都忘了，数组的声明都忘了怎么写了，简直无语。

```
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
