## **一、HashSet**

### **1. 数据结构**

HashSet 是基于 **HashMap** 实现的，底层实际是将元素作为 HashMap 的 key，value 是一个固定的哨兵对象（PRESENT）。

```
private transient HashMap<E,Object> map;
private static final Object PRESENT = new Object();
```



### **2. 构造方法**

```
public HashSet() {
    map = new HashMap<>();
}

public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
```

构造方法直接用 HashMap 的构造器，容量是根据集合大小或默认值传入。



### **3. 添加元素**

```
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

底层调用 HashMap.put(k, v)，我们重点关注：



#### **HashMap put 流程（JDK8）：**

- 计算哈希值（扰动函数）：

```
int hash = hash(key); // 高位混合
```

- 定位桶位（数组下标）：

```
int index = (n - 1) & hash;
```

- 桶为空：直接插入
- 否则：链表/红黑树查找并更新或追加
- 插入后检查是否需要扩容（元素个数 >= 阈值）



### **4. 删除元素**

```
public boolean remove(Object o) {
    return map.remove(o) == PRESENT;
}
```

底层调用 HashMap.remove(key)：

- 定位桶位置
- 链表或树中查找目标 key
- 找到后断链或重排树结构



### **5. 查找是否包含**

```
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

底层就是调用 HashMap.containsKey(key)，查找过程如 put 一样。



## **二、LinkedHashSet**

### **1. 数据结构**

LinkedHashSet 是 HashSet 的子类，底层用的是 LinkedHashMap 作为支持，因此是 **有序的 HashSet（插入顺序）**。

```
public class LinkedHashSet<E> extends HashSet<E> {
    public LinkedHashSet() {
        super(new LinkedHashMap<>());
    }
}
```



### **2. LinkedHashMap 关键点**

继承自 HashMap，但重写了内部结构：

```
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
}
```

- 双向链表维护插入顺序（head→tail）

每次插入或删除都会调整链表位置。



### **3. 插入、删除、查找**

LinkedHashSet 所有方法行为基本与 HashSet 相同，只是由于 LinkedHashMap 的支持，使得元素遍历时是按插入顺序。



## **三、TreeSet**

### **1. 数据结构**

TreeSet 是基于 **红黑树** 实现的，底层由 TreeMap<E, Object> 支撑。

```
private transient NavigableMap<E,Object> m;
```



### **2. 构造函数**

```
public TreeSet() {
    this(new TreeMap<>());
}

public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}
```

核心是构造了一个 TreeMap，用于自动排序。



### **3. 插入元素**

```
public boolean add(E e) {
    return m.put(e, PRESENT) == null;
}
```

TreeMap.put(key, value) 的过程：

#### **TreeMap 插入流程：**

- 如果是第一个节点，则设为根节点
- 比较 key（使用 compareTo 或传入的 Comparator）
- 插入到对应子树
- 插入后做 **红黑树平衡调整（变色 + 旋转）**

红黑树高度大约为 O(log N)，查找/插入/删除性能稳定。



### **4. 查找 & 删除** 

```
public boolean remove(Object o) {
    return m.remove(o) == PRESENT;
}

public boolean contains(Object o) {
    return m.containsKey(o);
}
```

底层走 TreeMap 的红黑树逻辑。



### **5. 特殊功能**

TreeSet 实现了 NavigableSet 接口，支持范围操作：

```
treeSet.lower(e);   // 严格小于 e
treeSet.floor(e);   // 小于等于 e
treeSet.ceiling(e); // 大于等于 e
treeSet.higher(e);  // 严格大于 e
```



## **总结对比表**



| **特性**     | **HashSet**  | **LinkedHashSet** | **TreeSet**          |
| ------------ | ------------ | ----------------- | -------------------- |
| 底层结构     | HashMap      | LinkedHashMap     | TreeMap（红黑树）    |
| 元素顺序     | 无序         | 插入顺序          | 有序（自然或自定义） |
| 时间复杂度   | O(1)（均摊） | O(1)（均摊）      | O(log N)             |
| 是否线程安全 | 否           | 否                | 否                   |
| 支持范围操作 | 否           | 否                | 是（lower、floor等） |