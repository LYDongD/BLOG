## Link List Cycle

### 问题

```
Given a linked list, determine if it has a cycle in it.

```

### 思路1

* 遍历节点
* 用set缓存节点
* 如果节点已缓存，说明链表循环

该方法需要消耗额外的存储空间

```

 public static boolean hasCycle1(ListNode head){

        if (head == null) return false;

        Set<ListNode> nodeSet = new HashSet<>();

        ListNode current = head;

        //cycle will never come to null
        while (current != null){

            if (nodeSet.contains(current)){
                return true;
            }

            nodeSet.add(current);
            current = current.next;
        }

        return false;
    }

```

### 思路2 

* 准备两个快慢指针
* 快的每次走2步，慢的走1步
* 如果链表循环，则快慢指针最终汇合

该方法不需要消耗额外的存储空间

```

public static boolean hasCycle2(ListNode head){
        ListNode fast = head;
        ListNode slow = head;
        while (fast != null && fast.next != null){
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow){
                return true;
            }

        }
        return false;
    }

```

