---
leetcode: LeetCode 236. Lowest Common Ancestor of a Binary Tree
difficulties: MEDIUM
link: https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree
tags:
  - LeetCode
  - Hot100
---

## 题目

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

## 题解

### C++

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    unordered_map<int, TreeNode*> fa;
    unordered_map<int, bool> visited;

    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        fa[root -> val] = nullptr;
        dfs(root);
        while (p != nullptr) {
            visited[p -> val] = true;
            p = fa[p -> val];
        }
        while (q != nullptr) {
            if (visited[q -> val])
                return q;
            q = fa[q -> val];
        }
        return nullptr;
    }

    void dfs(TreeNode* root) {
        if (root -> left != nullptr) {
            fa[root -> left -> val] = root;
            dfs(root -> left);
        }
        if (root -> right != nullptr) {
            fa[root -> right -> val] = root;
            dfs(root -> right);
        }
    }
};
```