---
leetcode: LeetCode 2. Add Two Numbers
difficulties: MEDIUM
link: https://leetcode.cn/problems/add-two-numbers
tags:
  - LeetCode
  - Hot100
  - 链表
---

## 题目

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *head = nullptr, *tail = nullptr;
        int c = 0;
        while (l1 || l2){
            int n1 = l1 ? l1 -> val: 0;
            int n2 = l2 ? l2 -> val: 0;
            int sum = n1 + n2 + c;
            if (!head) {
                head = tail = new ListNode(sum % 10);
            } else {
                tail -> next = new ListNode(sum % 10);
                tail = tail -> next;
            }
            c = sum / 10;
            if (l1) {
                l1 = l1 -> next;
            }
            if (l2) {
                l2 = l2 -> next;
            }
        }
        if (c > 0) {
            tail -> next = new ListNode(c);
        }
        return head;
    }
};
```