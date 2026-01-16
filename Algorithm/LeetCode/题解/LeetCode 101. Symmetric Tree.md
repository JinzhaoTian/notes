---
leetcode: LeetCode 101. Symmetric Tree
difficulties: EASY
link: https://leetcode.cn/problems/symmetric-tree
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给你一个二叉树的根节点 `root` ， 检查它是否轴对称。

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
    bool isSymmetric(TreeNode* root) {
        if (root == nullptr)
            return true;
        return check(root -> left, root -> right);
    }

    bool check(TreeNode* left, TreeNode* right) {
        if (left == nullptr && right == nullptr)
            return true;
        if (left == nullptr || right == nullptr)
            return false;
        return left -> val == right -> val && check(left -> left, right -> right) && check(left -> right, right -> left);
    }
};
```