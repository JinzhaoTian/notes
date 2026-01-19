---
leetcode: LeetCode 230. Kth Smallest Element in a BST
difficulties: MEDIUM
link: https://leetcode.cn/problems/kth-smallest-element-in-a-bst
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 小的元素（`k` 从 1 开始计数）。

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
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode*> stac;
        while (root != nullptr || stac.size() > 0) {
            while (root != nullptr) {
                stac.push(root);
                root = root -> left;
            }
            root = stac.top();
            stac.pop();
            k--;
            if (k == 0)
                break;
            root = root -> right;
        }
        return root -> val;
    }
};
```