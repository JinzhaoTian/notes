---
leetcode: LeetCode 437. Path Sum III
difficulties: MEDIUM
link: https://leetcode.cn/problems/path-sum-iii
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的路径的数目。

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
    int pathSum(TreeNode* root, int targetSum) {
        if (root == nullptr)
            return 0;
        
        int res = rootSum(root, targetSum);
        res += pathSum(root -> left, targetSum);
        res += pathSum(root -> right, targetSum);

        return res;
    }

    int rootSum(TreeNode* root, long long targetSum) {
        if (root == nullptr)
            return 0;

        int res = 0;
        if (root -> val == targetSum)
            res++;

        res += rootSum(root -> left, targetSum - root -> val);
        res += rootSum(root -> right, targetSum - root -> val);

        return res;
    }
};
```