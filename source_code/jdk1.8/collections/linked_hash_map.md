## LinkedHashMap

### 结构

HashMap + 双向链表

* HashMap实现读写效率接近O(1)
* 双向链表可对元素进行排序，依据是插入或访问的先后顺序
* 借助双向链表，可实现FIFO，LRU缓存等结构

### 应用

FIFO， 默认排序, 根据插入顺序

```

       LinkedHashMap<Integer, String> linkedHashMap = new LinkedHashMap<>();
        for (int i = 0; i < 10; i++) {
            linkedHashMap.put(i, i + "");
        }
        linkedHashMap.get(4);

        for (Map.Entry entry : linkedHashMap.entrySet()) {
            System.out.println(entry.getKey());
        }
    }

```

access order， 考虑元素读取，将最近读取的元素放到末尾

* 设置access flag = true

```
public static void accessOrderTest () {

        float loadFactor = (float) 0.7;
        Map<Integer, String> linkedHashMap = new LinkedHashMap<>(16, loadFactor, true);
        for (int i = 0; i < 10; i++) {
            linkedHashMap.put(i, i + "");
        }
        linkedHashMap.get(4);
        linkedHashMap.get(5);

        for (Map.Entry entry : linkedHashMap.entrySet()) {
            System.out.println(entry.getKey());
        }
    }

```

LRU， 实现容量有限的缓存，按照最近最少使用规则删除元素

* 需要重写removeEldestEntry方法

```
 public static void LRU() {
        Map<Integer, String> linkedHashMap = new LinkedHashMap<Integer, String>(16, (float)0.7, true){
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, String> eldest){
                return size() > 3;
            }
        };

        linkedHashMap.put(1,"1");
        linkedHashMap.put(2,"1");
        linkedHashMap.put(3,"3");
        linkedHashMap.get(1);
        linkedHashMap.put(4,"3");


        for (Map.Entry entry : linkedHashMap.entrySet()) {
            System.out.println(entry.getKey());
        }

    }

```

### 原理

> LRU实现中，每次访问节点都把节点转移到双向链表的尾部

每次调用get方法访问元素，如果accessFlag=true，则调整当前节点在双向链表中的位置

```
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder) 
            afterNodeAccess(e);
        return e.value;
    }

```

将当前节点移动到双向链表的结尾, 具体操作为，先移除该节点，再将该节点插入到尾部

* 注意当节点是头节点或尾节点时，需要特殊处理

```
void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        //如果访问节点本身是尾节点，则不需要处理，否则需要将它移动到双向链表的尾部
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;

            //删除
            p.after = null;
            if (b == null) //如果是头节点，则让头指针指向下一个节点
                head = a;
            else //移除当前节点
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;

            //添加到尾部，此刻last -> tail
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }

            //更新tail
            tail = p;
            ++modCount;
        }
    }

```

> LRU实现中，每次插入，都要判断是否需要删除队头元素(双向链表的头结点)

判断方法是，removeEldestEntry，因此可重写该方法定义LRU删除缓存的条件

```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }

```



