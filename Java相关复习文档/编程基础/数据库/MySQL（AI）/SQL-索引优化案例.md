## **一、初级优化（结构改写 + 减少重复扫描）**



### **1️⃣ 相关子查询改 JOIN**



**低效写法（相关子查询，每行扫描 orders 一次）**

```
SELECT u.name,
       (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;
```



**优化写法（先汇总，再 JOIN）**

```
SELECT u.name, COALESCE(o.cnt, 0) AS order_count
FROM users u
LEFT JOIN (
    SELECT user_id, COUNT(*) AS cnt
    FROM orders
    GROUP BY user_id
) o ON u.id = o.user_id;
```



**优化原理**：

- 原版是 N 次扫描（N=users 行数），优化后只扫描 orders 一次。
- 如果 orders 有索引 (user_id)，速度会更快。



### **2️⃣ 多次聚合 → 一次聚合**

**低效写法（多次扫描）**

```
SELECT COUNT(*) FROM orders WHERE status = 'paid';
SELECT COUNT(*) FROM orders WHERE status = 'pending';
SELECT COUNT(*) FROM orders WHERE status = 'cancelled';
```



**优化写法（一次扫描，分组统计）**

```
SELECT status, COUNT(*) AS cnt
FROM orders
GROUP BY status;
```



**优化原理**：一次 GROUP BY 取代多次全表扫描。



### **3️⃣ SELECT \* 改成只取需要的列**

**低效写法**

```
SELECT * FROM orders WHERE user_id = 123;
```



**优化写法**

```
SELECT order_id, amount, status FROM orders WHERE user_id = 123;
```



**优化原理**：

- SELECT * 会取所有列，可能触发大量 I/O 和网络传输。
- 只取用到的列，可以减少磁盘读取和网络流量。



## **二、中级优化（索引 + 条件优化）**



### **4️⃣ 利用索引覆盖查询（Covering Index）**



**低效写法（回表）**

```
SELECT order_id, amount
FROM orders
WHERE user_id = 123;
```

如果 orders 表有索引 (user_id)，但是没有包含 amount，就需要先找出匹配行，再回表读取 amount。



**优化写法（覆盖索引）**

```
-- 创建复合索引
CREATE INDEX idx_userid_orderid_amount ON orders(user_id, order_id, amount);

-- 查询
SELECT order_id, amount
FROM orders
WHERE user_id = 123;
```



**优化原理**：

- 覆盖索引让 MySQL 直接从索引里获取数据，不必回表，提高速度。



### **5️⃣ 范围查询优化**



**低效写法（索引失效）**

```
SELECT * FROM orders
WHERE YEAR(order_date) = 2025;
```

这样会让 MySQL 对每行调用函数，索引 (order_date) 失效。



**优化写法（改成范围条件）**

```
SELECT * FROM orders
WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01';
```



**优化原理**：范围条件可以直接用 B-Tree 索引做范围扫描。



### **6️⃣ 复杂 OR 改 UNION**



**低效写法**

```
SELECT * FROM orders
WHERE user_id = 1 OR status = 'paid';
```

如果 user_id 和 status 分别有索引，OR 会导致 MySQL 放弃索引（某些版本会优化，但仍可能慢）。



**优化写法**

```
SELECT * FROM orders WHERE user_id = 1
UNION
SELECT * FROM orders WHERE status = 'paid';
```



**优化原理**：

- 两次查询分别走索引，然后 UNION 合并结果。



## **三、高级优化（大数据量 & 报表优化）**





### **7️⃣ 用窗口函数替代子查询**



**低效写法**

```
SELECT o1.*
FROM orders o1
WHERE o1.amount = (
    SELECT MAX(o2.amount) FROM orders o2 WHERE o2.user_id = o1.user_id
);
```

每行都查一次最大值。



**优化写法**

```sql
SELECT *
FROM (
    SELECT o.*, 
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rnk
    FROM orders o
) t
WHERE rnk = 1;
```



**优化原理**：

- 窗口函数一次扫描完成所有分组排名，避免 N 次子查询。

```sql
SELECT o.*
FROM orders o
JOIN (
  SELECT user_id, MAX(amount) AS max_amt
  FROM orders
  GROUP BY user_id
) t ON o.user_id = t.user_id AND o.amount = t.max_amt;
```



### **8️⃣ 分区表（Partitioning）**

**场景**：大表数据（几亿行）+ 查询集中在最近数据。



**优化做法**

```
ALTER TABLE orders
PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026)
);
```

**优化原理**：

- 查询只扫描对应分区，而不是全表。



### **9️⃣ 物化视图 / 预计算表**



**低效写法（每次实时聚合）**

```
SELECT user_id, SUM(amount)
FROM orders
WHERE order_date >= CURDATE() - INTERVAL 7 DAY
GROUP BY user_id;
```



**优化写法**

- 定时任务（比如每天更新一次汇总表）：

```
CREATE TABLE user_sales_summary (
    user_id BIGINT,
    total_amount DECIMAL(10,2),
    last_update DATE
);
```



- 查询直接从预计算表拿：

```
SELECT * FROM user_sales_summary WHERE last_update = CURDATE();
```

**优化原理**：用空间换时间，适合报表类查询。





```sql
SELECT u.name,
       (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;
```



```sql
SELECT u.name, COALESCE(o.cnt, 0) AS order_count
FROM users u
LEFT JOIN (
    SELECT user_id, COUNT(*) AS cnt
    FROM orders
    GROUP BY user_id
) o ON o.user_id = u.id;
```





```sql
-- 有下单
SELECT u.*
FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.user_id = u.id
);

-- 从未下单
SELECT u.*
FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM orders o
  WHERE o.user_id = u.id
);
```





## **谓词下推（Predicate Pushdown）**

- **含义**：把 WHERE 条件（谓词）尽可能下推到**更靠近数据源**的阶段执行。
- **目的**：减少需要往上层传递的数据量，降低扫描成本。



### **举例**

```
SELECT * 
FROM orders o 
JOIN customers c ON o.cust_id = c.id
WHERE c.country = 'CN';
```

- 没有谓词下推：先把所有 customer 读出来 join，再过滤掉非中国的。
- 有谓词下推：优化器把 c.country='CN' 下推到访问 customers 表时就加上 → 只读出中国客户行。

**场景**：索引条件下推 (Index Condition Pushdown, ICP) 也是谓词下推的一种特例，把条件下推到索引遍历阶段，减少回表。





## **常量传递（Constant Propagation）**

- **含义**：优化器在解析 SQL 时，会把某些**条件中的常量信息传播**到其他等价条件，进行简化。





### **举例**

```
SELECT * FROM t
WHERE a = b AND b = 10;
```

- 原始条件：a=b AND b=10

- 常量传递优化后：推导出 a=10 AND b=10。

- 好处：

  - 可以直接利用 (a) 或 (b) 上的索引；
  - 减少运行时比较开销。

  







## **等价类（Equivalence Class）**

- **含义**：当查询里存在**相等连接/条件**时，优化器会把这些列和常量归到一个“等价类”中，互相传递约束。







### **举例**

```
SELECT * FROM t1, t2, t3
WHERE t1.a = t2.b AND t2.b = t3.c AND t3.c = 5;
```

- 等价类：{t1.a, t2.b, t3.c, 5}
- 优化器知道：这四个对象是等价的。
- 于是可以在任意一个表的访问路径中使用 =5 这个过滤条件，提升索引利用率。
