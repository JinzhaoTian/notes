## 定义

```csharp
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val = 0, ListNode next = null) {
        this.val = val;
        this.next = next;
    }
}

// 或者使用 LeetCode 标准定义
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int x) { val = x; }
}
```

### 操作

1. **创建链表**
```csharp
public ListNode CreateList(int[] arr) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    foreach (int num in arr) {
        curr.next = new ListNode(num);
        curr = curr.next;
    }
    return dummy.next;
}
```

2. **合并两个有序链表**
```csharp
public ListNode MergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            curr.next = l1;
            l1 = l1.next;
        } else {
            curr.next = l2;
            l2 = l2.next;
        }
        curr = curr.next;
    }
    
    curr.next = l1 ?? l2;
    return dummy.next;
}
```

3. **删除倒数第 N 个节点**
```csharp
public ListNode RemoveNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0, head);
    ListNode first = dummy;
    ListNode second = dummy;
    
    for (int i = 0; i <= n; i++) {
        first = first.next;
    }
    
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    
    second.next = second.next.next;
    return dummy.next;
}
```