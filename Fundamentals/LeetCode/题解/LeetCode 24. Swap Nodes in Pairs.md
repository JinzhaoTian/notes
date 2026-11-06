---
leetcode: LeetCode 24. Swap Nodes in Pairs
difficulties: MEDIUM
link: https://leetcode.cn/problems/swap-nodes-in-pairs
tags:
  - LeetCode
  - Hot100
  - 链表
---

## 题目

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即只能进行节点交换）。

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
    ListNode* swapPairs(ListNode* head) {
        ListNode* dummy = new ListNode(0);
        dummy -> next = head;
        ListNode* current = dummy;
        while (current -> next != nullptr && current -> next -> next != nullptr) {
            ListNode *p = current -> next, *q = current -> next -> next;
            current -> next = q;
            p -> next = q -> next;
            q -> next = p;
            current = p;
        }
        ListNode* ans = dummy -> next;
        delete dummy;
        return ans;
    }
};
```