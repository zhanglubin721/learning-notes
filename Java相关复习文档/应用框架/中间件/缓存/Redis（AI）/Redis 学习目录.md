# **1. 基础与总体架构**

- **定位与特性**

  - KV 存储、内存为主、单线程执行模型（命令原子）
  - CAP 取舍（单机、主从、Cluster 的不同侧重）
  - 延迟组成：网络、排队、执行、持久化、fork/COW

  

- **协议与客户端**

  - RESP2/RESP3 基本格式、流水线（pipeline）
  - 客户端连接参数：超时、保活、TCP-NODELAY、缓冲区

  

- **常用使用模式**

  - Cache-aside / Write-through / Write-behind
  - 分布式锁、ID 生成、限流、排行榜、消息队列（Streams）

  

# **2. 数据类型与时间复杂度（必背）**

- **String**

  - 命令族（GET/SET/INCR/GETSET/GETEX/SETNX/EX/PX/EXAT）
  - Bitmaps/Bitfield；计数、布尔位、签位/非签位
  - 内部编码：int / embstr / raw

  

- **List**

  - LPUSH/RPUSH/LPOP/RPOP/BLPOP/BRPOP/LRANGE/LREM
  - quicklist（节点 + listpack）的结构、索引复杂度

  

- **Hash**

  - HGET/HSET/HGETALL/HSCAN；小对象压缩阈值
  - 编码：listpack vs hashtable 切换条件

  

- **Set**

  - SADD/SREM/SISMEMBER/SMEMBERS/SINTER/SUNION
  - 编码：intset vs hashtable；去重/集合运算复杂度

  

- **Sorted Set (ZSet)**

  - ZADD/ZREM/ZRANGE/ZRANGEBYSCORE/ZMSCORE
  - 编码：listpack vs (跳表 + dict)；按分值/按字典序范围

  

- **Stream**

  - XADD/XRANGE/XREADGROUP/XACK/XCLAIM；Consumer Group/Pending
  - ID 语义、消息确认与重投递

  

- **HyperLogLog**

  - PFADD/PFCOUNT/PFMERGE；误差率、内存定常

  

- **GEO**

  - GEOADD/GEORADIUS/GEOSEARCH；底层 zset+geohash

  

- **Pub/Sub**

  - 发布/订阅模型、模式订阅；与 Streams 的差异与场景

  

# **3. 对象模型与内部编码（面试爱问“原理”）**

- **SDS（简单动态字符串）**

  - 结构字段、预分配、二进制安全

  

- **dict（哈希表）**

  - 渐进式 rehash、负载因子、哈希碰撞

  

- **listpack/quicklist**

  - ziplist 的替代；quicklist 节点压缩策略

  

- **skiplist（跳表）**

  - 层级、搜索/插入复杂度；zset 双结构设计（dict+skiplist）

  

- **intset**

  - 升级策略、内存布局

  

- **Rax（压缩前缀树）**

  - Streams 索引/ACL/模块内部使用

  

- **对象头/引用计数**

  - type/encoding/lru；共享对象（small ints）



# **4. 过期、内存与淘汰（maxmemory）**

- **过期字典**

  - 惰性删除 vs 定期删除；active-expire-cycle

  

- **内存统计与碎片**

  - jemalloc、fragmentation ratio、延迟来源

  

- **淘汰策略（政策）**

  - noeviction、volatile/allkeys + lru/lfu/random/ttl
  - LRU/LFU 采样近似、计数衰减；热点保护

  

- **大 Key / 热 Key**

  - 识别（MEMORY USAGE、HOT KEYS、monitor）与拆分策略

  

- **惰性释放**

  - UNLINK、lazyfree-lazy-eviction/lazy-expire/lazy-user-del



# **5. 持久化（RDB / AOF / 混合）**

- **RDB**

  - 触发（save/自动）、fork 子进程、COW 成本
  - 校验、压缩、加载行为；RDB 对延迟的影响

  

- **AOF**

  - appendfsync(always/everysec/no)；AOF 重写（bgrewriteaof）
  - rewrite buffer、AOF truncate/校验，恢复流程

  

- **混合/多段 AOF（7.x）**

  - BASE/INCR/HISTORY 文件与 manifest
  - RDB preamble、快速恢复权衡

  

- **持久化参数调优**

  - no-appendfsync-on-rewrite、自动重写阈值与大小

  

# **6. 复制（Replication）**

- **PSYNC/部分重同步**

  - replid/offset、复制积压缓冲区（backlog）

  

- **全量同步流程**

  - RDB 传输、命令流追赶；磁盘/无盘复制

  

- **延迟与一致性**

  - 异步复制、读写一致性风险；WAIT（半同步）

  

- **安全写**

  - min-replicas-to-write / min-replicas-max-lag

  

# **7. 高可用：Sentinel**

- **三大职责**

  - 监控（心跳/INFO）、通知、自动故障转移

  

- **主观/客观下线（SDOWN/ODOWN）**

  - quorum、选举 Leader、配置纪元（epoch）

  

- **故障转移流程**

  - 新主选举、复制树重建、配置传播

  

- **部署要**

  - 多副本多哨兵、网络分区与分脑防护

  

# **8. 集群（Cluster）**

- **槽位与路由**

  - 16384 槽、CRC16、MOVED/ASK、hash tags

  

- **再均衡/迁移**

  - 槽迁移、握手、import/migrating 状态

  

- **故障检测与选举**

  - Gossip、投票、epoch/term；主从切换

  

- **多 key 命令限制**

  - 同槽要求；EVAL 跨槽；事务/脚本限制

  

- **客户端实现**

  - 路由表更新、重定向处理、重试策略



# **9. 事务、脚本与函数**

- **事务（MULTI/EXEC/WATCH）**

  - 乐观锁、不可回滚、队列中的错误处理语义

  

- **Lua 脚本（EVAL/EVALSHA）**

  - 原子性、读写限制、脚本缓存、超时与 KILL

  

- **Redis Functions（7.x）**

  - FUNCTION/FCALL 库管理、与 EVAL 的差异

  

- **Pipeline vs 事务**

  - 吞吐与隔离的取舍，超时/失败处理

  

# **10. 网络与执行模型**

- **事件循环**

  - 单线程命令执行；I/O 多路复用（epoll/kqueue）

  

- **I/O 线程（6.0+）**

  - 读取/写回多线程、命令执行仍单线程

  

- **BIO 后台线程**

  - AOF/关闭文件/惰性释放等异步任务

  

- **慢查询与延迟监控**

  - slowlog、LATENCY DOCTOR、MONITOR

  

# **11. 性能优化清单**

- **命令级**

  - O(N) 命令慎用（KEYS、*ALL、HGETALL 大对象）
  - 批处理与 pipeline；SCAN 家族替代 KEYS

  

- **数据结构级**

  - 小对象压缩阈值（listpack）参数；zset/list/hash 选型
  - 大 key 拆分、分片；热点 key 限流/本地缓存

  

- **持久化与复制**

  - fsync 策略、AOF 重写窗口；复制 backlog 调参

  

- **系统级**

  - THP 关闭、overcommit、swap 禁用、透明大页
  - NUMA、句柄/内核参数、网卡队列/中断绑核

  

# **12. 安全与隔离**

- **ACL（6.0+）**

  - 用户/规则/类别、命令与 key 前缀限制

  

- **TLS（6.0+）**

  - 证书、双向认证、性能权衡

  

- **多租户隔离**

  - key 命名规范、前缀、防逃逸；Cluster 隔离

  

# **13. 观测与运维**

- **INFO 模块**

  - memory、persistence、replication、cluster、latency 指标

  

- **备份与恢复**

  - RDB 冷备、AOF 校验与截断；一致性快照策略

  

- **容量规划**

  - 数据量估算、对象开销、碎片率、淘汰压测

  

- **升级与兼容**

  - 主版本差异、数据迁移（RDB/AOF/导入导出）

  

# **14. 工具与命令生态**

- **redis-cli**

  - –pipe、–scan、–latency、–bigkeys

  

- **redis-benchmark**

  - 场景/并发/流水线参数；基准陷阱

  

- **检查/修复**

  - redis-check-rdb / redis-check-aof

  

- **迁移/复制工具**

  - MIGRATE、DUMP/RESTORE、异构迁移注意



# **15. 版本要点（面试常追问“你知道的新特性”）**

- **6.x**

  - ACL、TLS、I/O 线程；复制/持久化优化

  

- **7.x**

  - Multi-part AOF（manifest）、Redis Functions、listpack/quicklist v2 强化
  - Cluster/Streams 若干增强、命令族细化

  

- **类型演进**

  - ziplist → listpack 迁移；配置项名称随之变化

  

# **16. Java 侧工程实践（Jedis/Lettuce/Redisson/Spring）**

- **客户端选择**

  - Jedis（阻塞+连接池）、Lettuce（Netty 异步/Reactive）

  

- **连接管理**

  - 池大小、超时、背压、NIO 线程；序列化（String/JSON/Proto）

  

- **Redisson 分布式结构**

  - RLock/RReadWriteLock/Semaphore/RateLimiter；公平锁/续约/看门狗
  - Lua 脚本原子性；锁失效/漂移/时钟问题与防护（租约/栅栏）

  

- **Spring Data Redis**

  - 模板、Cache Abstraction、@Cacheable/@CacheEvict
  - 序列化策略（GenericJackson2JsonRedisSerializer 等）

  

- **典型缓存问题**

  - 穿透（Bloom）、击穿（互斥/逻辑过期）、雪崩（随机 TTL）
  - 双写一致性（先删后写/延迟双删/消息总线）、热点保护

  

- **实践细节**

  - Pipeline 回包顺序与错误处理、跨槽多 key 限制
  - 监控（pool 指标、命令耗时、慢日志上报）

  

# **17. 典型设计题库（会被追问实现细节）**

- **排行榜/Feed**

  - zset 设计、去重、分页、去热点

  

- **分布式锁**

  - SET NX PX；续约、崩溃安全、主从一致性风险、WAIT/仲裁/围栏

  

- **限流与漏斗**

  - 令牌桶（Lua）、滑动窗口、ZSET/Bitmap 方案

  

- **去重与计数**

  - HyperLogLog、Bitmap、布隆（模块/自建）

  

- **消息队列**

  - Streams vs List 阻塞队列；Exactly-once 的工程补偿

  

- **秒杀/库存**

  - 预扣/超卖防护、幂等、热点 key、异步落库

  

- **地理围栏/附近的人**

  - GEO + ZSET；距离计算与精度/性能权衡

  

# **18. 反模式与坑位**

- **全量扫描命令**

  - KEYS/*ALL 在大数据集；用 SCAN/SCAN* 替代

  

- **大 key / 大集合**

  - 单次 HGETALL/SMEMBERS/ZRANGE 巨量；拆分/分页

  

- **不可控脚本**

  - 长脚本阻塞事件循环；无超时保护；非幂等

  

- **持久化误配**

  - everysec 期望与磁盘能力不匹配；rewrite 抖动

  

- **复制/哨兵误配**

  - min-replicas 未设置；哨兵数量/拓扑不足；split-brain

  

- **Cluster 误用**

  - 跨槽多 key、事务/脚本不可跨槽；路由更新失败重试风暴

  

# **19. 面试自检清单（速查）**

- **复杂度口算题**：常见命令的 O(1)/O(logN)/O(N)
- **一次故障复盘**：延迟尖刺定位流程（slowlog/latency/AOF/复制）
- **容量计算**：对象个数×平均值 + 元数据开销 + 碎片比例
- **部署图**：主从+哨兵、Cluster、双机房（等待策略/降级）
- **调参模板**：maxmemory + 策略、appendfsync、周期任务、阈值