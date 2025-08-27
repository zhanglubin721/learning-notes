## **子查询（Subquery）**

子查询就是嵌套在其他 SQL 语句里的 SELECT 语句。根据是否依赖外层查询，可分为两类：



### **不相关子查询（Uncorrelated Subquery）**

- 定义：子查询的执行不依赖外层查询的列，先执行子查询，再把结果用于外层查询。
- 例子：

```
SELECT name 
FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

- 内层 (SELECT AVG(salary) ...) 与外层无关，执行一次即可。

- 特点：

  - 执行代价较低（只执行一次），可被优化器缓存结果。
  - 常出现在 WHERE、HAVING、FROM 子句中。

  

### **相关子查询（Correlated Subquery）**

- 定义：子查询依赖外层查询的列，必须对外层的每一行重新计算。
- 例子：

```
SELECT e1.name
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

- 内层子查询依赖 e1.department_id，每一行都会触发执行。

- 特点：

  - 执行代价高，常被优化器重写为 JOIN 或 半连接。
  - MySQL 优化器有时能做“子查询去相关化（de-correlation）”，把它转为等价的 JOIN。

  

## **派生表（Derived Table）**

- 定义：放在 FROM 子句中的子查询，相当于“临时表”。
- 例子：

```
SELECT t.department_id, AVG(t.salary) AS avg_salary
FROM (
    SELECT department_id, salary
    FROM employees
    WHERE salary > 5000
) t
GROUP BY t.department_id;
```

- 执行逻辑：

  - MySQL 会先执行内层查询，把结果集存放在临时表（派生表）中；
  - 外层查询再基于这个派生表执行。

  

- 特点：

  - MySQL 5.6 之前，派生表默认会“物化”成临时表。
  - MySQL 5.7+ 引入了 **派生表合并（derived merge）优化**，有些派生表不再落地临时表，而是直接嵌入外层查询，提高性能。
  - 常用于简化 SQL 逻辑，或作为优化器强制的中间步骤。

  

## **CTE（Common Table Expression, WITH）**

- 定义：通过 WITH 子句定义的临时结果集，可以在后续查询中多次引用。
- 例子：

```
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT e.name, e.salary
FROM employees e
JOIN dept_avg d ON e.department_id = d.department_id
WHERE e.salary > d.avg_salary;
```

- 特点：

  

  - 语义上类似派生表，但可被复用多次，语义更清晰。
  - 支持 **递归 CTE**（常用于树形结构查询）：

  



```
WITH RECURSIVE subordinates AS (
    SELECT id, manager_id, name
    FROM employees
    WHERE id = 1
    UNION ALL
    SELECT e.id, e.manager_id, e.name
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

- 在 MySQL 8.0 才引入，5.x 没有。



- 执行逻辑：

  - **非递归 CTE**：通常与派生表类似，也可能被优化器内联（merge into parent query）。
  - **递归 CTE**：逐步展开，直到满足终止条件；MySQL 内部会构建临时表存储中间结果。

  

你看到的 **“半连接”“优化器内联”** 等关键词，确实都是 MySQL 优化器在做的一类“子查询/派生表重写”的技术手段。很多文章容易泛泛而谈，下面我帮你系统梳理一下这些概念，按 **背景 → 含义 → 内部做法 → 举例** 来说明：



## **半连接（Semi-Join）**

**背景**

我们写 SQL 时经常有这样的语句：

```
SELECT * 
FROM employees e
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.emp_id = e.id
);
```

语义是“只要外表 e 的某一行存在至少一条匹配的 o 行，就输出 e 这一行”。

这里不关心 orders 中有多少条，只要存在就行。

**半连接的含义**

- **半连接 (Semi-Join)**：一种连接语义，只返回“是否匹配”的结果，而不是展开所有匹配。
- 与之相对：普通 INNER JOIN 会生成所有匹配行，可能导致笛卡尔放大。

**MySQL 内部做法**

MySQL 优化器会把 EXISTS、IN (SELECT …) 等子查询改写为 **半连接执行**，避免重复扫描和大结果集。

常见执行策略有：

- **Table Pullout**：把子查询拉出来直接 JOIN。
- **Duplicate Weed-out**：先把子查询结果去重，再 JOIN。
- **Materialization**：物化子查询结果成临时表，加索引去查。
- **Loose Index Scan**：对子查询直接利用索引快速判断存在性。



## **优化器内联（Optimizer Inline / Merge）**

**背景**

很多人写 SQL 时喜欢用派生表：

```
SELECT dept_id, AVG(salary)
FROM (SELECT * FROM employees WHERE salary > 5000) t
GROUP BY dept_id;
```

直觉是先执行内层，再外层。

**优化器内联的含义**

- **内联（merge/inline）**：优化器发现派生表不需要真正物化，就把它的查询语句直接合并到外层查询中。
- 这样减少了临时表开销，提高性能。

**内部逻辑**

- MySQL 5.6 之前：派生表一定会物化成临时表 → 慢。
- MySQL 5.7+ 引入 **derived_merge** 优化：能内联就尽量内联。
- 限制：如果派生表包含 LIMIT、GROUP BY、DISTINCT 等语义，可能必须物化。



## **去相关化（De-correlation）**

- 把“相关子查询”改写为普通 JOIN。
- 例如：

```
SELECT * FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e.dept_id
);
```



- 优化器可能改写成：

```
SELECT e.* 
FROM employees e
JOIN (
    SELECT dept_id, AVG(salary) AS avg_sal 
    FROM employees GROUP BY dept_id
) x ON e.dept_id = x.dept_id
WHERE e.salary > x.avg_sal;
```



## **物化（Materialization）**

- 优化器决定“把子查询/CTE 结果集落到临时表”，后续直接查临时表。
- 常用于 IN (SELECT ...)、EXISTS、CTE 等场景。