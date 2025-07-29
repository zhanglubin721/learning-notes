##  **PriorityQueue 是什么？**

PriorityQueue 是 Java 中一个**基于优先级堆（最小堆）**实现的队列，位于 java.util 包中。它的主要特性是：

- 元素按照优先级排列，而**不是插入顺序**。
- 默认是**小顶堆**（最小堆）：头元素是最小的元素。
- 可以通过传入自定义的 Comparator 实现**大顶堆或其他自定义排序规则**。



## **底层结构**

底层使用一个 **动态数组 Object[] queue** 来存储元素，逻辑上是一个**完全二叉树**。

- 根节点是优先级最高（最小）的元素，位于 queue[0]。
- 对于任意索引 i：
  - 左子节点索引是 2 * i + 1
  - 右子节点索引是 2 * i + 2
  - 父节点索引是 ( i - 1 ) / 2



## **核心操作**

### offer(E e) / add(E e)- 插入元素

| **方法**   | **定义于**      | **行为（队列满时）**                      |
| ---------- | --------------- | ----------------------------------------- |
| add(E e)   | Collection 接口 | 插入失败时抛出异常：IllegalStateException |
| offer(E e) | Queue 接口      | 插入失败时返回 false，**不抛异常**        |

- PriorityQueue 是无界的：add 和 offer 没有区别。
- 如果你处理的是**容量受限的队列**（如 ArrayBlockingQueue），**优先使用 offer**，这样代码更安全（不会抛异常，便于控制失败情况）。
- 写队列类的通用逻辑时推荐使用 offer，更贴合 Queue 接口语义（“尝试入队”）。



元素插入到数组末尾，然后向上“**上浮**”以维护最小堆性质。

```java
private void siftUp(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```



### poll()- 弹出堆顶元素

- 移除并返回最小元素（queue[0]）。
- 将数组最后一个元素移到堆顶，然后“**下沉**”恢复堆性质。

```java
private void siftDown(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];
        int right = child + 1;
        if (right < size && comparator.compare((E)c, (E)queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E)c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}
```



### peek()-查看堆顶元素

- 返回堆顶（最小元素），**不删除**。
- 时间复杂度 O(1)。



## 构造函数

```java
// 默认小顶堆
PriorityQueue<Integer> pq = new PriorityQueue<>();

// 指定初始容量
PriorityQueue<Integer> pq = new PriorityQueue<>(100);

// 自定义 Comparator：实现大顶堆
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
```

- 如果使用构造函数 new PriorityQueue(Collection<? extends E> c)，它内部会直接对整个数组一次性建堆（使用 siftDown 从中间向前处理），时间复杂度是 O(n)。
- 优于逐个 offer() 插入的 O(n log n)。



## **注意事项**

1. **不支持 null 元素**，否则抛 NullPointerException。
2. 不是线程安全的，需要外部同步。
3. 元素必须具备**可比较性**，即：
   - 要么 E 实现了 Comparable 接口。
   - 要么构造时传入 Comparator。
4. 插入或删除不会维持整体排序，只有堆顶元素是最小值。



## 模拟LRU

```java
PriorityQueue<CacheItem> pq = new PriorityQueue<>(
    (a, b) -> Long.compare(a.lastAccessTime, b.lastAccessTime)
);
```

