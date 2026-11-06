---
leetcode: LeetCode 234. Palindrome Linked List
difficulties: EASY
link: https://leetcode.cn/problems/palindrome-linked-list
tags:
  - LeetCode
  - Hot100
  - 链表
---

## 题目

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false`。

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
    ListNode* front;

    bool isPalindrome(ListNode* head) {
        front = head;
        return recursiveCheck(head);
    }

    bool recursiveCheck(ListNode* current) {
        if (current != nullptr) {
            if (!recursiveCheck(current -> next)) {
                return false;
            }
            if (current -> val != front -> val) {
                return false;
            }
            front = front -> next;
        }
        return true;
    }
};
```