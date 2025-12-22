---
leetcode: LeetCode 23. Merge k Sorted Lists
difficulties: HARD
link: https://leetcode.cn/problems/merge-k-sorted-lists
tags:
  - LeetCode
  - 链表
  - Hot100
---

## 分析

给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

先看看边界条件：链表的数量 `k == lists.length`，`0 <= k <= 10^4`，最多1万个链表，每个链表的最大长度 `0 <= lists[i].length <= 500` 不超过5百个，并且链表中的数字个数 `lists[i].length` 的总和不超过 `10^4`。

### 暴力解

暴力的解法就是遍历每个链表的第一个元素，取一个最小值拿出来，然后再比较剩下的元素，比较暴力，存在重复比较，时间复杂度打不住。

还有个想法是链表们两两合并，关联题目是 [LeetCode 21. Merge Two Sorted Lists](LeetCode%2021.%20Merge%20Two%20Sorted%20Lists.md)，剩下的再两两合并，`logk * n`。

### 高效解

其实高效解就是上面两两合并，再两两合并得来的，只不过更精确的描述就是分治法。

## 题解

### C++

```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        return merge(lists, 0, lists.size() - 1);
    }

    ListNode* merge(vector<ListNode*>& lists, int l, int r){
        if (l == r)
            return lists[l];
        if (l > r)
            return nullptr;
        int m = (l + r)/2;
        return MergeTwoLists(merge(lists, l, m), merge(lists, m + 1, r));
    }

    ListNode* MergeTwoLists(ListNode* a, ListNode* b){
        ListNode* head = new ListNode(-1);

        ListNode* prev = head;
        while(a != nullptr && b != nullptr){
            if (a -> val <= b -> val){
                prev->next = a;
                a = a->next;
            }
            else{
                prev->next = b;
                b = b -> next;
            }
            prev = prev->next;
        }

        prev->next = a == nullptr? b: a;

        return head->next;
    }
};
```