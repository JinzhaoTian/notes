---
leetcode: LeetCode 19. Remove Nth Node From End of List
difficulties: MEDIUM
link: https://leetcode.cn/problems/remove-nth-node-from-end-of-list
tags:
  - LeetCode
  - Hot100
  - 链表
---

## 题目

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

## 题解

### C++

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummy = new ListNode(0, head);
        int length = getLength(head);
        ListNode *current = dummy;
        for (int i = 1; i < length - n + 1; ++i) {
            current = current -> next;
        }
        current -> next = current -> next -> next;
        ListNode* ans = dummy -> next;
        delete dummy;
        return ans;
    }

    int getLength(ListNode* head) {
        int length = 0;
        while (head) {
            length++;
            head = head -> next;
        }
        return length;
    }
};
```