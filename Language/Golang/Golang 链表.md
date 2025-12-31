
## 定义

```go
// LeetCode 标准定义
type ListNode struct {
    Val  int
    Next *ListNode
}

// 构造器函数
func NewListNode(val int) *ListNode {
    return &ListNode{Val: val}
}
```

## 操作


1. **创建链表**
```go
func createList(arr []int) *ListNode {
    dummy := &ListNode{}
    curr := dummy
    for _, num := range arr {
        curr.Next = &ListNode{Val: num}
        curr = curr.Next
    }
    return dummy.Next
}
```

2. **遍历链表**
```go
func traverseList(head *ListNode) {
    for curr := head; curr != nil; curr = curr.Next {
        fmt.Print(curr.Val, " ")
    }
}
```

3. **反转链表**
```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        prev = curr
        curr = next
    }
    return prev
}
```

4. **判断回文链表**
```go
func isPalindrome(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return true
    }
    
    // 找中点
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
    }
    
    // 反转后半部分
    var prev *ListNode
    curr := slow
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        prev = curr
        curr = next
    }
    
    // 比较
    left, right := head, prev
    for right != nil {
        if left.Val != right.Val {
            return false
        }
        left = left.Next
        right = right.Next
    }
    return true
}
```