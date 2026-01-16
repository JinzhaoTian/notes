---
leetcode: LeetCode 94. Binary Tree Inorder Traversal
difficulties: EASY
link: https://leetcode.cn/problems/binary-tree-inorder-traversal
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给定一个二叉树的根节点 `root` ，返回它的中序遍历。

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
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> ans;
        inorder(ans, root);
        return ans;
    }

    void inorder(vector<int>& res, TreeNode* root) {
        if (!root)
            return;
        inorder(res, root -> left);
        res.push_back(root -> val);
        inorder(res, root -> right);
    }
};
```