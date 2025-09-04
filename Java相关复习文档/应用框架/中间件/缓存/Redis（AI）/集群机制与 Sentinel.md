# Sentinel

## **三大职责**

1. **监控（Monitoring）**

   - Sentinel 会定期向主节点和从节点发送 PING，通过心跳检测存活。
   - 也会定期执行 INFO replication 来确认主从关系、延迟等信息。

   

2. **通知（Notification）**

   - 当发现某个节点状态变化（比如主观下线），Sentinel 会通过 **发布订阅（pub/sub）** 机制把消息通知给其他 Sentinel 或客户端。

   

3. **自动故障转移（Automatic Failover）**

   - 当确认主节点下线后，Sentinel 会选举一个新的主节点（从原有从节点中选出），并通知其他从节点去复制它。
   - 同时也会让客户端更新主节点地址。

   



## **主观下线 / 客观下线**

1. **主观下线（SDOWN, Subjective Down）**

   - 单个 Sentinel 认为主节点不可达（心跳/PING 超时）。
   - 只是“个人观点”，不能直接触发故障转移。

   

2. **客观下线（ODOWN, Objective Down）**

   - 多个 Sentinel 都报告某主节点 SDOWN，当达成 **quorum（法定人数）** 时，认为主节点真的下线 → 进入 ODOWN。
   - quorum 配置示例：

```
sentinel monitor mymaster 127.0.0.1 6379 2
```



1. 

   - 意思是：至少有 2 个 Sentinel 同意，主节点才算 ODOWN。

   

2. **Leader 选举**

   - 进入 ODOWN 后，Sentinel 集群会进行 **Leader 选举**，由 Leader 负责发起故障转移。
   - 选举基于 Raft 类似思想：通过 **配置纪元（epoch）** 来防止冲突，每个纪元只有一个 Leader。

   

## **故障转移流程**

1. **新主节点选举**

   - Leader Sentinel 挑选一个从节点晋升为新主节点。
   - 选择依据：复制偏移量最新、延迟最小、优先级最高。

   

2. **复制树重建**

   - 其他从节点被通知去复制新的主节点。
   - 原来的主节点如果恢复上线，会被重新配置为从节点，去复制新的主节点。

   

3. **配置传播**

   - Sentinel 会把新的主节点地址通过 pub/sub 通知给客户端。
   - 客户端（支持 Sentinel 的驱动）会自动切换连接。

   

## **部署要点**

1. **多副本多哨兵**

   - 至少 **3 个 Sentinel**，才能保证选举有多数派，避免脑裂。
   - 主从节点至少 **1 主 2 从**，保证转移时有候选。

   

2. **网络分区与分脑（Split-Brain）防护**

   - 如果 Sentinel 分布不均衡，可能出现不同哨兵集群各自选主 → 脑裂。
   - quorum 要合理配置，确保没有多数派时不会进行 failover。
   - 配合 min-replicas-to-write 等参数，避免主节点在孤立情况下继续写入导致数据不一致。

   

3. **安全写保护**

   - min-replicas-to-write：要求主节点至少有 N 个健康副本，否则拒绝写入。
   - min-replicas-max-lag：定义“健康副本”的最大复制延迟。
   - 避免网络抖动/分区时，主节点单点写入导致数据丢失。

   

**总结**

- **三大职责**：监控、通知、故障转移。
- **SDOWN/ODOWN**：单个 Sentinel → 主观；多个 Sentinel 达成 quorum → 客观，才会触发选举。
- **故障转移流程**：选主 → 复制树重建 → 配置传播。
- **部署要点**：至少 3 个 Sentinel，多副本，防脑裂，配合安全写配置。

✅ **一句话总结**：

Redis Sentinel = “一套分布式哨兵机制”，通过 **心跳监控 + 多数派判定 + Leader 选举 + 自动转移** 来保证主从集群的高可用，同时依赖合理部署和 min-replicas 配置防止脑裂和数据丢失。



# 集群模式

下面把 **Redis Cluster（集群）**的核心机制系统讲清楚，并按你给的提纲逐条展开。读完你能搞明白：键怎么路由、迁移时客户端为什么收到 MOVED/ASK、怎么做再均衡、脑裂是怎么被避免的、多 key 有哪些限制、客户端需要实现哪些重定向与重试策略。



------





## **1) 槽位与路由**

### **1.1 16384 槽 + CRC16**

- Redis Cluster把 key 空间划成 **16384 个哈希槽**（0..16383）。
- 路由规则：slot = CRC16(key) % 16384。
- 每个槽被分配给某个**主节点**，节点对自己负责的槽处理所有命令。



### **1.2 MOVED 与 ASK**

- **MOVED**：**永久重定向**。说明该槽已经迁走，客户端应更新路由表：

```
MOVED <slot> <ip:port>
```

- 之后该槽的所有请求直接发往新节点。

- **ASK**：**临时重定向**。说明槽正在**迁移进行中**（源 MIGRATING → 目标 IMPORTING）。

  客户端收到 ASK <slot> <ip:port> 时，这一条命令要：

  1. 先向目标节点发送 ASKING

  2. 再把**原命令**发到目标节点

     路由表不更新（因为还没“永久”迁完）。

  



### **1.3 Hash Tags（同槽强制）**

- 形如 user:{123}:cart，CRC16 只对花括号里的 **123** 做哈希 → **同 tag 的键必定位于同一个槽**。
- 作用：让多 key 操作（如事务/Lua/MSET）满足“同槽”约束。



## **2) 再均衡 / 迁移**

### **2.1 槽迁移的基本流程**

- 管理工具（如 redis-cli --cluster reshard/rebalance）会在**源节点**把某些槽标记为 **MIGRATING**，在**目标节点**把这些槽标记为 **IMPORTING**。

- 迁移数据：对目标槽中的每个 key 执行内部的 MIGRATE（移动键值）。

- 期间客户端会遇到：

  - 源节点对该槽返回 **ASK**，引导你把本次命令临时发给目标；
  - 迁移完成后，源/目标执行 CLUSTER SETSLOT <slot> NODE <targetNodeId>，全网达成一致 → 客户端开始收到 **MOVED**（永久）。

  

### **2.2 节点握手与扩缩容**

- 新节点加入：在任意一个已知节点上执行 CLUSTER MEET <ip> <port> 完成**握手**，分配 nodeId。
- 分配槽：CLUSTER ADDSLOTS ... 或用 --cluster add-node --cluster reshard 自动化。
- 缩容/下线：将槽迁走，再 CLUSTER FORGET。





### **2.3 import/migrating 状态**

- CLUSTER NODES 能看到节点/槽状态：

  - **migrating**：该节点是**源**，槽正被迁出；
  - **importing**：该节点是**目标**，槽正被迁入。

  

- 这两种状态就是 ASK 产生的根源。



## **3) 故障检测与选举**

### **3.1 Gossip**

- 节点间通过**集群总线**（端口 = 服务端口+10000）互发 PING/PONG/MEET/FAIL 消息，交换视图（gossip）。
- 当多数**主节点**一致认为某主**不可达**，会把它标记为 **FAIL**。



### **3.2 主从切换 & 投票**

- 每个主节点可以挂多个从节点。主挂了，其中一个从发起 **故障转移（failover）**。

- 选举机制：

  - 候选从节点基于**复制偏移量**（越接近主、数据越新）有优势；
  - 需要获得**多数主节点**的授权（投票）；
  - 使用 **配置纪元（config epoch）** 抢占“任期”，防止并发转移冲突。

  

- 选出新主后，其他从节点切到新主，中止对旧主的复制。





### **3.3 集群可用性判定**

- 必须有**多数主节点**正常，且**所有槽都有覆盖**，集群才处于 OK。
- cluster-require-full-coverage yes|no：缺槽是否禁止对外服务（默认 **yes** 更安全）。
- 读从：客户端可 READONLY 走从节点读（牺牲一致性，获得扩展读能力）。



## **4) 多 key 命令限制**



### **4.1 同槽要求**

- MSET/DEL/RENAMENX、事务 MULTI/EXEC、Lua EVAL 的 **所有 key 必须落在同一槽**，否则报

```
CROSSSLOT Keys in request don't hash to the same slot
```

- 

- 解决：使用 **Hash Tags** 统一到一个槽，例如：

  - mset user:{42}:name ... user:{42}:age ...
  - 脚本/事务里所有 key 都写成相同的 {tag}。

  



### **4.2 EVAL 跨槽**

- Cluster 模式下，Lua 脚本**不能**在服务器端跨槽访问键。
- 方案：把操作分解到客户端按槽路由，或利用 Hash Tag 规整到同槽。



### **4.3 事务/脚本限制**

- 事务的**路由节点 = 第一个 key 的槽**；后续 key 必须同槽。
- WATCH/UNWATCH 同理，监视的 key 也需同槽。



## **5) 客户端实现（重点）**

### **5.1 路由表**

- 启动时：向任一节点发 CLUSTER SLOTS / CLUSTER NODES，构建**槽 → 节点**映射（0..16383）。
- 常驻缓存：内存里维护这张表，命令根据 key 算 slot，直发目标节点。



### **5.2 重定向处理**

- **MOVED**：更新路由表（把该槽指向新节点），并**重试**本次命令。

- **ASK**：对本次命令：

  1. 向目标节点先发 ASKING；
  2. 再发原命令；
  3. **不更新**路由表。

  

- **TRYAGAIN**：目标槽在迁移/繁忙，小退避后重试。

- **CLUSTERDOWN**：集群不可用（缺槽/无多数），客户端应快速失败或降级。



### **5.3 重试策略（伪代码）**

```c++
for attempt in 1..MAX:
  node = route(slot(key))
  resp = send(node, cmd)
  if resp is OK: return
  if resp is MOVED(s,newNode):
      updateRoute(s,newNode); continue
  if resp is ASK(s,newNode):
      send(newNode, ASKING); resp = send(newNode, cmd); return resp
  if resp is TRYAGAIN or BUSY:
      sleep(backoff(attempt)); continue
  if resp is CROSSSLOT:
      fail-fast (开发期就该避免)
fail with error
```

- **幂等性**：重试要优先用于**幂等**或**可去重**的写（例如使用业务幂等键/流水号），避免重复写入副作用。
- **Pipeline**：同一 pipeline 必须是**同槽**；不同槽分拆到不同连接。



## **6) 常用管理命令与场景**

- 槽计算：CLUSTER KEYSLOT mykey
- 拓扑：CLUSTER SLOTS / CLUSTER NODES
- 加节点：CLUSTER MEET ip port
- 分配槽：CLUSTER ADDSLOTS 0 1 ...
- 迁槽（工具）：redis-cli --cluster reshard <any-node>
- 再均衡：redis-cli --cluster rebalance <any-node>
- 下线节点：迁空槽 → CLUSTER FORGET <nodeId>
- 只读从：客户端 READONLY，或直连从节点（注意过期/一致性）



## **7) 小图速记（ASCII）**

```
Slots(0..16383)
 ┌───────────────┬───────────────┬───────────────┐
 │ 0..5460       │ 5461..10922   │ 10923..16383  │
 │  → Master A   │  → Master B   │  → Master C   │
 │    \_ Slave A'│    \_ Slave B'│    \_ Slave C'│
 └───────────────┴───────────────┴───────────────┘

迁移时：
 Source(MIGRATING s) ──ASK──► Target(IMPORTING s)
 完成后：SET SLOTS s -> Target  → 客户端收到 MOVED，更新路由
```



## **8) 实战建议（踩坑规避）**

- **统一 Hash Tag 规范**：把“同一业务实体”的所有 key 统一成 biz:{entityId}:field，提前避免 CROSSSLOT。
- **控制再均衡窗口**：迁槽在低峰做；客户端要实现 **ASK** 流程以避免“短暂不可用”。
- **读从风险**：READONLY 读的是**异步复制**数据，允许短暂过期/不一致。
- **故障演练**：定期模拟主挂、迁槽、扩缩容，确认客户端重定向/幂等/重试策略正确。
- **覆盖策略**：cluster-require-full-coverage yes（默认）能防“部分槽缺失仍对外写”的数据错乱。



# 主从模式

## **一、主从模式的作用**

- **高可用性**：当主节点故障时，可以把从节点提升为主节点（通常结合 Sentinel 来做自动故障转移）。
- **读写分离**：主节点负责写，从节点负责读，提升系统的读性能。
- **数据冗余**：主从复制能保证数据多副本存储，降低数据丢失风险。



## **二、主从复制原理**

### **1. 连接与握手**

- 从节点执行：

```
SLAVEOF <ip> <port>
# Redis 5.0+ 改名为
REPLICAOF <ip> <port>
```

- 
- 从节点会与主节点建立 TCP 连接，并发送 PSYNC 请求，表示自己要开始复制。



### **2. 同步过程**

Redis 的复制分为 **全量复制** 和 **部分复制** 两种情况：

**(1) 全量复制**

首次建立复制关系，或部分复制失败时会触发：

1. 从节点发送 PSYNC ? -1 → 请求全量同步。
2. 主节点执行 BGSAVE 生成 RDB 快照。
3. 把 RDB 文件发送给从节点。
4. 从节点清空自己的数据库，加载 RDB 数据。
5. 在 RDB 生成期间产生的新写命令，主节点会写入复制缓冲区，并在 RDB 传输完成后发送给从节点，保证数据不丢。



**(2) 部分复制**

- 主从之间维护一个复制偏移量 offset。

- 主节点有一个 **复制积压缓冲区（backlog）**（默认 1MB，环形）。

- 如果从节点短暂掉线后重连，能提供自己的 replid + offset，主节点会判断 backlog 里是否还有这部分数据：

  - 在 → 只发缺失部分 → 部分复制；
  - 不在 → 必须做全量复制。

  

## **三、异步复制与一致性**

- **Redis 主从复制默认是异步的**：主节点执行写命令后立即返回客户端，不等待从节点确认。

- **风险**：如果主节点宕机，最近写的部分数据可能尚未复制到从节点 → 数据丢失。

- **改进**：

  - 使用 WAIT numreplicas timeout 命令，可以让写操作等到指定数量的从节点确认才返回（半同步），牺牲性能换一致性。
  - 生产上常结合 **持久化（AOF/RDB）+ 从节点冗余 + Sentinel** 一起保障。

  

## **四、主从切换与 Sentinel**

- Redis 本身只提供复制功能，不提供自动故障转移。

- 需要 **Sentinel** 来监控主节点，一旦主节点失联：

  - 多个 Sentinel 达成一致 → 标记主节点为 ODOWN（客观下线）。
  - 选举一个 Sentinel 为 Leader，挑选一个从节点晋升为新主节点。
  - 其他从节点自动切换去复制新的主节点。

  

## **五、常见问题与配置**

### **1. 主从延迟**

- 主从复制是异步 + 单线程的，从节点可能延迟几 ms~几百 ms，写多读少的业务影响不大，但读一致性要求高的场景要谨慎。



### **2. 链式复制 vs 星型复制**

- Redis 6.0 之后支持 **从节点再挂从节点**（Replica of Replica）。
- 节点拓扑类似链式，减轻主节点的复制压力。



### **3. 配置项**

- repl-backlog-size：复制积压缓冲区大小，过小会导致频繁全量复制。
- min-replicas-to-write + min-replicas-max-lag：保护性写策略，避免主节点孤立时仍然写入导致数据不一致。



## **六、总结**

- **主从复制**是 Redis 保证高可用的基础。
- **全量复制**：RDB 快照 + 命令流追赶。
- **部分复制**：依靠 backlog + offset 实现，避免频繁全量。
- **默认异步**，一致性无法保证，但可结合 WAIT 提升保障。
- **自动故障转移**：需要 Sentinel 组件。
- **生产建议**：至少 1 主 2 从 + 多个 Sentinel，保证高可用与数据安全。



✅ 一句话总结：

Redis **主从模式 = 数据复制机制**，为高可用和读扩展服务，但本身只负责复制，不负责自动切换；要做到高可用，需要配合 Sentinel 或 Cluster。

