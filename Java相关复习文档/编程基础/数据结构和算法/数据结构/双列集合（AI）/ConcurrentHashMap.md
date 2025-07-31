## 扩容



- **数组：** Node<K,V>[] table，主存储结构，默认大小 16。
- **节点：** 每个桶是链表或红黑树结构（类似 HashMap）。
- **并发机制：** JDK 1.8 使用 **CAS + synchronized** 替代了老版本的 Segment 分段锁。
- **扩容时机：** 当前元素数量超过阈值（loadFactor * capacity）。





### **触发扩容的入口**

在 putVal() 方法中：

```java
sizeCtl < 0 //表示正在扩容

if (size >= threshold) {
    resize();
}
```

但是注意：ConcurrentHashMap 是懒加载+并发触发扩容的，**不是只有一个线程扩容**，而是多个线程一起帮忙迁移数据。



### 扩容核心流程详解：transfer()

#### 伪代码

```
开始扩容
│
├──► 设置 sizeCtl 为负数，代表“正在扩容”
│
├──► 分配新的 table（nextTable），准备把旧 table 的数据迁移过来
│
├──► 多个线程并发参与扩容（包括触发扩容的线程）
│     │
│     ├──► 每个线程 CAS 从 transferIndex 中领取一小段桶区间 [i, j)
│     │
│     ├──► 遍历 i 到 j，每个桶 bin 做以下迁移：
│     │     ├── 若 bin 为空：在新表对应位置设为 null
│     │     ├── 若 bin 是 ForwardingNode：表示已迁移，跳过
│     │     └── 否则：
│     │           ├── 计算 hash，把节点分成低位链/高位链
│     │           ├── 拷贝到新表的两个位置：index 和 index + oldCap
│     │           └── 原 bin 设为 ForwardingNode（标记已迁移）
│     │
│     ├──► 处理完当前领取区间后，再 CAS 获取新的桶段，直到 transferIndex <= 0
│     │
│     └──► 发现没有桶可领，但迁移未完成：
│            ├── 检查 finishing 是否为 true
│            │     ├── 否：设为 true，重进循环做最后的检查
│            │     └── 是：进入 synchronized 收尾代码块 ↓
│						 │
└────► synchronized 收尾阶段（只允许最后一个线程进入）
        │
        ├── 判断 sizeCtl < 0 且 transfererThreadCount == 1
        │      └── 是：这是最后一个线程
        │             ├── 设置 table = nextTable
        │             ├── 清空 nextTable
        │             └── 更新 sizeCtl 为新阈值
        │
        └── 迁移正式完成，扩容结束
```

最后的收尾阶段 JDK 8 和 JDK 9+有明显的区别，JDK 9+不再使用同步代码块而是使用CAS抢占来做最后的收尾



#### Step 1：构建新表 nextTable

```java
Node<K,V>[] oldTab = table;
int oldCap = oldTab.length;
int newCap = oldCap << 1;
Node<K,V>[] newTab = (Node<K,V>[]) new Node[newCap];
nextTable = newTab;
```

- 容量翻倍
- 把 nextTable 赋值后，不立即替换 table（并发协助迁移需要用到它）



#### Step 2：设置扩容标志 transferIndex

```java
transferIndex = oldCap; // 设置桶迁移任务的“分发起点”
```

- 用于分批迁移桶数据，多个线程会竞争该索引段来“领取任务”
- 每次迁移固定数量的桶（如 16 个）



#### Step 3：多线程协同迁移桶（helpTransfer）

```java
for (int i = transferIndex - 1; i >= 0; i--) {
    ...
}
```

所有参与扩容的线程会循环：

- 从 transferIndex 中**CAS 地领取桶的区间**（如 [128, 112)）
- 遍历这些桶，把数据复制到新表中
- 复制完之后用一个特殊的**占位符节点** ForwardingNode 替代旧桶



#### **Step 4：迁移桶的细节（链表、红黑树）**

每个桶迁移会分情况处理：



##### **情况1：空桶**

```
if (tab[i] == null) {
    newTab[i] = null;
}
```



##### **情况2：ForwardingNode（表示已经迁移）**

跳过，说明别的线程已经迁移



##### **情况3：普通链表节点**

- 把链表拆分成两个子链：
  - low 链（hash & oldCap == 0）→ 放在新表的 index
  - high 链（hash & oldCap != 0）→ 放在新表的 index + oldCap

```
int idx = e.hash & (newCap - 1); // 决定新位置
```



##### **情况4：红黑树节点**

- 拆分后如果节点数较少会转回链表
- 否则仍用红黑树结构插入新表中



#### **Step 5：设置 ForwardingNode**

```
tab[i] = new ForwardingNode<>(...);
```

表示当前桶已被迁移，别的线程可以跳过这个桶。

一个桶迁移完成后才把旧桶设置为ForwardingNode，这时候会通过find()跳转到 nextTable 继续查找

```java
static final class ForwardingNode<K,V> extends Node<K,V> {
  final Node<K,V>[] nextTable;
  Node<K,V> find(int h, Object k) {
  	//使用nextTable引用去已经迁移过的里面找
  }
}
//ForwardingNode重写了父类Node的find方法，这就实现了无感转接
```

ForwardingNode就是为了实现扩容期间依旧保持可读状态



#### **Step 6：迁移完成后替换 table**

所有桶都被迁移后：

```
table = nextTable;
nextTable = null;
```

- 用 table 替换原表
- 清除临时变量
- 结束扩容过程



## **多线程是如何协助扩容的？**

- 调用 put 的线程发现正在扩容，会调用 helpTransfer()
- 通过 transferIndex 分配桶区间任务，每次领一批桶
- 同步操作通过 **CAS** 和 **synchronized** 保证线程安全
- 避免了全表锁，**并发迁移效率高**



## **锁粒度？**

- **桶迁移：** 不锁整个 table，而是**逐桶加锁（synchronized）**

- **桶内节点复制：** 无需加锁，遍历老链构建新链即可（读是安全的）

- 以桶为最小单位，参与扩容的线程每次N个桶，处理这N个桶期间不会发生桶争抢，这N个桶成为一个段

  ```java
  //确定段大小的逻辑
  
  int stride = (n > MIN_TRANSFER_STRIDE) ? (n >>> 3) / NCPU : n;
  //意图：计算每个线程一次应该迁移多少个桶，称为“步长”stride。
  //解释：n >>> 3 等价于 n / 8（无符号右移3位）：只迁移 1/8 的桶总数作为并发迁移上限，避免线程争抢得太细。(n >>> 3) / NCPU：把这 1/8 的桶均分给 NCPU 个线程，得到每个线程一次处理的桶数。
  //这保证了：即使桶总数很多，也不会分得太细，避免线程切换成本高。如果桶总数较小，就直接给一个线程处理较大步长，减少线程争抢。
  
  if (stride < MIN_TRANSFER_STRIDE)
      stride = MIN_TRANSFER_STRIDE;
  //意图：设定一个最低步长，避免步长太小导致 CAS 竞争太激烈。比如如果分配给每个线程的桶太少（如2个桶），多个线程都疯狂争抢，就会有高频 CAS 失败 + 切换开销。这确保了每次迁移任务最小是 MIN_TRANSFER_STRIDE（默认16），提升效率。
  ```

  - n：旧表的长度（桶数量）。
  - MIN_TRANSFER_STRIDE：最小迁移步长（一般为16），防止每次迁移的桶数太小，线程切换频繁。
  - NCPU：当前机器的CPU核心数。





## **扩容的并发可见性**

- table、nextTable、transferIndex 都是 volatile，确保线程间可见性
- ForwardingNode 的使用也可以帮助读线程识别迁移完成



## ConcurrentHashMap 中 putVal 方法的循环逻辑说明

在 ConcurrentHashMap 的 putVal 方法中，核心逻辑大量使用了循环（主要是 for (;;) {} 死循环），目的是为了实现线程安全的插入、节点遍历与扩容协助等操作。每一轮循环尝试处理以下几种情况之一，直到真正完成插入任务为止才会通过 break 跳出循环：



- **CAS 尝试插入新节点（空桶）**：若当前桶位是 null，则尝试通过 CAS 插入新的节点。如果成功，说明当前线程完成了插入工作，可以跳出循环。
- **链表插入或树形结构处理**：若桶中已存在链表或红黑树结构，则通过加锁（链表）或 synchronized（红黑树）完成插入，并处理 key 冲突等逻辑。
- **ForwardingNode 情况（扩容中）**：若发现当前桶中为 ForwardingNode，说明当前 ConcurrentHashMap 正在扩容中，此时当前线程会主动协助扩容任务，并在完成自身责任范围的迁移后，重新计算新表位置再尝试插入。
- **扩容协助逻辑处理完后重新尝试插入**：由于扩容过程涉及两个表（旧表和新表），线程协助迁移期间并不能立即插入，只能等待部分迁移完成后再尝试插入正确的新表位置。

这种设计确保了并发写操作具有 **重试语义和阶段性协助扩容能力**，避免了加锁操作对整体写入性能的影响。



## **ConcurrentHashMap 中 get 方法如何支持扩容过程中的读操作**

ConcurrentHashMap 的 get 方法实现了在扩容过程中的无锁读支持，其核心机制依赖于桶节点的多态分发 —— 也就是不同类型的 Node 实例（普通节点、树节点、ForwardingNode）实现不同的 find 方法：

- **正常读操作流程**：get 方法首先根据 key 的哈希值计算出桶下标，然后直接读取 table[index]，如果是普通节点则调用其 find 方法（遍历链表或红黑树）寻找对应的 key。
- **ForwardingNode 情况**：当 table[index] 是一个 ForwardingNode 时，说明当前桶的内容已经被迁移至 nextTable 中。此时 ForwardingNode.find(...) 方法内部会根据原始 key 的 hash，再次在 nextTable 中定位新的桶位并继续查找。
- **迁移感知型读操作**：这种“从旧表跳转到新表”的机制是通过 ForwardingNode 提供的逻辑实现的，它允许读操作在线程无感知的情况下继续进行，**无需阻塞或等待扩容完成**。

这种读扩容解耦的机制使得 ConcurrentHashMap 即便在进行复杂的 rehash 和 table 替换过程中，也不会对读操作产生可见影响，从而保证了高并发读场景下的一致性与高性能。





## **总结重点**

| **环节**         | **特点**                                            |
| ---------------- | --------------------------------------------------- |
| 扩容触发点       | size >= threshold，put 时触发                       |
| 扩容方式         | 多线程协助迁移 buckets，每个线程负责一部分          |
| 新旧表结构       | nextTable 是临时新表，迁移完后再赋值给 table        |
| 协同机制         | CAS + 共享 transferIndex 实现线程任务分发           |
| 避免重复迁移     | 迁移完后标记为 ForwardingNode，其他线程看到直接跳过 |
| 并发安全保证     | volatile + synchronized（局部） + CAS               |
| 读取时支持迁移中 | 如果读线程读到 ForwardingNode，可以查 nextTable     |