---
leetcode: LeetCode 199. Binary Tree Right Side View
difficulties: MEDIUM
link: https://leetcode.cn/problems/binary-tree-right-side-view
tags:
  - LeetCode
  - Hot100
  - 二叉树
---

## 题目

给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

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
    vector<int> rightSideView(TreeNode* root) {
        unordered_map<int, int> rightValues;

        int max_depth = -1;

        queue<TreeNode*> nodeQueue;
        queue<int> depthQueue;

        nodeQueue.push(root);
        depthQueue.push(0);

        while (!nodeQueue.empty()) {
            TreeNode* node = nodeQueue.front();
            nodeQueue.pop();
            int depth = depthQueue.front();
            depthQueue.pop();

            if (node != nullptr) {
                max_depth = max(max_depth, depth);
                rightValues[depth] = node -> val;

                nodeQueue.push(node -> left);
                nodeQueue.push(node -> right);
                depthQueue.push(depth + 1);
                depthQueue.push(depth + 1);
            }
        }

        vector<int> rightView;
        for (int depth = 0; depth <= max_depth; ++depth){
            rightView.push_back(rightValues[depth]);
        }

        return rightView;
    }


};
```