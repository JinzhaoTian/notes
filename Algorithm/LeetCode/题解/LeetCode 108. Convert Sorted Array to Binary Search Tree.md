---
leetcode: LeetCode 108. Convert Sorted Array to Binary Search Tree
difficulties: EASY
link: https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给你一个整数数组 `nums` ，其中元素已经按升序排列，请你将其转换为一棵平衡二叉搜索树。

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
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return build(nums, 0, nums.size() - 1);
    }

    TreeNode* build(const vector<int>& nums, int left, int right) {
        if (left > right)
            return nullptr;
        
        int mid = (left + right) / 2;

        TreeNode* root = new TreeNode(nums[mid]);
        root -> left = build(nums, left, mid - 1);
        root -> right = build(nums, mid + 1, right);
        return root;
    }
};
```