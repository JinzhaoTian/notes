---
leetcode: LeetCode 141. Linked List Cycle
difficulties: EASY
link: https://leetcode-cn.com/problems/linked-list-cycle
tags:
  - LeetCode
  - 快慢指针
  - 链表
  - Hot100
  - 双指针
---

## 题目

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 `true` 。 否则，返回 `false` 。




## 题解


### C++

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode *p1 = head, *p2 = head;
        while(p1 && p2){
            if(!p2->next || !p2->next->next) break;
            p1 = p1->next;
            p2 = p2->next->next;
            if(p1 == p2) return true;
        }
        return false;
    }
};
```


### Python

```python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        p1 = head
        p2 = head
        while p1 and p2:
            if p2.next is None:
                return False
            if p2.next.next is None:
                return False
            p1 = p1.next
            p2 = p2.next.next
            if (p1 == p2):
                return True
        return False
```

