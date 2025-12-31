## 定义

```typescript
// TypeScript
class ListNode {
    val: number;
    next: ListNode | null;
    constructor(val?: number, next?: ListNode | null) {
        this.val = (val === undefined ? 0 : val);
        this.next = (next === undefined ? null : next);
    }
}

// JavaScript
class ListNode {
    constructor(val, next = null) {
        this.val = val;
        this.next = next;
    }
}
```
## 操作

1. **创建链表**
```typescript
function createList(arr: number[]): ListNode | null {
    const dummy = new ListNode(0);
    let curr = dummy;
    for (const num of arr) {
        curr.next = new ListNode(num);
        curr = curr.next;
    }
    return dummy.next;
}
```

2. **遍历链表**
```typescript
function traverseList(head: ListNode | null): void {
    let curr = head;
    while (curr !== null) {
        console.log(curr.val);
        curr = curr.next;
    }
}
```

3. **快慢指针找中点**
```typescript
function findMiddle(head: ListNode | null): ListNode | null {
    let slow = head;
    let fast = head;
    while (fast && fast.next) {
        slow = slow!.next;
        fast = fast.next.next;
    }
    return slow;
}
```

4. **检测环**
```typescript
function hasCycle(head: ListNode | null): boolean {
    let slow = head;
    let fast = head;
    while (fast && fast.next) {
        slow = slow!.next;
        fast = fast.next.next;
        if (slow === fast) return true;
    }
    return false;
}
```