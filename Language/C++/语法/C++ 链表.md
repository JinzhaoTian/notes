
## 定义

```cpp
// LeetCode 标准定义
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

// 手动实现
class ListNode {
public:
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
```

## 基本操作

1. **创建链表**
```cpp
ListNode* createList(vector<int>& arr) {
    ListNode* dummy = new ListNode(0);
    ListNode* curr = dummy;
    for (int num : arr) {
        curr->next = new ListNode(num);
        curr = curr->next;
    }
    return dummy->next;
}
```

2. **遍历链表**
```cpp
void traverseList(ListNode* head) {
    ListNode* curr = head;
    while (curr != nullptr) {
        cout << curr->val << " ";
        curr = curr->next;
    }
}
```

3. **插入节点**
```cpp
void insertNode(ListNode* prev, int val) {
    ListNode* newNode = new ListNode(val);
    newNode->next = prev->next;
    prev->next = newNode;
}
```

4. **删除节点**
```cpp
void deleteNode(ListNode* prev) {
    if (prev->next == nullptr) return;
    ListNode* toDelete = prev->next;
    prev->next = prev->next->next;
    delete toDelete; // 注意释放内存
}
```

5. **反转链表**
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr) {
        ListNode* next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```