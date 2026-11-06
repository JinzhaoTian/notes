---
leetcode: LeetCode 457. Circular Array Loop
difficulties: MEDIUM
link: https://leetcode.cn/problems/circular-array-loop
tags:
  - LeetCode
  - 快慢指针
---

## 题目

存在一个不含 `0` 的**环形**数组 `nums` ，每个 `nums[i]` 都表示位于下标 `i` 的角色应该向前或向后移动的下标个数：
- 如果 `nums[i]` 是正数，**向前**（下标递增方向）移动 `nums[i]` 步
- 如果 `nums[i]` 是负数，**向后**（下标递减方向）移动 `abs(nums[i])` 步

因为数组是**环形**的，所以可以假设从最后一个元素向前移动一步会到达第一个元素，而第一个元素向后移动一步会到达最后一个元素。

数组中的**循环**由长度为 `k` 的下标序列 `seq` 标识：

- 遵循上述移动规则将导致一组重复下标序列 `seq[0] -> seq[1] -> ... -> seq[k - 1] -> seq[0] -> ...`
- 所有 `nums[seq[j]]` 应当不是**全正**就是**全负**
- `k > 1`

如果 `nums` 中存在循环，返回 `true` ；否则，返回 `false` 。

## 分析

这应该是一个模拟题，操作这个数组的下标进行移动。达成循环的条件就是向一个方向移动，可以回到最开始的位置。

假设最开始的位置是下标 `0` 的位置，假设当前 `nums[0]` 是 `3`，那么要向前移动 `3` 步，达到 `nums[0 + 3]` 的位置，若要达成循环，这个 `nums[0 + 3]` 只能为正数，继续往前走，

`seq[0] = 0`
`seq[1] = 3 = 0 + 3 = 0 + nums[0]`
`seq[2] = (3 + nums[3]) % n = (seq[1] + nums[seq[1]]) % n` 

`seq[i] = (seq[i - 1] + nums[seq[i - 1]]) % n`


### 快慢指针

看了一下题解，要把这个数组抽象成图中的点，每个点之间是可能相连，其实这也好理解，抽象成链表也行，只不过可能是一堆可能链接不上的散链，不管怎么说，只要有帮助就行。检测环的经典算法就是快慢指针，如果有环，走得慢的一定会被走的快的追上。


## 题解

### C++

```cpp
class Solution {
public:
    bool circularArrayLoop(vector<int>& nums) {
        int n = nums.size();
        auto next = [&](int v){
            return ((v + nums[v]) % n + n) % n; // 负数取模
        };
        for (int i = 0; i < n; i++){
            if (nums[i] == 0)
                continue;
            int slow = i, fast = next(i);
            while (nums[slow] * nums[fast] > 0 && nums[slow] * nums[next(fast)] > 0) {
                if (slow == fast) { // 跳出条件
                    if (slow == next(slow)) { // n 的整数倍
                        break;
                    } else {
                        return true;
                    }
                }
                slow = next(slow);
                fast = next(next(fast));
            }
        }
        return false;
    }
};
```

**再优化**：在实际代码中，我们无需新建一个数组记录每个点的访问情况，而只需要将原数组的对应元素置零即可（题目保证原数组中元素不为零）。遍历过程中，如果快慢指针相遇，或者移动方向改变，那么我们就停止遍历，并将快慢指针经过的点均置零即可。

```cpp
class Solution {
public:
    bool circularArrayLoop(vector<int>& nums) {
        int n = nums.size();
        auto next = [&](int v){
            return ((v + nums[v]) % n + n) % n; // 负数取模
        };
        for (int i = 0; i < n; i++){
            if (nums[i] == 0)
                continue;
            int slow = i, fast = next(i);
            while (nums[slow] * nums[fast] > 0 && nums[slow] * nums[next(fast)] > 0) {
                if (slow == fast) { // 跳出条件
                    if (slow == next(slow)) { // n 的整数倍
                        break;
                    } else {
                        return true;
                    }
                }
                slow = next(slow);
                fast = next(next(fast));
            }
            int add = i;
            while (nums[add] * nums[next(add)] > 0) {
                int tmp = add;
                add = next(add);
                nums[tmp] = 0;
            }
        }
        return false;
    }
};
```