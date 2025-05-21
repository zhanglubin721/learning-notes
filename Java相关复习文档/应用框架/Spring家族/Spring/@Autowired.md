## 工作原理

### **Step 1：Spring 扫描并注册 Bean**

当你在某个类上加了如 @Component、@Service 或通过 XML 注册，它就会被 Spring IoC 容器识别为一个待管理的 Bean，生成一个 BeanDefinition。

### **Step 2：Spring 注册** 

### **AutowiredAnnotationBeanPostProcessor**

在启动阶段，Spring 自动注册一系列的 BeanPostProcessor，其中就包括：

```
AutowiredAnnotationBeanPostProcessor
```

它是专门处理 @Autowired 和 @Value 注解的。

### **Step 3：通过反射找出哪些字段/方法上加了 @Autowired**

当 Spring 实例化你的 Bean 后，会执行 postProcessMergedBeanDefinition() 和 postProcessProperties() 方法。

来看一下它的内部处理逻辑（简化）：

```java
public class AutowiredAnnotationBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    // 处理合并后的 BeanDefinition，提取注入元数据
    public void postProcessMergedBeanDefinition(...) {
        InjectionMetadata metadata = findAutowiringMetadata(beanClass);
        // 缓存起来，避免重复解析
    }

    // 真正执行注入
    public PropertyValues postProcessProperties(...) {
        InjectionMetadata metadata = findAutowiringMetadata(beanClass);
        metadata.inject(bean, beanName, pvs);
    }

    private InjectionMetadata findAutowiringMetadata(Class<?> clazz) {
        // 这里使用反射扫描字段/方法上是否加了 @Autowired/@Value
        for (Field field : clazz.getDeclaredFields()) {
            if (field.isAnnotationPresent(Autowired.class)) {
                // 创建 AutowiredFieldElement，记录它
            }
        }
        ...
    }
}
```

### **所以关键点是：**

- Spring 利用 **反射机制** (Class.getDeclaredFields() 和 Field.isAnnotationPresent(Autowired.class))
- 将所有加了 @Autowired 的字段/方法封装为 InjectionMetadata 对象，记录注入点
- 注入前使用 InjectionMetadata.inject() 来执行实际的依赖注入

## **举个简单例子：**

```java
@Component
public class OrderService {

    @Autowired
    private UserService userService;
}
```

在 Spring 实例化 OrderService 后：

- 它通过反射发现 userService 字段加了 @Autowired
- 将该字段包装为一个 AutowiredFieldElement
- 调用 field.setAccessible(true) + field.set(orderService, resolvedBeanInstance)，将容器中 UserService 的实例注入进去

## **总结一句话：**

> Spring 是通过在 AutowiredAnnotationBeanPostProcessor 中使用 **反射** 机制，**扫描字段/方法上的注解元信息**，并通过构建 InjectionMetadata 来记录注入点，随后再执行自动注入的。

## 与@Resource的区别

@Autowired 和 @Resource 都用于依赖注入（Dependency Injection），但它们在**注入方式、默认行为和底层机制**上存在一些关键区别：

### **1. 注解来源不同**

| **注解**   | **所属规范**         |
| ---------- | -------------------- |
| @Autowired | Spring 框架          |
| @Resource  | JSR-250（Java 标准） |

### **2. 注入方式不同**

**@Autowired（Spring 提供）**

- **按类型** 注入（byType）
- 如果找到多个匹配类型的 Bean，会尝试根据名字匹配（byName），或用 @Qualifier 指定。
- 默认 **必须注入**，否则会抛出异常；可以使用 @Autowired(required = false) 改为可选注入。

```
@Autowired
private UserService userService;
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

**@Resource（Java 标准）**

- 默认 **按名称** 注入（byName）
- 如果找不到匹配名称的 Bean，再尝试按类型注入（byType）
- 没有 required=false 的选项，注入失败会抛出异常
- 可以指定 name 或 type 明确注入目标

```
@Resource
private UserService userService;
@Resource(name = "userServiceImpl")
private UserService userService;
```

### **3. 是否支持泛型注入**

- @Autowired 支持泛型（例如 List<UserService>）
- @Resource 不支持泛型集合自动注入（如 List<UserService>）

### **4. 使用场景建议**

| **场景**               | **推荐使用** |
| ---------------------- | ------------ |
| Spring 项目            | @Autowired   |
| 希望控制名称注入       | @Resource    |
| 希望跨容器使用标准注解 | @Resource    |

### **总结对比表：**

| **特性**             | @Autowired       | @Resource            |
| -------------------- | ---------------- | -------------------- |
| 来源                 | Spring           | JSR-250（Java 标准） |
| 注入方式默认         | 按类型（byType） | 按名称（byName）     |
| 支持 @Qualifier      | 是               | 否                   |
| 可选注入支持         | required = false | 不支持               |
| 是否支持泛型集合注入 | 支持             | 不支持               |