---
leetcode: LeetCode 106. Construct Binary Tree from Inorder and Postorder Traversal
difficulties: MEDIUM
link: https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给定两个整数数组 `inorder` 和 `postorder` ，其中 `inorder` 是二叉树的中序遍历，`postorder` 是同一棵树的后序遍历，请你构造并返回这颗二叉树。

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
    unordered_map<int, int> index;

    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        int n = inorder.size();
        for (int i = 0; i < n; ++i) {
            index[inorder[i]] = i;
        }
        return build(inorder, postorder, 0, n - 1, 0, n - 1);
    }

    TreeNode* build(const vector<int>& inorder, const vector<int>& postorder, int in_left, int in_right, int post_left, int post_right) {
        if (in_left > in_right)
            return nullptr;
        int post_root = post_right, in_root = index[postorder[post_root]];
        TreeNode* root = new TreeNode(postorder[post_root]);
        int left_size = in_root - in_left;
        root -> left = build(inorder, postorder, in_left, in_root - 1, post_left, post_left + left_size - 1);
        root -> right = build(inorder, postorder, in_root + 1, in_right, post_left + left_size, post_right - 1);
        return root;
    }
};
```