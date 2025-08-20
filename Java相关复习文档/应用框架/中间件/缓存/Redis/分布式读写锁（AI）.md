## **一、设计目标和背景**



- **可重入性**：支持单个线程多次获取读锁或写锁。

- **读写语义**：
  - 多个线程可以同时获取读锁（共享）
  - 写锁是互斥的（排他）
  - 写锁期间禁止获取读锁
  
- **支持 Redis 分布式部署**：可在多个节点间协调读写访问

- **高性能**：使用 Lua 保证原子性操作，避免分布式并发带来的 race condition





## **二、核心数据结构（在 Redis 中）**



Redisson 的锁本质是通过 Redis 的 Hash 和 String 实现的。每个 RReadWriteLock 对应 Redis 中的一个 key。



### **1. 锁主 key 格式（Redis Key）**

假设锁名为 myLock，那么主键是：

```
{myLock}:rwlock
```



### **2. 读锁结构（Hash 存储）**

```
HSET {myLock}:rwlock
   mode               read     # 当前是读锁模式
   {UUID1}:threadId1  3        # 表示 UUID1 的 threadId1 拿了3次读锁
   {UUID2}:threadId2  1        # 表示另一个线程也持有读锁
```



### **3. 写锁结构（Hash 存储）**

```
HSET {myLock}:rwlock
   mode               write    # 当前是写锁模式
   {UUID1}:threadId1  2        # 写锁可重入，UUID1 的线程持有2次写锁
```



### **4. TTL 管理（写锁或读锁均设置过期时间）**

为防止死锁，锁会设置 expireTime，Redisson 支持锁续约机制。





## **三、加锁解锁逻辑详解**



### **加读锁逻辑（Lua 脚本中）**



加读锁成功的条件：

1. 当前无锁（mode 不存在）→ 直接设置 mode 为 read，记录线程计数；
2. 当前已是 read 模式 → 同一个线程可重入，或其他线程共享；
3. 当前是 write 模式 → 只有同一个线程可重入获取读锁（写锁降级场景）；



#### **核心流程：**

```lua
if (not exists(key)) or (HGET key mode == 'read') then
    HINCRBY key uuid:threadId 1
    HSET key mode 'read'
    EXPIRE key ttl
    return OK
elseif (HGET key mode == 'write') then
    if (HGET key uuid:threadId exists) then
        # 写锁持有线程获取读锁（降级）
        HINCRBY key uuid:threadId 1
        return OK
    else
        return FAIL
end
```



### **加写锁逻辑**



加写锁成功的条件：

1. 当前无锁 → 设置 mode=write，记录计数；
2. 当前已是 write 模式 → 可重入；
3. 当前是 read 模式 → 只有当前线程唯一持有所有读锁才可以升级为写锁，否则失败；



#### **核心流程：**

```lua
if (not exists(key)) then
    HSET key mode 'write'
    HSET key uuid:threadId 1
    EXPIRE key ttl
    return OK
elseif (HGET key mode == 'write') then
    if (HGET key uuid:threadId exists) then
        HINCRBY key uuid:threadId 1
        return OK
    else
        return FAIL
elseif (HGET key mode == 'read') then
    # 判断是否只有当前线程在读
    if (仅有 HGET key uuid:threadId 存在，其它都为 0) then
        HSET key mode 'write'
        return OK
    else
        return FAIL
```



### **解锁逻辑（读/写通用）**

- 解锁时判断当前线程是否是持有者（必须是自己加的才能解）
- 递减线程的计数
- 如果计数为 0，删除该线程标识字段
- 如果无任何线程标识了 → 删除整个锁





## **四、可重入性支持**

每个线程加锁都会传入唯一标识：UUID + threadId，类似：

```
UUID:threadId
```

Redis 中记录了每个线程加锁的次数（计数器），实现了可重入逻辑。**Redisson 本身维护 UUID + threadId 的唯一性和传递。**





## **五、公平性和防饿死问题**

Redisson 原生实现中并**不保证严格公平性**，但它通过以下机制避免饿死：

### **写锁饿死？**

- Redisson 有**内部等待队列**（本地），在锁释放时优先唤醒写锁线程（有一定写优先机制）



### **写锁持有时读锁排队？**

- 写锁持有期间新读锁请求会被阻塞（因为 mode=write） → 正常行为，避免数据不一致。



### **持有读锁之后会连续释放读锁？**

- 是的，多个读线程会连续获得锁（共享模式）直到出现写锁等待。



## **六、Lua 脚本原子性 + Redisson 实现细节**

所有加锁、解锁操作都通过 Lua 脚本完成，确保分布式原子性。



### **Redisson 主要封装在类：**

- org.redisson.RedissonReadWriteLock
- 其底层封装为 RedissonLock 和 RedissonReadLock, RedissonWriteLock



### **执行逻辑（调用栈）**

- 调用 readLock.lock() 或 writeLock.lock() 时会触发 Lua 脚本
- 如果失败（冲突）会调用 await() 进入等待队列（阻塞或超时）



## **总结图示**

```
Redis Key: myLock:rwlock

读锁（共享）
    mode=read
    uuid:thread1 = 3
    uuid:thread2 = 1

写锁（独占）
    mode=write
    uuid:thread3 = 2

加锁策略：
    - 可重入（计数器）
    - Lua 原子性
    - 支持写锁降级为读锁
    - 有限避免写锁饿死（写优先）
```





## **适用场景**

- 多实例协同写/读操作（如配置中心、订单系统、排行榜）
- 需要强一致但性能要求高的分布式环境
- Redis 高可用集群部署（推荐使用 RedLock 补充容错）