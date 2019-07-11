## 二叉树

* 构造二叉查找树
* 实现深度优先和广度优先遍历
* 深度优先实现前序，中序和后序遍历


```

package go_algorithm

import (
    "fmt"
    "container/list"
)

type TreeNode struct {
    Val int
    Left *TreeNode
    Right *TreeNode
}

type BT struct {

}

//构建树
func (bt *BT) BuildTree(nums []int) *TreeNode{
    if len(nums) == 0 {
        return nil
    }

    var root *TreeNode
    for _, num := range nums {
        root = bt.insert(root, num)
    }

    return root
}

//BST insert
func (bt* BT) insert(root *TreeNode, num int) *TreeNode{
    if root == nil {
        return &TreeNode{num, nil, nil}
    }

    if root.Val > num {
        root.Left = bt.insert(root.Left, num)
    }else {
        root.Right = bt.insert(root.Right, num)
    }
    
    return root
}

//前序遍历
func (bt *BT) PreOrder(root *TreeNode) {
    if root == nil {
        return
    }
    fmt.Println(root.Val)
    bt.PreOrder(root.Left)
    bt.PreOrder(root.Right)
}

//中序遍历
func (bt *BT) InOrder(root *TreeNode) {
    if root == nil {
        return
    }

    bt.InOrder(root.Left)
    fmt.Println(root.Val)
    bt.InOrder(root.Right)
}


//后序遍历
func (bt *BT) postOrder(root *TreeNode) {
    if root == nil {
        return
    }

    bt.postOrder(root.Left)
    bt.postOrder(root.Right)
    fmt.Println(root.Val)
}

//层遍历
func (bt *BT) LevelOrder(root *TreeNode) {

    if root == nil {
        return 
    }

    queue := list.New()
    queue.PushBack(root)
    for queue.Len() > 0 {
        e := queue.Front()
        queue.Remove(e)
        root = e.Value.(*TreeNode)
        fmt.Println(root.Val)
        if root.Left != nil {
            queue.PushBack(root.Left)
        }

        if root.Right != nil {
            queue.PushBack(root.Right)
        }
    }

}

```
