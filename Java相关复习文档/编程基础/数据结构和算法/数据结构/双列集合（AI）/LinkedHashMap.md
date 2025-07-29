## Node结构

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after; // 维护双向链表
}
```

```
HashMap 桶位结构：

table[0] → Node(hash, key, value, next) → ...

LinkedHashMap 的新增结构：

每个 Node 再多两个字段：
before ← Node(hash, key, value, next) → after

形成一个双向链表，连接所有 entry
```

### **重点**

在 LinkedHashMap 中，它复用了 HashMap 的 table 数组和 Node 节点结构，但通过定义自己的 Entry<K,V> 类（继承自 HashMap.Node<K,V>），**在 Node 上额外加了 before 和 after 字段**，实现了一个双向链表。

- next 是用于哈希桶内的链式结构（解决哈希冲突用的）；
- before/after 是用于维护插入顺序或访问顺序的链表结构；
- head 和 tail 是维护顺序链表的首尾指针。

所以LinkedHashMap的成员变量相较于HashMap多了

```java
transient LinkedHashMap.Entry<K,V> head;
transient LinkedHashMap.Entry<K,V> tail;
final boolean accessOrder;
```

这三个用于维护链表头尾，方便链表快速操作



## put

逻辑与HashMap基本一致，只不过多了：

- 插入新键值对时，会将新节点加到链表尾部并且变更LinkedHashMap存储的tail。
- 如果覆盖已有 key 的值，不变顺序，除非 accessOrder=true 且你是通过 get 访问的。
- 如果重写了removeEldestEntry方法，则每次put都会调用该方法判断是否要清除链表的head节点



## get

- 如果 accessOrder=false（默认），就和HashMap完全一致；
- 如果 accessOrder=true，则额外会将访问当前的 key 的节点从链表中移除再挂到尾部，并且变更LinkedHashMap存储的tail，以实现 LRU 顺序。



## remove

除了原本HashMap的逻辑以外，还需要将删除的node从链表里截出，特殊情况下还需要改变LinkedHashMap存储的head和tail

```java
node.before.after = node.after;
node.after.before = node.before;
```



## resize

扩容逻辑与HashMap完全一致，因为是put会触发扩容，且扩容逻辑是在put方法的最末尾执行，所以在扩容期间完全不需要考虑链表，因为链表以及LinkedHashMap存储的head和tail根本不用动

##  

## LRU触发点

```java
LinkedHashMap<String, String> cache = new LinkedHashMap<>(16, 0.75f, true) {
  	@Override  
  	protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
        return size() > 100; // 超过100条自动移除最旧的
    }
};
```

如果你在子类中重写这个方法，就可以在每次 put 之后自动判断是否需要移除最旧的 entry，**这就是 LRU 缓存的基础逻辑**。
