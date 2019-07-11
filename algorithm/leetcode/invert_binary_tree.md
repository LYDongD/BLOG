## Invert Binary Tree

### 问题

```
Invert a binary tree.

Example:

Input:

     4
   /   \
  2     7
 / \   / \
1   3 6   9
Output:

     4
   /   \
  7     2
 / \   / \
9   6 3   1


```
### 思路

树相关问题优先考虑使用递归解决，把整棵树当成一颗最小子树来处理

1 对左右节点递归进行交换

```

public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;
        
        TreeNode tmp = root.left;
        root.left = invertTree(root.right);
        root.right = invertTree(tmp);
        
        return root;
 }


```
