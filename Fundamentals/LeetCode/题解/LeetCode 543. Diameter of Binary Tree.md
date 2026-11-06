---
leetcode: LeetCode 543. Diameter of Binary Tree
difficulties: EASY
link: https://leetcode.cn/problems/diameter-of-binary-tree
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给你一棵二叉树的根节点，返回该树的 **直径** 。

二叉树的 **直径** 是指树中任意两个节点之间最长路径的 **长度** 。这条路径可能经过也可能不经过根节点 `root` 。

两节点之间路径的 **长度** 由它们之间边数表示。

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
    int ans = 0;
    
    int diameterOfBinaryTree(TreeNode* root) {
        ans = 1;
        dfs(root);
        return ans - 1;
    }

    int dfs(TreeNode* root) {
        if (root == nullptr)
            return 0;
        int l = dfs(root -> left);
        int r = dfs(root -> right);
        ans = max(ans, l + r + 1);
        return max(l, r) + 1;
    }
};
```