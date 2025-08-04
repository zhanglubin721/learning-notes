## AT模式总体流程流程

```java
@GlobalTransactional
public void purchase(String userId, String commodityCode, int orderCount) {
    storageService.deduct(commodityCode, orderCount);
    orderService.create(userId, commodityCode, orderCount);
}
```

这个 purchase 是一个 **Seata** 的全局事务发起者



### **执行流程详解**

我们按照时间顺序来，还原真实执行逻辑，重点解释你问的这几个点：



#### **1. 事务发起方开始事务**

- @GlobalTransactional 被解析（是一个 Spring AOP 切面实现）。
- 发起调用前，会：
  - 向 **Transaction Coordinator（TC）** 注册并开启一个全局事务（生成 XID）。
  - 将 XID 放入当前线程的上下文中（RootContext.bind(xid)）。
  - 后续的 RPC 请求都会自动在 header 里传这个 XID。



#### 2. 执行 storageService.deduct

- storageService 是一个 **Seata 的参与者（分支事务发起者）**。
- 执行 RPC 请求时，header 里携带了 XID。
- 被调用的存储服务里，Seata 的 RM（ResourceManager） 会检测到这个 XID，参与到这个全局事务中。



**❗重点：**

- storageService.deduct 的方法体会完整执行，执行过程中会：
  - 注册一个分支事务到 TC（记录 SQL undo 日志、加行锁）。
  - 执行 SQL，比如库存扣减 update storage set count = count - ? where commodity_code = ?。
  - 这个 update 语句会通过 Seata 的数据源代理做以下操作：
    - 写 undo_log
    - 获取全局锁（加锁的是这条 SQL 影响到的行）
    - 执行SQL并且提交本地事务（⚠️）
- 方法体返回后，表示 deduct 执行完毕，并不是立即返回。



#### 3. 执行 orderService.create

- 同理，通过 XID 传递给 orderService。
- orderService 也向 TC 注册为一个分支事务，执行业务逻辑和 SQL，记录 undo 日志、加行锁。



#### 4. purchase 方法结束，触发事务提交

- @GlobalTransactional 的代理逻辑在方法返回之后会触发一次：

  - 向 TC 汇报：我要提交这个全局事务。
  - TC 会并发通知所有分支事务（storage、order）进行 **一阶段提交**（实际上是“提交本地事务”）。

  

#### **若出现异常：**

- 会向 TC 报告失败，TC 会通知所有分支事务进行 **回滚**。





### **常见疑问总结如下：**

| **你的说法**                                | **正确性** | **说明**                                                     |
| ------------------------------------------- | ---------- | ------------------------------------------------------------ |
| purchase 方法开始执行就会向 TC 报告开始事务 | 正确       | @GlobalTransactional 会发起全局事务，并绑定 XID              |
| deduct 方法没有注解也能加入全局事务         | 正确       | 只要传递了 XID，参与方就会自动加入                           |
| deduct 方法会锁资源后立刻返回               | ❌ 错误     | 会执行完整方法逻辑（包括 SQL 执行、锁、undo_log），不是“锁完就返回” |
| purchase 最后执行完毕后才提交事务           | 正确       | 全局提交发生在 purchase 方法体完全成功返回之后               |
| TC 通知分支提交                             | 正确       | TC 控制所有分支是否提交或回滚                                |



**关键机制补充**

- Seata 的分支事务不是靠注解来识别，而是靠数据源代理 + XID 传播 + 启动事务的拦截器。
- 即便你不加任何注解，只要用的是 Seata 代理的数据源 + 有 XID 传过来，服务就能正确参与分支事务。



### **常规流程：**

```
Client调用 purchase()
└──> GlobalTransactional Interceptor 开启全局事务 → TC 注册 → 生成 XID
    ├──> storageService.deduct() [传XID]
    │     └──> 加入分支 → undo_log → 行锁 → 本地事务提交
    ├──> orderService.create() [传XID]
    │     └──> 加入分支 → undo_log → 行锁 → 本地事务提交
    └──> purchase() 返回，触发 TC → 发起全局提交
          ├──> 通知 storage 分支提交
          └──> 通知 order 分支提交
```



### 报错流程

```
Client 调用 purchase()
└──> GlobalTransactionalInterceptor 开启全局事务 → TC 注册 → 生成 XID
    ├──> storageService.deduct() [传 XID]
    │     └──> 加入分支 → undo_log 记录 → 行锁 → 本地事务提交
    ├──> orderService.create() [传 XID]
    │     └──> 加入分支 → undo_log 记录 → 行锁 → ❌ 抛出异常 → 本地事务回滚
    └──> 捕获异常 → 全局事务触发回滚 (TC 发起)
          ├──> 通知 storage 分支回滚
          │     └──> 查询 undo_log → 执行反向 SQL → 释放行锁 → 本地回滚成功
          └──> order 分支因本地已回滚，无需重复操作（状态标记回滚）
```

