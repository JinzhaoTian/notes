---
leetcode: LeetCode 206. Reverse Linked List
difficulties: EASY
link: https://leetcode-cn.com/problems/reverse-linked-list
tags:
  - LeetCode
  - Hot100
  - 链表
---

## 题目

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

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
    ListNode* reverseList(ListNode* head) {
        ListNode *p = head, *q = nullptr;
        while(p) {
            ListNode *t = p -> next;
            p -> next = q;
            q = p;
            p = t;
        }
        return q;
    }
};
```