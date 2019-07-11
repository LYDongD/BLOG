## LRU缓存

### 定义

least recently use 最近最少使用原则，即当缓存满时，删除最近最少使用的元素

### 实现

* hashMap存储key和节点，实现快速查找O1
* 双向链表组织节点，实现LRU

### 案例

* LinkedHashMap
* redis的Zset


### 代码

```
/**
 * 用hashMap + double linked list 实现LRU
 * hashMap support fast search O(1)
 * double linked list support LRU cache strategy
 */
public class LRUCache2<K, V> {

    private Node head;

    private Node tail;

    //cache limit
    private int limit;

    private Map<K, Node> hashMap;


    /**
     * 双向链表节点
     */
    private class Node {

        Node prev;
        Node next;
        K key;
        V value;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }

    public LRUCache2(int limit) {
        this.limit = limit;
        this.hashMap = new HashMap<>();
    }


    /**
     * 从双向链表尾部添加一个节点
     *
     * @param node 节点
     */
    private void addNode(Node node) {

        if (tail != null) {
            tail.next = node;
            node.prev = tail;
        }

        tail = node;

        if (head == null) {
            head = node;
        }
    }


    /**
     * 获取缓存
     *
     * @param key 缓存key
     * @return 缓存value
     */
    public V get(K key) {
        //get from hashMap
        Node node = hashMap.get(key);
        if (node == null) return null;
        refreshNode(node);
        return node.value;
    }

    public void put(K key, V value) {
        Node node = hashMap.get(key);
        if (node == null) {

            //limit check
            if (hashMap.size() >= limit) {
                //LRU delete cache
                K removedKey = remove(head);
                hashMap.remove(removedKey);
            }

            Node newNode = new Node(key, value);
            addNode(newNode);
            hashMap.put(key, newNode);
        } else {
            node.value = value;
            refreshNode(node);
        }
    }

    /**
     * refresh node
     *
     * @param node 节点
     */
    private void refreshNode(Node node) {
        if (tail == null) return;

        //remove and re add
        remove(node);
        addNode(node);
    }

    /**
     * remove node
     *
     * @param node 节点
     */
    private K remove(Node node) {

        if (node == head) {
            head = node.next;
        } else if (node == tail) {
            tail = node.prev;
        } else {
            node.prev.next = node.next;
            node.next.prev = node.prev;
        }

        return node.key;
    }

    public static void main(String args[]) {

        LRUCache2<Integer, String> cache = new LRUCache2(5);
        cache.put(1, "01");
        cache.put(2, "02");
        cache.put(3, "03");
        cache.put(4, "04");


        System.out.println(cache.get(1));
        System.out.println(cache.get(4));

        cache.put(5, "05");

        //get to the limit, trigger LRU removement
        cache.put(6, "06");

        //2 should be deleted
        System.out.println(cache.get(2));
        System.out.println(cache.get(6));
    }


}


```
