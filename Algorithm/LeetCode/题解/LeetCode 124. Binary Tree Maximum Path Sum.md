---
leetcode: LeetCode 124. Binary Tree Maximum Path Sum
difficulties: HARD
link: https://leetcode.cn/problems/binary-tree-maximum-path-sum
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

二叉树中的**路径**被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中至多出现一次。该路径至少包含一个节点，且不一定经过根节点。

**路径和**是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其**最大路径和** 。

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
    int ans = INT_MIN;

    int maxPathSum(TreeNode* root) {
        maxGain(root);
        return ans;
    }

    int maxGain(TreeNode* root) {
        if (root == nullptr)
            return 0;

        int leftGain = max(maxGain(root -> left), 0), rightGain = max(maxGain(root -> right), 0);
        int gain = root -> val + leftGain + rightGain;
        
        ans = max(ans, gain);

        return root -> val + max(leftGain, rightGain);
    }
};
```