---
leetcode: LeetCode 114. Flatten Binary Tree to Linked List
difficulties: MEDIUM
link: https://leetcode.cn/problems/flatten-binary-tree-to-linked-list
tags:
  - LeetCode
  - Hot100
  - 二叉树
  - 链表
---

## 题目

给你二叉树的根结点 `root` ，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode` ，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null` 。
- 展开后的单链表应该与二叉树先序遍历顺序相同。


## 题解

### C++

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void flatten(TreeNode* root) {
        vector<TreeNode*> nodes;
        preorderTraversal(nodes, root);
        for (int i = 1; i < nodes.size(); ++i) {
            TreeNode *prev = nodes[i - 1], *curr = nodes[i];
            prev -> left = nullptr;
            prev -> right = curr;
        }
    }

    void preorderTraversal(vector<TreeNode*> &nodes, TreeNode* root) {
        if (root != nullptr) {
            nodes.push_back(root);
            preorderTraversal(nodes, root -> left);
            preorderTraversal(nodes, root -> right);
        }
    }
};
```