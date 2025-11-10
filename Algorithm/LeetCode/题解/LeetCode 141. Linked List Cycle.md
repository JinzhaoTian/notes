[141. Linked List Cycle](https://leetcode-cn.com/problems/linked-list-cycle/)


Python:

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

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

C++:

```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
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
