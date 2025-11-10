[Leetcode 449. Serialize and Deserialize BST](https://leetcode-cn.com/problems/serialize-and-deserialize-bst/)

二叉搜索树的一个性质就是中序遍历会输出一个有序的结果。利用这个性质和前序遍历或者后序遍历就可以序列化或者还原一个二叉搜索树。

```
class Codec:
    def serialize(self, root):
        """
        Encodes a tree to a single string.
        """
        def postorder(root):
            return postorder(root.left) + postorder(root.right) + [root.val] if root else []
        return ' '.join(map(str, postorder(root)))

    def deserialize(self, data):
        """
        Decodes your encoded data to tree.
        """
        def helper(lower = float('-inf'), upper = float('inf')):
            if not data or data[-1] < lower or data[-1] > upper:
                return None
            
            val = data.pop()
            root = TreeNode(val)
            root.right = helper(val, upper)
            root.left = helper(lower, val)
            return root
        
        data = [int(x) for x in data.split(' ') if x]
        return helper()
        
```
