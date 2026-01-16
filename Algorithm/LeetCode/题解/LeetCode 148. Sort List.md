---
leetcode: LeetCode 148. Sort List
difficulties: MEDIUM
link: https://leetcode.cn/problems/sort-list
tags:
  - LeetCode
  - Hot100
  - 链表
---

## 题目

给你链表的头结点 `head` ，请将其按升序排列并返回排序后的链表。

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
    ListNode* sortList(ListNode* head) {
        if (head == nullptr || head -> next == nullptr)
            return head;
        
        ListNode* mid = findMiddle(head);
        ListNode* rightHead = mid -> next;
        mid -> next = nullptr;

        ListNode* left = sortList(head);
        ListNode* right = sortList(rightHead);

        return mergeTwoLists(left, right);
    }

    ListNode* findMiddle(ListNode* head) {
        ListNode *slow = head, *fast = head -> next;
        while (fast != nullptr && fast -> next != nullptr) {
            slow = slow -> next;
            fast = fast -> next -> next;
        }
        return slow;
    }

    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode* dummy = new ListNode(-1);
        ListNode* current = dummy;

        while (l1 != nullptr && l2 != nullptr) {
            if (l1 -> val < l2 -> val) {
                current -> next = l1;
                l1 = l1 -> next;
            } else {
                current -> next = l2;
                l2 = l2 -> next;
            }
            current = current -> next;
        }
        if (l1 != nullptr) 
            current -> next = l1;
        if (l2 != nullptr)
            current -> next = l2;

        return dummy -> next;
    }
};
```