---
leetcode: LeetCode 88. Merge Sorted Array
difficulties: EASY
link: https://leetcode-cn.com/problems/merge-sorted-array
tags:
  - LeetCode
---


## 题解

### C++

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int p1 = m - 1, p2 = n - 1, wp = m + n - 1;
        while(p1 >= 0 && p2 >= 0){
            if (nums1[p1] >= nums2[p2]){
                nums1[wp] = nums1[p1];
                p1--;
                wp--;
            } else {
                nums1[wp] = nums2[p2];
                p2--;
                wp--;
            }
        }
        while (p1 >= 0){
            nums1[wp] = nums1[p1];
            p1--;
            wp--;
        }
        while (p2 >= 0){
            nums1[wp] = nums2[p2];
            p2--;
            wp--;
        }
    }
};
```


### Python

```python
class Solution:
    def merge(self, nums1: List[int], m: int, nums2: List[int], n: int) -> None:
        """
        Do not return anything, modify nums1 in-place instead.
        """
        p1, p2, wp = m - 1, n - 1, m + n - 1
        while p1 >=0 and p2 >= 0:
            if nums1[p1] >= nums2[p2]:
                nums1[wp] = nums1[p1]
                p1 -= 1
                wp -= 1
            else:
                nums1[wp] = nums2[p2]
                p2 -= 1
                wp -= 1
        while p1 >= 0:
            nums1[wp] = nums1[p1]
            p1 -= 1
            wp -= 1
        while p2 >= 0:
            nums1[wp] = nums2[p2]
            p2 -= 1
            wp -= 1        
```

