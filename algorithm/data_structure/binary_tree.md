## 二叉搜索树

BST是一个适合动态数据构建查询的数据结构，查询和写入的平均时间复杂度都是对数阶级。这篇文章，我们来实现一个二叉搜索树的构建(插入），删除，查找距离目标值最近的元素等算法，该算法的运用可参考leetcode 220题。

二叉树非常适合使用递归法进行处理，因为每个节点的处理思路是一致的。而对于值相同的节点，我们采用计数的方式处理，根据不同的场景，我们处理方式可以不同。如果某个节点值出现多次，则累计次数，而不是新增节点。

```
type TreeNode struct {
    left  *TreeNode //左孩子
    right *TreeNode //右孩子
    value int //节点值
    count int //计数
}

```

插入时，我们仅考虑一个最小子树，递归的插入左节点或右节点, 递归算法会找到第一个空节点进行插入后返回。

```
func insert(num int, root *TreeNode) *TreeNode {
    if root == nil {
        return &TreeNode{value: num, count: 1}
    }

    if root.value == num {
        root.count++
    } else if root.value > num {
        root.left = insert(num, root.left)
    } else {
        root.right = insert(num, root.right)
    }

    return root
}

```

删除时，先确定待删除节点，然后需要考虑3种情况：


* 左右孩子都为空： 直接将该节点置空
* 左孩子为空或右孩子为空：将不为空的另一个节点替换为当前节点
* 左右孩子都不为空：将左子树最小节点删除并替换为当前节点


```
func remove(num int, root *TreeNode) *TreeNode {
    if root == nil {
        return root
    }

	//当前节点为目标节点，进行删除
    if root.value == num {
        root.count--
        //只有count==0才移除该节点
        if root.count == 0 {
            if root.left == nil && root.right == nil {
                root = nil
            } else if root.left == nil {
                root = root.right
            } else if root.right == nil {
                root = root.left
            } else {
                node := removeLeftSmallest(root.left)
                node.left = root.left
                node.right = root.right
                root = node
            }
        }
        return root
    }

	//递归查找目标节点并删除
    if root.value > num {
        root.left = remove(num, root.left)
    } else {
        root.right = remove(num, root.right)
    }

    return root
}

//删除左子树最小节点
func removeLeftSmallest(root *TreeNode) *TreeNode {
    if root.left == nil {
        return root
    }

    node := root
    for {
        if node.left.left == nil {
            leftSmallest := node.left
            node.left = nil
            return leftSmallest
        }

        node = node.left
    }
}

```

查找时，这里我们实现一个查找离目标值最接近的两个节点，分别是较小和较大的两个节点，如果目标值本身是最小或最大值，只能找到一个节点。

例如，查找最接近的较大的节点，分成3种情况：

* 当前节点和目标值相同，返回节点值
* 当前节点值 > 目标值，设置为候选结果，并继续考察左子节点(可能有更接近的)
* 当前节点值 < 目标值，继续考察右子节点(需要找到比目标值大的）

```
func findAdjacentBiggerVal(num, adjacentNum int, node *TreeNode) int {

    if node == nil {
        return adjacentNum
    }

    if node.value == num {
        return node.value
    }

    if node.value > num {
        adjacentNum = node.value
        return findAdjacentBiggerVal(num, adjacentNum, node.left)
    }

    return findAdjacentBiggerVal(num, adjacentNum, node.right)
}

```

同理，查找较小节点的算法也是类似的：

```
func findAdjacentSmallerVal(num, adjacentNum int, node *TreeNode) int {
    if node == nil {
        return adjacentNum
    }

    if node.value == num {
        return adjacentNum
    }

    if node.value < num {
        adjacentNum = node.value
        return findAdjacentSmallerVal(num, adjacentNum, node.right)
    }

    return findAdjacentSmallerVal(num, adjacentNum, node.left)
}

```

综上，我们完成了BST的构建，删除和特殊查找，借由这3个算法，我们可以实现大小为k的滑动窗口，并基于该窗口完成特定的功能，例如leetcode 220:

判断k区间内是否存在2个数满足绝对值差小于t:

```
func containsNearbyAlmostDuplicate(nums []int, k int, t int) bool {
    if len(nums) <= 1 || k <= 0 || t < 0 {
        return false
    }

    var root *TreeNode
    for i, num := range nums {
        smaller := findAdjacentSmallerVal(num, num+1, root)
        if smaller <= num && abs(num-smaller) <= t {
            return true
        }
        bigger := findAdjacentBiggerVal(num, num-1, root)
        if bigger >= num && abs(num-bigger) <= t {
            return true
        }

        if i >= k {
            root = remove(nums[i-k], root)
        }

        root = insert(num, root)
    }

    return false
}

func abs(a int) int {
    if a >= 0 {
        return a
    }

    return -a
}

```



