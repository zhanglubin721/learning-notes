## Spring事务

### Spring 的 @Transactional 注解，默认传播行为（Propagation.REQUIRED）

- **事务的启动时机**是：**在目标方法执行前（准确说是在进入方法体之前）就已经开启事务了**。
- Spring 是通过 **AOP（代理）机制**来管理事务的：当你调用一个被 @Transactional 注解的方法时，Spring 的事务拦截器会拦截这个调用，**在调用方法之前开启事务**，然后再调用方法，最后在方法执行完后提交或回滚事务。

### **示例流程（以默认传播行为为例）：**



```
@Transactional
public void doSomething() {
    // 第1步：这里执行前事务已开启
    // 第2步：执行数据库操作
    // 第3步：方法执行完毕，根据是否有异常提交或回滚事务
}
```



### **事务何时不会立即启动？**

以下情况可能不会立即开启事务：

1. **传播行为为 SUPPORTS、NOT_SUPPORTED、NEVER 等**，且当前线程中没有事务上下文时，不会开启新事务。
2. **方法内部自己调用自身类中另一个 @Transactional 方法**，这种情况可能会**绕过 Spring 的代理**，导致事务根本没被开启。



## 源码细节

### **1.入口类：TransactionInterceptor**

这是事务拦截的核心类，实现了 MethodInterceptor。

```java
@Override
@Nullable
public Object invoke(MethodInvocation invocation) throws Throwable {
    // 1. 提取事务属性（从 @Transactional 注解读取）
    Method method = invocation.getMethod();
    Class<?> targetClass = invocation.getThis().getClass();
    TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);

    // 2. 获取对应的事务管理器，如 DataSourceTransactionManager
    PlatformTransactionManager tm = determineTransactionManager(txAttr);

    // 3. 方法唯一标识（用于日志或监控）
    String joinpointIdentification = methodIdentification(method, targetClass);

    // 4. 开启事务并执行方法
    return createTransactionIfNecessary(tm, txAttr, joinpointIdentification, () -> {
        try {
            return invocation.proceed(); // 执行目标方法
        } catch (Throwable ex) {
            throw ex;
        }
    });
}
```

### **2.核心执行方法：createTransactionIfNecessary**

```java
protected Object createTransactionIfNecessary(
    PlatformTransactionManager tm, 
    TransactionAttribute txAttr,
    String joinpointIdentification,
    InvocationCallback invocation) throws Throwable {

    TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);

    try {
        // 执行目标方法
        Object retVal = invocation.proceedWithInvocation();
        commitTransactionAfterReturning(txInfo);
        return retVal;
    } catch (Throwable ex) {
        // 异常处理：回滚事务
        completeTransactionAfterThrowing(txInfo, ex);
        throw ex;
    } finally {
        // 清理事务上下文
        cleanupTransactionInfo(txInfo);
    }
}
```

### **3.真正调用事务管理器的方法：getTransaction()**

```java
protected TransactionInfo createTransactionIfNecessary(
    PlatformTransactionManager tm,
    TransactionAttribute txAttr,
    String joinpointIdentification) {

    if (txAttr != null) {
        // 关键调用：开启事务
        TransactionStatus status = tm.getTransaction(txAttr);
        // 封装为 TransactionInfo 返回
        return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
    } else {
        return null;
    }
}
```

### 4.事务结束：cleanupTransactionInfo()

```java
protected void cleanupTransactionInfo(@Nullable TransactionInfo txInfo) {
    if (txInfo != null) {
        // 恢复之前的事务信息（如果是嵌套事务，恢复上一个 TransactionInfo）
        txInfo.restoreThreadLocalStatus();
    }
}
```

```java
public void restoreThreadLocalStatus() {
    // 将当前 ThreadLocal 中的 TransactionInfo 恢复为旧的（previousTransactionInfo）
    TransactionAspectSupport.transactionInfoHolder.set(this.oldTransactionInfo);
}
```

**为什么需要这么做？**

Spring 使用 AOP 和线程本地变量（ThreadLocal）来维护事务上下文信息：

- 当你调用事务方法 A -> 方法 B（也是事务方法），Spring 会为 B 创建一个新的 TransactionInfo 并放入 ThreadLocal。
- 方法 B 执行完毕后，需要将 ThreadLocal 恢复为调用方法 A 时的事务信息（否则就乱套了）。
- 如果不清理，ThreadLocal 会持有错误或过期的事务状态，导致事务传播和资源管理异常。

**举例**

```java
@Transactional
public void methodA() {
    methodB(); // 也是 @Transactional
}

@Transactional
public void methodB() {
    // 做点事
}
```

Spring 会：

1. methodA：创建 txInfoA，绑定到 ThreadLocal。
2. methodB：再创建 txInfoB，覆盖 ThreadLocal。
3. methodB 执行完后：调用 cleanupTransactionInfo(txInfoB) 恢复为 txInfoA。
4. methodA 执行完后：再恢复为 null，清理线程状态。

| **动作**                                                   | **说明**                                                 |
| ---------------------------------------------------------- | -------------------------------------------------------- |
| TransactionAspectSupport.transactionInfoHolder.set(txInfo) | 方法调用前设置当前事务                                   |
| txInfo.restoreThreadLocalStatus()                          | 方法调用后恢复之前事务（或清除）                         |
| 作用                                                       | 避免事务信息在多层嵌套中混乱，确保线程事务上下文正确清理 |