# 字段解读

- id：子查询/派生表的执行顺序标号，越大越晚执行。
- select_type：SIMPLE/PRIMARY/SUBQUERY/DERIVED/UNION 等。
- table：当前访问的表/别名；<derivedN> 表示派生表。
- type（连接类型，越靠前越好）：
  - system > const > eq_ref > ref > range > index > ALL
  - 一看到 ALL≈全表扫描，要警惕。
- possible_keys：可能可用的索引。
- key：实际用到的索引。
- key_len：使用索引的字节长度，可看“最左前缀”利用到了几列。
- ref：和索引匹配的比较对象（常见是 const 或某列）。
- rows：**估计**要读的行数（不是返回行数）。
- filtered：预计过滤比例（%）。
- Extra：常见值
  - Using index（覆盖索引）
  - Using where（还有行要用 WHERE 过滤）
  - Using filesort（需要外部排序→可能很慢）
  - Using temporary（需要临时表→可能很慢）
  - Using index condition（索引条件下推，5.6+）



# 什么是 Using filesort（为什么慢、怎么避免）



## **概念**

Using filesort 出现在 EXPLAIN ... Extra 里，表示**排序不能直接按索引顺序完成**，MySQL 需要把待排序的行放入排序缓冲区（内存 sort buffer），必要时 spill 到磁盘临时文件，做额外排序。这并不一定真的“落盘到文件”，但意味着**非索引排序**的额外 CPU/内存/IO 开销。

MySQL 的 filesort 会根据行宽/配置（如 max_length_for_sort_data）选择：

- **两趟算法**：先只排序**排序键+行指针**，再按排序指针回表取整行（随机 IO 多）；
- **一趟算法**：直接把整行拷入排序缓冲区排序（内存占用大）。



## **触发 filesort 的典型场景**

- ORDER BY 的列**没有**匹配到**同一个**合适的**联合索引顺序**；
- ORDER BY 列上做了**函数/表达式**（破坏索引序）；
- **混向排序**（5.7 难以完全利用 B-Tree；8.0 可用降序索引大幅改善）；
- DISTINCT + ORDER BY 组合、UNION 去重后再排序等。



## **避免 filesort 的套路（示例）**



### A) WHERE 等值 + ORDER BY 同向匹配联合索引

```
-- 差：
SELECT id, amount, created_at
FROM orders
WHERE user_id = 100
ORDER BY created_at DESC
LIMIT 20;  -- Extra: Using filesort

-- 优：
CREATE INDEX idx_user_created ON orders(user_id, created_at); -- 5.7
-- 8.0 如需混向可：
-- CREATE INDEX idx_user_created ON orders(user_id, created_at DESC);

-- 查询同上，EXPLAIN 期望：不再出现 Using filesort
```



### **B) 多列排序保持顺序一致 & 列顺序匹配**

```
-- 目标：先 status 再 created_at 升序
CREATE INDEX idx ON orders(user_id, status, created_at);

SELECT ...
FROM orders
WHERE user_id = 100
ORDER BY status ASC, created_at ASC
LIMIT 50;  -- 能用 idx，避免 filesort
```



### **C) 混向排序（5.7 vs 8.0）**

```
-- 目标：ORDER BY created_at DESC, id ASC
-- 5.7：难完全用索引 → 常 filesort
-- 8.0：建降序/升序混合索引
CREATE INDEX idx ON orders(user_id, created_at DESC, id ASC);
```



### **D) 避免函数破坏索引序**

```
-- 差：
SELECT * FROM orders
WHERE user_id = 100
ORDER BY DATE(created_at) DESC;  -- 函数导致 filesort

-- 优：
SELECT * FROM orders
WHERE user_id = 100
ORDER BY created_at DESC;        -- 或生成列 + 索引
```



### **E) 深分页减少排序输入（延迟关联）**

```
-- 先只按索引扫描主键，再回表
SELECT o.*
FROM (
  SELECT id
  FROM orders 
  WHERE user_id = 100
  ORDER BY created_at DESC, id DESC
  LIMIT 100000, 20
) t
JOIN orders o ON o.id = t.id;
```

> 虽不一定完全消除 filesort，但大幅**减小排序输入规模**。





# 什么是 Using temporary（什么时候会有、怎么避免）



## **概念**

Using temporary 表示 MySQL 为了完成**分组/去重/部分复杂查询**，需要创建**内部临时表**（内存临时表或磁盘临时表）。常见触发：

- GROUP BY / DISTINCT；
- 聚合后再排序（GROUP BY ... ORDER BY，且 ORDER BY 不等同于 GROUP BY 顺序）；
- 在分组列上使用**表达式/函数**；
- 复杂 UNION、派生表、子查询等。



临时表引擎取决于版本与配置（internal_tmp_mem_storage_engine、tmp_table_size/max_heap_table_size）。溢出会落盘（变慢）。



## **避免 Using temporary 的套路（示例）**



### A) 让 GROUP BY 直接走索引顺序



```
-- 差：无索引或乱序 → Using temporary
SELECT user_id, COUNT(*)
FROM orders
GROUP BY user_id;

-- 优：
CREATE INDEX idx_user ON orders(user_id);

-- 期望 Extra：Using index for group-by（没有 Using temporary）
```



### B) GROUP BY 顺序 = ORDER BY 顺序

```
-- 差：
SELECT user_id, COUNT(*)
FROM orders
GROUP BY user_id
ORDER BY COUNT(*) DESC;  -- 聚合后再按聚合结果排序，常需要临时表

-- 优（可接受时）：
-- 取消额外排序或换成可利用索引的排序列；
-- 或者把这类重度报表下放到 OLAP/物化视图预计算。
```



### **C) 避免在分组列上做函数**

```
-- 差：DATE() 破坏索引 → Using temporary
SELECT DATE(created_at) d, COUNT(*)
FROM orders
GROUP BY DATE(created_at);

-- 优：生成列 + 索引，或用范围：
ALTER TABLE orders 
  ADD COLUMN created_date DATE 
  GENERATED ALWAYS AS (DATE(created_at)) STORED,
  ADD INDEX idx_created_date(created_date);

SELECT created_date, COUNT(*)
FROM orders
GROUP BY created_date;
```



### **D) 两列去重/分组匹配联合索引**

```
SELECT user_id, status
FROM orders
GROUP BY user_id, status;
-- 索引 (user_id, status) → 常见 “Using index for group-by”
```



# **小结与检查清单**

- **Seek 分页不丢/不重**：

  排序必须**稳定**：ORDER BY major DESC, tie DESC；

  续页条件用**元组比较**：(major, tie) < (:last_major, :last_tie)；

  建索引：(filter_eq..., major, tie)；MySQL 8.0 可用**降序索引**匹配混向。

- **消 filesort**：

  WHERE 等值列放前，ORDER BY列按顺序放后；

  排序方向一致（5.7），8.0 可用降序索引；

  避免函数/表达式破坏索引序；深分页用**延迟关联**或**seek**。

- **消 temporary**：

  让 GROUP BY 列按**联合索引顺序**读取；

  让 GROUP BY 与 ORDER BY 尽量“同列同序”；

  避免对分组列做函数；必要时报表**预聚合/物化视图**。

- **诊断**：

  EXPLAIN 看 type、rows、Extra；

  MySQL 8.0 用 EXPLAIN ANALYZE 看**实际耗时/行数**；

  调整索引与 SQL 改写，直到 Extra 里**不再**看到 Using filesort / Using temporary（或能接受其成本）。