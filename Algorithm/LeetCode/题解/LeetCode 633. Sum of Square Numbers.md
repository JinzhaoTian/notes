[633. Sum of Square Numbers](https://leetcode-cn.com/problems/sum-of-square-numbers/)

Python:

```
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

C++:

```
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
