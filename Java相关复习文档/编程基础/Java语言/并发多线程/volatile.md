## **一、volatile 的作用**

volatile 有两个主要作用：

### **1. 可见性（visibility）**

当一个线程修改了一个 volatile 变量的值，**其他线程可以立即看到这个修改**。

**举例说明：**

```
volatile boolean flag = false;

Thread A:
    while (!flag) {
        // do something
    }

Thread B:
    flag = true;
```

如果 flag **没有使用 volatile 修饰**，那么线程 A 可能永远看不到线程 B 修改后的值，因为 A 可能缓存了旧值在自己的工作内存中。

加上 volatile 后，线程 A 会每次都从主内存读取最新值，确保看到了修改。



### **2. 禁止指令重排序（happens-before）**

使用 volatile 修饰的变量可以**禁止特定指令重排序**，避免出现重排序带来的线程安全问题。

```
// 示例：DCL（双重检查锁）模式
class Singleton {
    private static volatile Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // 1. 分配内存 2. 调用构造方法 3. 赋值引用
                }
            }
        }
        return instance;
    }
}
```

如果 instance 没有用 volatile 修饰，JVM 可能会重排序 1 → 3 → 2，导致另一个线程拿到未完全初始化的对象。



## **二、volatile 的底层实现原理**



### **1. JVM 层实现**

**使用内存屏障（Memory Barrier）**

JVM 在编译含 volatile 的代码时，会**在读写 volatile 变量的地方插入特殊的汇编指令（内存屏障）**，保证：

- **volatile 写之前**，JMM 会插入一个 **StoreStore 屏障**
- **volatile 写之后**，插入一个 **StoreLoad 屏障**
- **volatile 读之前**，插入一个 **LoadLoad 屏障**
- **volatile 读之后**，插入一个 **LoadStore 屏障**

这些屏障的目的是保证指令顺序和可见性。



### **2. Java 内存模型（JMM）视角**

JMM 为每个线程规定了一个“工作内存”，线程读取变量时是从工作内存读取，不一定实时从主内存读取。

但对于 volatile 修饰的变量：

- **线程在写入前必须先将该变量刷新到主内存**
- **线程在读取时必须从主内存中重新读取值**

这就保证了线程之间的 **可见性**。



### **3. CPU 层支持**

CPU 也有 **缓存一致性机制（如 MESI 协议）**，用于实现 volatile 的语义：

- 在 Intel x86 上，volatile 写操作对应 lock 前缀指令（如 lock add），会触发 **缓存一致性协议**，使得其他 CPU 核心失效其缓存行，强制重新从内存读取。
- 有些平台（如 ARM）不保证缓存一致性，JVM 会显式插入 **memory barrier（DMB/DSB/ISB）** 指令来保障 volatile 的语义。





| **点**                                | **内容**                          |
| ------------------------------------- | --------------------------------- |
| ✅ 可见性                              | 修改后的值对其他线程立即可见      |
| ✅ 禁止指令重排                        | 保证代码执行顺序                  |
| ❌ 不保证原子性                        | 如 ++ 操作要用 AtomicInteger 或锁 |
| 💡 JVM 插入内存屏障                    | 保证读写顺序和主内存交互          |
| 💡 CPU 层通过缓存一致性或 barrier 实现 | 不同架构下处理方式不同            |