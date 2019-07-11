## Remove Linked List Elements

### 问题

```
Remove all elements from a linked list of integers that have value val.

Example:

Input:  1->2->6->3->4->5->6, val = 6
Output: 1->2->3->4->5

```

### 思路

* 遍历单向链表，删除节点
* 特殊处理头节点

```
 public ListNode removeElements(ListNode head, int val) {
        ListNode cursor = head;
        ListNode lastCursor = null;
        while (cursor != null){

            ListNode nextCursor = cursor.next;

            //node to be removed
            if (cursor.val == val){

                //remove first node
                if (lastCursor == null){
                    head = cursor.next;
                }else {
                    lastCursor.next = cursor.next;
                }

                //ready for gc
                cursor.next = null;
            }else {
                lastCursor = cursor;
            }
            
            cursor = nextCursor;
        }

        return head;
    }

```

