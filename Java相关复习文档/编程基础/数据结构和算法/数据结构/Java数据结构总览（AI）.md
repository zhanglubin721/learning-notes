## **一、线性数据结构**

### 数组（Array）

- 特点：定长、支持随机访问（通过索引）、插入删除效率低。
- 示例：int[] arr = new int[10];



### 链表（LinkedList）

- 接口/实现类：java.util.LinkedList
- 特点：增删元素快（特别是头尾），但随机访问慢（O(n)）。
- 支持双向链表结构。
- 示例：LinkedList<String> list = new LinkedList<>();



### 栈（Stack）

- 类：java.util.Stack（已过时，推荐使用 Deque 替代）
- 推荐：ArrayDeque 实现 LIFO（后进先出）
- 示例：

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.pop();
```



### 队列（Queue）

- 接口：Queue<E>
- 常见实现：
  - 普通队列：LinkedList
  - 双端队列：ArrayDeque
  - 并发队列：ConcurrentLinkedQueue
- 示例：

```java
Queue<String> queue = new LinkedList<>();
queue.offer("a");
queue.poll();
```



### 优先队列（PriorityQueue）

- 类：java.util.PriorityQueue
- 特点：自动按优先级排序（最小堆）。
- 示例：PriorityQueue<Integer> pq = new PriorityQueue<>();



## **二、集合结构（无重复）**

### 集合（Set）

- 接口：Set<E>
- 不允许重复元素。

#### **常见实现：**

| **实现类**          | **特点**                                      |
| ------------------- | --------------------------------------------- |
| HashSet             | 基于哈希表，插入查找删除为 O(1)，无序         |
| LinkedHashSet       | 有插入顺序                                    |
| TreeSet             | 基于红黑树，自动排序，插入删除查找为 O(log n) |
| CopyOnWriteArraySet | 线程安全，适合读多写少场景（拷贝机制）        |



## **三、映射结构（键值对）**

### 映射（Map）

- 接口：Map<K, V>

#### **常见实现：**

| **实现类**        | **特点**                                                |
| ----------------- | ------------------------------------------------------- |
| HashMap           | 非线程安全，哈希表实现，插入查找删除为 O(1)             |
| LinkedHashMap     | 有插入顺序，支持 LRU 缓存淘汰（结合 removeEldestEntry） |
| TreeMap           | 红黑树实现，自动按 key 排序（O(log n)）                 |
| Hashtable         | 线程安全但已过时，建议用 ConcurrentHashMap              |
| ConcurrentHashMap | 支持高并发访问（分段锁或 CAS），适用于并发场景          |
| WeakHashMap       | Key 是弱引用，常用于缓存                                |
| IdentityHashMap   | 使用 == 比较 key，而不是 equals()                       |



## **四、并发数据结构（java.util.concurrent）**

| **类型** | **实现类**                                | **特点**               |
| -------- | ----------------------------------------- | ---------------------- |
| 并发队列 | ConcurrentLinkedQueue（非阻塞）           | 基于链表，线程安全     |
|          | LinkedBlockingQueue（阻塞）               | 支持生产者消费者模型   |
|          | ArrayBlockingQueue（固定容量阻塞队列）    |                        |
|          | DelayQueue、PriorityBlockingQueue         | 延迟队列、优先级队列   |
| 并发集合 | CopyOnWriteArrayList, CopyOnWriteArraySet | 写时复制，适合读多写少 |
| 并发 Map | ConcurrentHashMap                         | 线程安全，高并发场景   |



## **五、其他特殊结构**

| **名称**          | **类/接口**                     | **特点**                  |
| ----------------- | ------------------------------- | ------------------------- |
| 位图              | BitSet                          | 高效地处理布尔数组/标记位 |
| 多维数组          | int[][] matrix = new int[3][4]; | 二维表格类结构            |
| 枚举映射          | EnumMap、EnumSet                | 高效地用 Enum 做 key      |
| Deque（双端队列） | ArrayDeque、LinkedList          | 可当队列或栈使用          |



## **六、Java 中的常用集合框架图**

```
                Collection (接口)
                    |
   -------------------------------------
   |                |                 |
 List             Set              Queue
   |                |                 |
ArrayList       HashSet         LinkedList
LinkedList      TreeSet         ArrayDeque
Vector          LinkedHashSet   PriorityQueue
CopyOnWriteArrayList          ConcurrentLinkedQueue

                   Map (接口)
                     |
        -----------------------------
        |            |             |
     HashMap       TreeMap     ConcurrentHashMap
     LinkedHashMap WeakHashMap IdentityHashMap
```