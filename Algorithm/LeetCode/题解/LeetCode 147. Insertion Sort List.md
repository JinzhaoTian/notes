---
leetcode: LeetCode 147. Insertion Sort List
difficulties: MEDIUM
link: https://leetcode.cn/problems/insertion-sort-list
tags:
  - LeetCode
  - 链表
  - 排序
---

## 题目

给定单个链表的头 `head` ，使用插入排序对链表进行排序，并返回排序后链表的头。

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
    ListNode* insertionSortList(ListNode* head) {
        if (head == nullptr)
            return head;
        ListNode* dummy = new ListNode(-1);
        dummy -> next = head;
        ListNode *lastSorted = head, *current = head -> next;
        while (current != nullptr) {
            if (lastSorted -> val <= current -> val) {
                lastSorted = lastSorted -> next;
            } else {
                ListNode *p = dummy;
                while (p -> next -> val <= current -> val) {
                    p = p -> next;
                }
                lastSorted -> next = current -> next;
                current -> next = p -> next;
                p -> next = current;
            }
            current = lastSorted -> next;
        }
        ListNode * ans = dummy -> next;
        delete dummy;
        return ans;
    }
};
```