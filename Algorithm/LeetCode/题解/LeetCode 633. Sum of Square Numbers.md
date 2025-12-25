---
leetcode: LeetCode 633. Sum of Square Numbers
difficulties: MEDIUM
link: https://leetcode-cn.com/problems/sum-of-square-numbers
tags:
  - LeetCode
  - 双指针
---

## 题目

给定一个非负整数 `c` ，你要判断是否存在两个整数 `a` 和 `b`，使得 `a2 + b2 = c` 。

## 题解

### C++

```cpp
class Solution {
public:
    bool judgeSquareSum(int c) {
        long long l = 0, r = sqrt(c) + 1;
        while(l <= r) {
            if(l * l + r * r > c) {
                r--;
            } else if (l * l + r * r < c){
                l++;
            } else {
                return true;
            }
        }
        return false;
    }
};
```


### Python

```python
class Solution:
    def judgeSquareSum(self, c: int) -> bool:
        l, r = 0, (int)(c**0.5) + 1
        while l <= r:
            if l*l + r*r > c:
                r -= 1
            elif l*l + r*r < c:
                l += 1
            else:
                return True
        return False
```

