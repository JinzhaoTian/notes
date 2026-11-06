---
leetcode: LeetCode 98. Validate Binary Search Tree
difficulties: MEDIUM
link: https://leetcode.cn/problems/validate-binary-search-tree
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题解

给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

有效二叉搜索树定义如下：
- 节点的左子树只包含 **严格小于** 当前节点的数。
- 节点的右子树只包含 **严格大于** 当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。


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
    bool isValidBST(TreeNode* root) {
        return check(root, LONG_MIN, LONG_MAX);
    }

    bool check(TreeNode* root, long long lower, long long upper) {
        if (root == nullptr)
            return true;
        if (root -> val <= lower || root -> val >= upper)
            return false;
        return check(root -> left, lower, root -> val) && check(root -> right, root -> val, upper);
    }
};
```