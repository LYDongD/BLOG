### [19] Remove Nth Node From End of List


### 方法1：

* 1 单指针法，遍历整个链表，求链表长度, L
* 2 倒数第N个节点，即正数的第 L - N 个节点
* 3 再次遍历链表，找到第L-N-1个节点，删除下一个节点

缺点：需要遍历链表2次

### 方法2 

双指针法 -> 快慢指针

* 准备2个指针，快指针先走N步
* 快慢指针一起走，当快指针走到尾节点时，慢指针的next刚好是第N个节点

优点: 仅需要遍历链表一次

实现：

```

type ListNode struct {
    Val int
    Next *ListNode
}

func removeNthFromEnd(head *ListNode, n int) *ListNode {
    if head == nil {
        return head
    }

	//从哨兵节点开始往前走
    puppy := &ListNode{0, head}
    fastCursor, slowCursor := puppy, puppy
    i := 0
    
    //快指针先走N步
    for fastCursor != nil  {
        fastCursor = fastCursor.Next
        i++
        if i == n {
            break
        }
    }

    if fastCursor == nil {
        return nil
    }

	//快慢指针一起走
    for fastCursor.Next != nil {
        slowCursor = slowCursor.Next
        fastCursor = fastCursor.Next
    }

	//删除时区分一般节点和头节点
    if slowCursor.Next == head {
        head = head.Next
    }else {
        slowCursor.Next = slowCursor.Next.Next
    }

    puppy.Next = nil

    return head
}