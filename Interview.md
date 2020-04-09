### 1.1.4、LRU缓存机制

##### 题目：

运用你所掌握的数据结构，设计和实现一个  [LRU (最近最少使用) 缓存机制](https://baike.baidu.com/item/LRU)。它应该支持以下操作： 获取数据 `get` 和 写入数据 `put` 。

获取数据 `get(key)` - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 `put(key, value)` - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

**进阶:**

你是否可以在 **O(1)** 时间复杂度内完成这两种操作？

**示例:**

```
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```

##### 思路：

          由题目中要求的O(1)时间复杂度想到缓存可以想到用一个map来存储key、value结点，题目最近最少使用到的（缓存）放到最后，最新访问的（缓存）放到最前面，可以考虑用双端队列来实现，这样，这个map的key对应的是缓存的Key, value对应的是双端队列的一个节点，即队列的节点同时存在map的value中。
    
        这样，当新插入一个节点时，它应该在这个双端队列的队头处，同时把这个节点的key和这个节点put到map中保留下来。当LRU缓存队列容量达到最大又要插入新元素时，把队列的最后一个节点删除掉，同时在map中移除该节点对应的key。这个双端队列的数据结构为：

```
class DLinkedList{
    int key;
    int value;
    DLinkedList pre; //前一个节点
    DLinkedList next;//后一个节点
}
```

      这个map就用HashMap即可。

##### 解答代码为：

```
class DLinkedList{
    int key;
    int value;
    DLinkedList pre;
    DLinkedList next;
}

class LRUCache {  
    private DLinkedList head;
    private DLinkedList tail;
    private Map <Integer, DLinkedList> cache;
    private int count;
    private int capacity;
    
    public LRUCache(int capacity) {
        count = 0;
        this.capacity = capacity; 
        cache = new HashMap<Integer, DLinkedList>();
        head = new DLinkedList();
        tail = new DLinkedList();
        head.pre = null;
        head.next = tail;
        tail.next = null;
        tail.pre = head;
    }
    
    public void add(DLinkedList node){
        node.pre = head;
        node.next = head.next;
        head.next.pre = node;
        head.next = node;
    }
    
    public void remove(DLinkedList node){
        DLinkedList pre = node.pre;
        DLinkedList next = node.next;
        pre.next = next;
        next.pre = pre;
    }
    
    public void moveToHead(DLinkedList node){
        remove(node);
        add(node);
    }
    
    //删除队尾元素
    public DLinkedList popTail(){
        DLinkedList res = tail.pre;
        remove(res);
        return res;
    }
    
    public int get(int key) {
        DLinkedList node = cache.get(key);
        if(node == null){
            return -1;
        }
        moveToHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        DLinkedList node = cache.get(key);
        if(node != null){
            node.value = value;
            moveToHead(node);
            return;
        } 
        //如果队列数量大于容量，删掉队尾最近最少使用的那个元素
        if(++count > capacity){
           DLinkedList delNode = popTail();
           cache.remove(delNode.key);
        }
        node = new DLinkedList();
        node.key = key;
        node.value = value;
        add(node);
        cache.put(key,node);
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```