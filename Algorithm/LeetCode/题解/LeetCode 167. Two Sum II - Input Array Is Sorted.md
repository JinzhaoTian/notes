[Two Sum II - Input Array Is Sorted](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)

Python:

```
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers)-1
        while l<r:
            if numbers[l] + numbers[r] > target:
                r -= 1
            elif numbers[l] + numbers[r] < target:
                l += 1
            else:
                break
        return [l+1, r+1]
```

C++:

```
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int l = 0, r =numbers.size() - 1;
        while(l < r){
            if(numbers[l] + numbers[r] > target){
                r--;
            } else if(numbers[l] + numbers[r] < target) {
                l++;
            } else {
                break;
            }
        }
        return {l + 1, r + 1};
    }
};
```
