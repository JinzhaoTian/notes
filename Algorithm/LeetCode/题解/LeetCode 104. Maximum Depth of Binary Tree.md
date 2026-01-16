---
leetcode: LeetCode 104. Maximum Depth of Binary Tree
difficulties: EASY
link: https://leetcode.cn/problems/maximum-depth-of-binary-tree
tags:
  - LeetCode
  - Hot100
  - 二叉树
  - 深度优先搜索
---

## 题目

给定一个二叉树 `root` ，返回其最大深度。二叉树的最大深度是指从根节点到最远叶子节点的最长路径上的节点数。

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
    int maxDepth(TreeNode* root) {
        if (root == nullptr)
            return 0;
        return max(maxDepth(root -> left), maxDepth(root -> right)) + 1;
    }
};
```