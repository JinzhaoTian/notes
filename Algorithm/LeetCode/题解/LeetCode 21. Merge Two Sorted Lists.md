---
leetcode: LeetCode 21. Merge Two Sorted Lists
difficulties: MEDIUM
link: https://leetcode.cn/problems/merge-two-sorted-lists
tags:
  - LeetCode
---

## 分析

将两个升序链表合并为一个新的升序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

其实这就是基础的链表操作。


## 题解

### C++

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode* preHead = new ListNode(-1);

        ListNode* prev = preHead;
        while (l1 != nullptr && l2 != nullptr) {
            if (l1->val < l2->val) {
                prev->next = l1;
                l1 = l1->next;
            } else {
                prev->next = l2;
                l2 = l2->next;
            }
            prev = prev->next;
        }
        
        prev->next = l1 == nullptr ? l2 : l1;

        return preHead->next;
    }
};
```