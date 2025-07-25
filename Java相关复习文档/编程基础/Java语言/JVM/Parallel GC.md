## **总览：什么是 Parallel GC？**

> **Parallel GC（并行垃圾回收器）** 是 Java HotSpot 虚拟机中默认的垃圾回收器（JDK 8 默认），它的设计目标是最大化 **吞吐量（Throughput）**，即最小化 GC 所占应用程序运行时间的比例。

- 🔧 官方名称：-XX:+UseParallelGC
- 💡 特点：**多个线程并行执行 GC（尤其是年轻代）**，适用于 CPU 多核、对响应时间要求不敏感但追求高吞吐的场景（如批处理、后台服务等）。

## **一、内存结构（搭配 Parallel GC 时的 JVM 堆结构）**

Parallel GC 是一种 **分代收集器**，采用 **经典的两代结构（Young + Old）**，没有使用 G1 的 Region 划分方式。

### **堆划分如下：**

```
+------------------------+-------------------+
|      Young Generation  |   Old Generation  |
|     (新生代，复制算法)   	|  (老年代，标记-压缩) |
+--------+--------+------+-------------------+
|  Eden  | S0     | S1   |                   |
+--------+--------+------+-------------------+
```

- **Young（新生代）**：
  - 包含 Eden 区和两个 Survivor 区（S0、S1）
  - 新生对象首先分配在 Eden 区
  - 满了之后触发 **Minor GC**，并行清理 Eden，幸存对象进入 Survivor 或晋升到 Old
- **Old（老年代）**：
  - 长寿命对象存在于这里
  - 满了之后触发 **Major GC（Full GC）**
  - 使用**标记-整理（Mark-Compact）算法**

## 多线程

1️⃣ GC Roots 多线程扫描

- 启动多个 GC 线程并发扫描 GC Roots（栈、静态字段、JVM 结构等）
- 找出可达对象，加入每个线程的对象扫描队列
- 这是整个 GC 的起点

> ✅ 多线程进行，不像 Serial GC 单线程做

2️⃣ Eden → Survivor To 区 的对象复制

- 多线程遍历 Eden 中的对象
- 将存活对象复制到 Survivor To 区
- 复制过程中更新引用，并把新对象继续加入队列（递归处理）
- 引用处理用的是 ParScanClosure，是并行安全的

3️⃣ Survivor From → To 的对象复制

- 原本上轮 GC 后留在 From 区的“中年对象”
- 这轮 GC 中会被扫描判断是否还活着
- 如果还活着：
  - 年龄 +1（age）
  - 如果没达到晋升年龄，复制到 To 区
  - 如果达到晋升年龄，直接晋升到老年代

4️⃣ 晋升到老年代（Old）

- 有两种情况会触发对象晋升：

  1. 年龄到达晋升阈值（如 15）
  2. Survivor To 区空间不足，提前晋升（空间分配失败）

- 并发线程负责将这些对象复制到老年代合适的位置

  > 老年代使用 **分配指针 bump-the-pointer** 分配空间 + CAS 保证并发安全

## **二、垃圾扫描与回收过程**

### **1. Minor GC（年轻代回收）**

> 触发条件：Eden 区满了

#### **流程：**

1. **应用线程暂停（STW）**
2. 多个 GC 线程并行：
   - **标记**：从 GC Roots 出发（如栈、静态变量），遍历对象引用
   - **复制**：将 Eden 和 From Survivor 中存活的对象复制到 To Survivor 或晋升到老年代
3. 清空 Eden 和 From Survivor
4. **交换 Survivor 区角色（S0/S1）**
5. 应用线程恢复

#### **⚙算法：**

#### **复制算法（Copying）**

- 适用于大多数对象“朝生暮死”的新生代特性
- 减少内存碎片，但浪费 Survivor 空间

#### **年龄计数 & 晋升逻辑：**

- 对象每经历一次 Minor GC，年龄 +1
- 当年龄 ≥ MaxTenuringThreshold（默认 15）或 Survivor 区空间不足，就会**晋升到老年代**

### **2. Full GC（老年代回收）**

> 触发条件：老年代空间不足，或调用 System.gc()，或晋升失败（Promotion Failed）

**注意：Full GC 会收集整个堆（年轻代 + 老年代 + 元空间）**

#### **流程：**

1. 应用线程暂停（STW）

2. 多个 GC 线程并行进行老年代**标记-整理（Mark-Compact）**

   

   - 标记阶段：遍历 GC Roots，标记所有可达对象
   - 压缩阶段：将所有存活对象“压缩”到一边，清理碎片

#### **⚙算法：**

#### **标记-整理（Mark-Compact）**

- 不使用 CMS 的“标记-清除”，而是压缩内存，避免碎片
- 更适合内存整洁、高吞吐的场景

## **三、并行能力体现在哪？**

Parallel GC 的“并行”指的是：

| **阶段**               | **是否并行**                |
| ---------------------- | --------------------------- |
| **Minor GC**（年轻代） | ✅ 多线程并行标记、复制      |
| **Full GC**（老年代）  | ✅ 多线程并行标记和压缩      |
| **应用线程（STW）**    | ❗ GC 期间全部暂停，无法并发 |

> 与 CMS 不同，Parallel GC **不能并发回收老年代**，所有 GC 都是 STW（Stop-The-World）。

## **四、调优参数（部分重要配置）**



| **参数**                   | **含义**                                               |
| -------------------------- | ------------------------------------------------------ |
| -XX:+UseParallelGC         | 启用 Parallel GC（年轻代）                             |
| -XX:+UseParallelOldGC      | 启用 Parallel GC 回收老年代（Full GC）                 |
| -XX:MaxGCPauseMillis=n     | 期望的最大 GC 停顿时间（会影响 GC 线程数等）           |
| -XX:+UseAdaptiveSizePolicy | 启用自适应调整 Eden/S1/S0 比例、晋升阈值等（默认开启） |
| -XX:ParallelGCThreads=n    | 设置 GC 线程数（默认：CPU 核数）                       |

## **五、吞吐量优化目标**

Parallel GC 的核心目标是：

```
吞吐量 = 应用运行时间 / （应用运行时间 + GC 停顿时间）
```

举例：

- 应用运行 90 秒，GC 停顿 10 秒 → 吞吐量 = 90 / (90+10) = 90%
- Parallel GC 会尝试提升 Eden 大小、调整 Survivor 比例，以延迟 GC 的触发频率

## **六、与其他 GC 的对比**

| **特性** | **Parallel GC**                  | **CMS**      | **G1**   | **ZGC**            |
| -------- | -------------------------------- | ------------ | -------- | ------------------ |
| 吞吐量   | ✅ 高                             | 一般         | 高       | 高                 |
| 停顿时间 | ❌ 长                             | ✅ 较低       | ✅ 更低   | ✅ 超低（<10ms）    |
| 是否并发 | ❌ 否（STW）                      | ✅ 老年代并发 | ✅ 并发   | ✅ 全并发           |
| 内存碎片 | ✅ 低                             | ❌ 会产生     | ✅ 整理   | ✅ 整理             |
| 适用场景 | CPU 充足、对停顿不敏感的后台程序 | 偏交互式程序 | 混合场景 | 极端低延迟需求场景 |

**总结一句话**

> **Parallel GC 是基于 Stop-The-World + 多线程并行回收的高吞吐回收器**，适用于批处理、日志分析、后台计算等对响应时间不敏感但吞吐量要求高的系统。它的 GC 线程数可调，策略可自适应，但缺点是老年代 Full GC 时停顿时间较长，无法并发处理。

## GC Root

在 HotSpot JVM 中，GC Roots 是一组 JVM 运行时 **强引用持有者**，它们不会被 GC 本身回收，所有 GC 都以它们为起点。

**🔹 Parallel GC 中的 GC Roots 包括：**

| **Root 类型**                   | **来源**                          | **说明**                       |
| ------------------------------- | --------------------------------- | ------------------------------ |
| **Java 虚拟机栈（本地变量表）** | 每个线程的栈帧                    | 方法局部变量、参数中的对象引用 |
| **方法区中的类静态属性**        | 类加载器 -> 类元数据 -> 静态字段  | 类加载后常驻 JVM               |
| **方法区中的运行时常量池引用**  | ldc 加载的字符串、Class 引用等    |                                |
| **本地方法栈中的 JNI 引用**     | Native 方法中的 JNIEnv 指向的对象 |                                |
| **同步锁对象（monitor）**       | 处于锁状态的对象头指针            |                                |
| **线程对象自身**                | Java Thread 对象自身引用链        |                                |
| **JVM 系统类引用**              | ClassLoader、System 类等保留引用  |                                |

**🚀 三、Parallel GC 如何并行扫描 GC Roots？**

**🌟 关键点：GC Roots 扫描 + 并行标记对象图**

在 Parallel GC 中，垃圾回收器启动多个 **GC 工作线程（ParallelGCThreads）**，在 STW（Stop-The-World）状态下：

1. **GC Threads 拆分 GC Roots**
2. 各线程**并行扫描本地 GC Roots 段**
3. 将可达对象放入 “待处理队列”
4. 多线程递归扫描对象图（Breadth-First or Depth-First）

**🧩 四、深入机制解析（含结构）**

### **JVM 是如何标记和识别引用关系的？**

底层依赖以下几个关键结构：

| **结构**                               | **作用**                                              |
| -------------------------------------- | ----------------------------------------------------- |
| **OopMap（Object Pointer Map）**       | 在 GC 过程中记录哪些栈帧、寄存器、字段中含有对象引用  |
| **StackFrame**                         | 每个方法调用对应的 JVM 栈帧，包含局部变量表、操作数栈 |
| **ClassMetadata**                      | 包含类的静态字段、类型元信息                          |
| **对象头（MarkWord + Klass Pointer）** | 每个对象头部指向其类型和同步状态，也用于标记存活信息  |
| **Handle Table / JNI Table**           | Native 引用的跟踪工具                                 |
| **Card Table**                         | 辅助记录跨代引用（老年代 → 新生代）                   |

**🔁 五、标记流程细节（以 Minor GC 为例）**

```
1. GC Roots Collection（线程栈、类静态字段）
2. 并行 GC 线程遍历引用：
   - 读取对象
   - 检查对象是否已被标记
   - 若未标记，则标记并将其加入“扫描队列”
3. 使用任务队列 + work stealing 并发处理引用图
4. 完成可达对象图的遍历，未被访问到的对象为垃圾
```

**💡 为什么要“并行”？——任务队列 + work stealing**

每个 GC 线程持有一个本地任务队列，如果队列空了，会从其他线程偷任务（work stealing）来保持负载均衡。

这种设计提升了大对象图的扫描效率，是 Parallel GC 的核心优势。

**🔬 六、对象是否被标记的判断逻辑（HotSpot实现）**

对象是否已被访问/标记，JVM 会用位标记或 bitmap 标记结构：

- **位图（Mark Bitmap）**：代表某地址是否已经被标记过
- **对象头的标志位**：某些 GC 会直接在 MarkWord 中临时存标记位（如 CMS）
- **Card Table 位图**：用于判断老年代对象是否引用新生代

这些结构配合判断是否需要继续扫描引用。

**💡 七、与 CMS 或 G1 中的 GC Root 算法的差异**

| **方面**              | **Parallel GC**       | **CMS**               | **G1**                            |
| --------------------- | --------------------- | --------------------- | --------------------------------- |
| Root 扫描方式         | STW 且并行            | 初始标记并发 + STW    | 初始标记 STW + 并发 Root Scanning |
| 是否支持并发标记      | ❌ 否                  | ✅ 支持                | ✅ 支持                            |
| 写屏障/Remembered Set | ❌ 不依赖              | ✅ 强依赖              | ✅ 更复杂的 RSet                   |
| GC Root 分发          | 手动分配给 GC Threads | 并发遍历 Thread Roots | RSet 帮助定位跨 Region 的 Roots   |

**🔍 八、代码角度（HotSpot 实现）**

在 OpenJDK HotSpot 中：

- **GC Roots 枚举：**
  - Threads::oops_do() 处理线程局部变量
  - ClassLoaderDataGraph::roots_do() 处理类静态字段
  - Universe::oops_do() 处理常量池等全局对象
- **Parallel GC 标记线程：**
  - 位于 psMarkSweep.cpp、psScavenge.cpp
  - 使用 ParScanClosure 封装对对象字段的遍历
- **任务队列：**
  - ObjArrayTaskQueue 用于并行队列操作
  - WorkStealingQueue 实现任务窃取调度

**✅ 总结一句话**

> 在 Parallel GC 中，GC Root 算法通过 **Stop-The-World + 多线程并行遍历 GC Roots + 广度优先图遍历** 来完成存活对象的标记。这一过程高度依赖 JVM 内部结构如 OopMap、对象头、方法区静态引用等，在 GC 线程之间通过**任务队列和 Work Stealing** 保证并行效率，是其吞吐量优化的核心。

非常棒的问题，🌟你提到 -XX:+UseAdaptiveSizePolicy 与“轻量级锁升级重量级锁时会根据自旋次数与动态 max 的关系来判断”的对比，是非常专业且精准的联想。



它们确实都体现了 **HotSpot JVM 的自适应动态优化策略**，但实现机制略有不同。下面我们来深入讲解：



## **UseAdaptiveSizePolicy**

这个参数的作用是：

> **让 JVM 根据实际运行时的 GC 统计信息，动态调整堆中各区域（Eden、Survivor、Old）的大小，以及晋升阈值等，自动达到性能目标。**

- ✅ 默认开启（在使用 Parallel GC 时）
- 📌 依赖于 GC 期间收集的一系列监控指标
- 🎯 目标是优化：**吞吐量（Throughput）、响应时间（Pause Time）、堆大小利用率**

**🧬 二、它是怎么实现的？（核心：GC Ergonomics 模块）**

在 HotSpot 中，这部分逻辑是由 JVM 内部的 **GC Ergonomics 子系统** 实现的，文件主要分布在：

```
src/hotspot/share/gc/shared/gcPolicyCounters.cpp
src/hotspot/share/gc/shared/adaptiveSizePolicy.cpp
src/hotspot/share/gc/shared/gcPolicy.hpp
```

**☑️** 

AdaptiveSizePolicy类 —— 控制自适应逻辑的核心

它维护以下逻辑：

1. 每次 GC 后收集指标：
   - GC 停顿时间
   - Eden / Survivor 的使用率
   - 晋升速率（promotion rate）
   - 对象复制失败率
   - Survivor 填满比例
2. 与目标值做比较（如下所示）
3. 根据偏差进行动态调整：

```java
// 伪代码逻辑
if (avg_promotion_rate > target) {
    // Survivor 容量不足，提前晋升 → 调整 SurvivorRatio、降低 TenuringThreshold
}
if (avg_gc_pause_time > max_gc_pause_goal) {
    // 停顿过长，缩小 Eden 或调整 GC 线程数
}
```

**🎯 动态调整的参数包括：**

| **参数**                             | **作用**                    |
| ------------------------------------ | --------------------------- |
| **EdenRatio**                        | Eden 与 Survivor 之间的比例 |
| **SurvivorRatio**                    | S0/S1 与 Eden 的比例        |
| **MaxTenuringThreshold**             | 对象晋升年龄                |
| **HeapFreeRatio / MinHeapFreeRatio** | 控制堆扩容或缩容            |
| **YoungGenerationSizeIncrement**     | 增加年轻代大小步长          |
| **TenuringThreshold**                | 当前晋升阈值                |
| **ParallelGCThreads**                | GC 线程数（可选项）         |

**⚖️ 三、是否像锁自旋次数判断升级？**

你提到的这个对比是 **非常接近本质的：动态策略选择**。

| **方面**             | **自适应锁（轻量级 → 重量级）** | **自适应 GC 策略（Eden/S1/S0）** |
| -------------------- | ------------------------------- | -------------------------------- |
| 控制目标             | 避免浪费 CPU（自旋过长）        | 控制 GC 停顿/吞吐量目标          |
| 动态参数             | max_spins（可能会增长）         | promotion_rate, gc_pause_time    |
| 是否基于历史运行数据 | ✅ 是                            | ✅ 是                             |
| 是否硬编码静态阈值   | ❌ 动态计算                      | ❌ 动态优化                       |

所以两者都体现了 **HotSpot 中 runtime 自适应 heuristics（启发式）调控能力**，只是 **锁优化偏向响应延迟（低 latency）**，而 GC 自适应偏向 **吞吐+堆利用率综合调控**。

**🔍 四、一个示例：TenuringThreshold 是如何被调的？**

```java
if (survivor_space_used > target_fill_ratio) {
    // Survivor 填满了，不如早点晋升
    reduce TenuringThreshold;
} else if (survivor 空间富裕 && 晋升速率很高) {
    // 可以增加 TenuringThreshold 以减少老年代压力
    increase TenuringThreshold;
}
```

注意：TenuringThreshold 最终值由 MaxTenuringThreshold 限制，且调节步长也有调控。

**🧠 五、如何查看这些自适应变化？**

你可以通过以下方式在运行时观察：

```
-XX:+PrintAdaptiveSizePolicy
```

会输出类似日志：

```
Desired survivor size 2097152 bytes, new threshold 10 (max threshold 15)
Desired survivor size 1048576 bytes, new threshold 3 (max threshold 15)
```

还能配合：

```
-XX:+PrintGCDetails
-XX:+PrintTenuringDistribution
```

**✅ 总结一句话**

> -XX:+UseAdaptiveSizePolicy 启动了 **AdaptiveSizePolicy** 模块，它在每次 GC 后根据一系列运行时统计数据，**动态调节 Eden、Survivor 大小和晋升阈值**，目标是平衡 GC 停顿时间、吞吐量和内存利用率。这个机制和“锁自旋升级重量级锁”的动态判断非常类似——都体现了 HotSpot 的 runtime adaptive heuristics 机制。

核心代码：AdaptiveSizePolicy::compute_survivor_space_size_and_threshold()

这段函数是负责**计算 Survivor 区大小和晋升年龄（TenuringThreshold）**的核心逻辑之一。

```java
void AdaptiveSizePolicy::compute_survivor_space_size_and_threshold(...) {
    // 获取当前 GC 后 survivor 使用大小
    size_t used = _tenuring_threshold_data->accum_survivors();

    // 比较使用量与目标 survivor 填充比例
    if (used > desired_survivor_size) {
        // Survivor 空间压力大，减少晋升年龄
        if (_tenuring_threshold > 1) {
            _tenuring_threshold--; // 尽早晋升对象
        }
    } else {
        // Survivor 空间没满，可以多存几年再晋升
        if (_tenuring_threshold < _max_tenuring_threshold) {
            _tenuring_threshold++; // 延迟晋升
        }
    }

    // 动态调整 survivor size，以匹配晋升率
    if (avg_promotion_rate > threshold) {
        _survivor_size += survivor_growth_step;
    } else if (avg_promotion_rate < low_threshold) {
        _survivor_size -= survivor_shrink_step;
    }

    _survivor_size = clamp(_survivor_size, min_size, max_size);
}
```

| **行号**                  | **作用**                                      |
| ------------------------- | --------------------------------------------- |
| used = accum_survivors()  | 统计 GC 期间存活下来的对象总量（来自 S0→S1）  |
| desired_survivor_size     | JVM 预估目标 survivor 大小                    |
| tenuring_threshold-- / ++ | 动态调节晋升阈值，控制对象停留在 S 区的“年龄” |
| avg_promotion_rate        | 晋升速率，衡量 S 区无法容纳的对象             |
| survivor_size 调整        | 动态增长/缩小 survivor 空间大小               |
| clamp()                   | 控制调整范围在 min/max 限内，避免过激振荡     |

**背后的哲学**

JVM 这里使用了一种“负反馈调节机制”：

- 如果晋升过多，说明 Survivor 不够大 or 晋升太快 → 放缓晋升（调高阈值）、加大 Survivor
- 如果 Survivor 压力大，Survivor 空间**容量不足以容纳这些“中年对象”** → 缩短晋升阈值，及时送走

整个逻辑就像是一个**GC 自动调节控制器**，用历史 GC 采样数据实时调节堆布局。            