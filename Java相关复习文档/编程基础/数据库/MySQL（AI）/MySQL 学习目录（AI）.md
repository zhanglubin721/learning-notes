# **1. SQL 基础与语法**

- 查询基础

  - SELECT/FROM/WHERE、DISTINCT、ORDER BY/LIMIT、GROUP BY/HAVING
  - 聚合函数与常见陷阱（COUNT(*)/COUNT(col)/COUNT(1)）
  - 子查询（相关/不相关）、派生表、CTE（WITH）

  

- DML

  - INSERT（多值、INSERT…ON DUPLICATE KEY）、REPLACE
  - UPDATE（多表更新）、DELETE（多表删除）、TRUNCATE

  

- DDL

  - CREATE/ALTER/DROP（表、索引、视图、触发器、事件）
  - 约束（PRIMARY/UNIQUE/FOREIGN KEY/CHECK）、默认值、NOT NULL

  

- 进阶 SQL

  - 窗口函数（OVER/ORDER/PARTITION、排名/滑动窗口）
  - 公用表表达式递归查询
  - JSON（存储、函数、生成列索引）
  - 地理空间（GIS，SRID、R-Tree、函数）

  

# **2. 服务器架构与组件**

- 逻辑分层

  - 连接管理/线程模型、解析器、优化器、执行器、插件

  

- 协议与会话

  - 连接握手、会话/事务状态、Prepared Statements、临时表目录

  

- 插件机制

  - 认证插件、审计/日志插件、查询重写插件

  

# **3. 存储引擎概览**

- InnoDB（默认、事务/崩溃安全、行级锁）
- MyISAM（只读/归档场景、表锁、崩溃不安全）
- Memory（内存表、哈希索引、易丢失）
- NDB、Archive、Blackhole（了解即可）
- 引擎选择与混用注意事项



# **4. InnoDB 物理存储**

- 表空间

  - 系统表空间、独立表空间（file-per-table）、通用表空间
  - Undo 表空间、临时表空间

  

- 页与页结构

  - 16KB 页、页头 FIL/FSP/IBUF、Infimum/Supremum、目录槽

  

- 行格式

  - COMPACT/DYNAMIC、溢出页、变长列/行溢出

  

- B+Tree

  - 聚簇索引（主键即数据）、二级索引指向主键

  

- 关键结构

  - LSN、双写缓冲（doublewrite）、Change Buffer、AHI（自适应哈希）

  

# **5. 事务、隔离级别与 MVCC**

- 事务性质：ACID

- 隔离级别：READ UNCOMMITTED / READ COMMITTED / REPEATABLE READ / SERIALIZABLE

- MVCC

  - Read View、Undo Log、可见性规则
  - RC vs RR 的幻读表现与处理差异

  

- 自增与事务

  - innodb_autoinc_lock_mode、批量插入并发

  

# **6. 锁体系与并发控制**

- 锁类型

  - S/X、IS/IX、记录锁、间隙锁、Next-Key 锁、临键锁

  

- 加锁粒度与范围

  - 主键/唯一键等值、范围、覆盖扫描

  

- 死锁

  - 形成原因、等待图、检测/回滚策略、避免手段

  

- 元数据锁（MDL）

  - DDL/DML 互斥、长事务与 MDL 互卡

  

- 行锁 vs 表锁、意向锁语义



# **7. 日志与崩溃恢复**

- Redo Log（物理、WAL、checkpoint）
- Undo Log（逻辑回滚、MVCC 可见性）
- 二进制日志 Binlog（SBR/ROW/MIXED，GTID）
- Relay Log（复制中继）
- 错误/通用/慢日志
- 崩溃安全与恢复流程（LSN、checkpoint、双写）



# **8. 索引体系与数据访问**

- 索引类型

  - B+Tree、哈希（Memory）、全文索引、前缀索引、函数/生成列索引、不可见索引

  

- 设计原则

  - 选择性/基数、最左前缀、覆盖索引、宽表/长列影响
  - 聚簇主键选择（自增 vs UUID v4/v7）、写放大/页分裂

  

- 索引使用细节

  - ICP（Index Condition Pushdown）
  - MRR（Multi-Range Read）、BKA（Batched Key Access）
  - 索引合并（intersection/union）

  

- ORDER BY / GROUP BY 与索引

  - 索引顺序、等值+范围中断、前导列/方向一致性

  

# **9. 优化器与执行计划**

- 成本模型与统计信息

  - 基数估计、持久化统计、直方图、采样

  

- 执行计划解读

  - type（const/eq_ref/ref/range/index/ALL）、key/rows/filtered、Extra 字段
  - EXPLAIN FORMAT=JSON、EXPLAIN ANALYZE

  

- 优化器变换

  - 子查询半连接（IN/EXISTS 转换）、去相关、物化/合并
  - 派生表合并、谓词下推、常量传递、等价类

  

- Optimizer Hints

  - JOIN_ORDER/STRAIGHT_JOIN/USE|FORCE INDEX/NO_MERGE/SEMIJOIN/NO_ICP 等

  

- Optimizer Trace（决策过程审计）



# **10. 连接与复杂查询**

- JOIN 类型：INNER/LEFT/RIGHT/FULL（模拟）、CROSS

- JOIN 算法

  - 嵌套循环（NLJ）、Block Nested-Loop、BNL 与 BKA 对比
  - Join Buffer、驱动表选择、连接顺序

  

- 反/半连接技巧

  - NOT EXISTS vs NOT IN、LEFT JOIN IS NULL

  

- 去重/集合

  - UNION vs UNION ALL、去重代价

  

# **11. 排序/分组/临时表/filesort**

- filesort 算法（单路/双路）
- Using temporary 的触发与内存/磁盘临时表（InnoDB 临时表）
- Top-N（ORDER BY … LIMIT）优化（堆/索引扫描）
- GROUP BY 执行策略与分组临时结构



# **12. 统计信息与直方图**

- 统计信息计算、持久化、过期与 ANALYZE TABLE
- 直方图类型（等宽/等高）、适用场景与误判
- 列相关性与多列统计的局限



# **13. 数据类型/字符集/时区**

- 数值/小数精度、DECIMAL 存储代价
- 文本/二进制（TEXT/BLOB）与溢出页
- 日期时间（DATETIME vs TIMESTAMP、时区转换、夏令时）
- 字符集/排序规则
  - utf8mb3 vs utf8mb4、ci/cs/bin、Unicode 0900 collations
- 枚举/集合的利弊、生成列（虚拟/存储）



# **14. DDL、在线变更与锁**

- DDL 算法：COPY / INPLACE / INSTANT
- ALTER 变更可否在线、LOCK 级别控制
- 在线变更工具：gh-ost、pt-online-schema-change（切换、回滚）
- 元数据锁（MDL）与长事务/长查询的影响
- 自增列/主键变更的开销与风险



# **15. 内存与缓存**

- InnoDB Buffer Pool（大小、实例数、热页）
- 自适应哈希索引（AHI）
- Change Buffer（非唯一二级索引插入缓冲）
- 日志缓冲（innodb_log_buffer_size）
- 临时表空间/排序缓冲/Join Buffer/TMP 表大小
- 线程池/连接池（MySQL 企业/Percona/Proxy）



# **16. 复制与高可用**

- 复制模式

  - 异步、半同步（ACK 时点差异）、GTID 复制

  

- 复制事件与格式

  - ROW/STATEMENT/MIXED 的优缺点与重放确定性

  

- 延迟与一致性

  - 延迟监控、只读从库读一致性（READ ONLY/超读）

  

- MGR / InnoDB Cluster

  - 单主/多主、认证与冲突检测、仲裁

  

- 传统 HA

  - MHA、Keepalived+VIP、ProxySQL/MySQL Router、读写分离

  

# **17. 分区与分库分表**

- 分区表

  - RANGE/LIST/HASH/KEY、子分区、分区裁剪
  - 唯一键/外键限制（分区键要求）、维护（REORGANIZE/EXCHANGE）

  

- 分库分表

  - 水平拆分、分片键设计、跨分片 JOIN/聚合、全局二级索引
  - 分布式中间件（ShardingSphere、MyCat、ProxySQL 规则）

  

# **18. 备份、恢复与数据迁移**

- 逻辑备份

  - mysqldump/mysqlpump、mydumper/myloader、导入并发

  

- 物理备份

  - Percona XtraBackup、热备与一致性点

  

- 恢复

  - Point-in-Time Recovery（备份+binlog）、GTID 精准恢复

  

- 迁移/上线

  - 双写/对账、灰度切换、回滚预案

  

# **19. 安全、权限与审计**

- 认证与密码策略、TLS、caching_sha2_password
- 用户/角色/权限粒度（库/表/列/过程/代理）
- 最小权限与密钥轮换
- 审计与合规（审计插件/日志）



# **20. 参数与性能调优**

- 核心参数

  - innodb_buffer_pool_size/instances、innodb_log_file_size、innodb_flush_log_at_trx_commit
  - sync_binlog、binlog_cache_size、tmp_table_size/max_heap_table_size
  - innodb_flush_neighbors、io_capacity/io_capacity_max
  - innodb_autoinc_lock_mode、transaction_isolation、sql_mode

  

- 磁盘/IO/NUMA/文件系统选择

- 连接/线程/并发度（thread_cache、线程池）

- 热点写/热点行/热点索引（自增 PK、二级唯一、页分裂）



# **21. 监控与可观测性**

- 慢日志（阈值、采样、未用索引、管理语句记录）
- performance_schema / sys schema（等待/锁/内存/语句摘要）
- information_schema（表/索引/统计/元信息）
- SHOW ENGINE INNODB STATUS / 死锁信息
- Optimizer Trace、EXPLAIN ANALYZE 基线



# **22. 常见性能模式与反模式**

- 分页优化（基于游标/覆盖索引、延迟关联）
- 反连接（NOT EXISTS）与反模式（NOT IN NULL 陷阱）
- 批量写入/批处理事务（组提交、fsync 频率）
- COUNT 优化、去重优化、Top-N、滚动聚合
- 范围查询与复合索引中断、排序+LIMIT 的最佳实践
- N+1 查询、函数对索引的破坏、隐式类型转换
- UUID/雪花 ID 对聚簇的影响、主键选择
- 大事务/长事务危害（undo 膨胀、阻塞 purge）



# **23. 版本差异与演进（5.6/5.7/8.0）**

- 8.0 的字典/元数据事务化、移除查询缓存
- 8.0 新特性：窗口函数、CTE、角色、资源组、直方图、隐式主键、不可见索引、JSON 强化、DDL INSTANT
- 5.7：虚拟列/生成列、InnoDB 改进、GTID 稳定
- 兼容性与升级路径、降级风险点



# **24. 表结构与范式/建模**

- 三范式/反范式、宽表与窄表、索引覆盖与列裁剪
- 约束设计（外键取舍、应用层约束 vs 数据库约束）
- 时间维（时区、软删除、审计列）、多租户建模
- 大字段与冷热分离、归档表/历史表



# **25. 运维与故障处理**

- 常见告警与排障流程（连接暴涨、锁等待、IO 饱和、CPU 飙高）
- 死锁排查（等待图、重试/拆批/索引改造）
- 复制断链/延迟处理、GTID 修复
- 磁盘满/表空间满、碎片整理、OPTIMIZE TABLE 的影响



# **26. 云与生态工具**

- 云 RDS 特性（只读实例、备份策略、参数模板、只读一致性）
- 代理与网关（ProxySQL/MySQL Router）、读写分离与路由策略
- 变更平台（审核、回滚、工单）、数据校验
- 数据同步/变更捕获（Debezium、Canal、DataX）