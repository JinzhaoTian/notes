---
leetcode: LeetCode 105. Construct Binary Tree from Preorder and Inorder Traversal
difficulties: MEDIUM
link: https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。


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

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = preorder.size();
        for (int i = 0; i < n; i++) {
            index[inorder[i]] = i;
        }
        return build(preorder, inorder, 0, n - 1, 0, n - 1);
    }

    TreeNode* build(const vector<int>& preorder, const vector<int>& inorder, int pleft, int pright, int ileft, int iright) {
        if (pleft > pright)
            return nullptr;
        int proot = pleft, iroot = index[preorder[proot]];
        TreeNode* root = new TreeNode(preorder[proot]);
        int left_size = iroot - ileft;
        root -> left = build(preorder, inorder, pleft + 1, pleft + left_size, ileft, iroot - 1);
        root -> right = build(preorder, inorder, pleft + left_size + 1, pright, iroot + 1, iright);
        return root;
    }
};
```