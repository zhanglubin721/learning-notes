## ** Spring Bean 生命周期流程（精细版）**

> 以下是 Spring Bean 创建过程的详细生命周期，左边是生命周期阶段，右边是处理器扩展点（你可以自定义实现它们）：

### **一、容器启动阶段（还没开始实例化 Bean）**

| **阶段**                  | **说明**                                               | **相关扩展接口**                                         |
| ------------------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| 1️⃣ 读取配置类 / XML        | 解析配置、扫描类、加载 BeanDefinition（Bean 的元信息） | 无                                                       |
| 2️⃣ BeanDefinition 注册完成 | Bean 还没实例化，只是定义被注册到容器                  | BeanDefinitionRegistryPostProcessor（可以动态注册 Bean） |
| 3️⃣ 容器刷新前              | 可以修改 BeanDefinition 内容（如 scope、lazy-init）    | BeanFactoryPostProcessor                                 |

### **二、单个 Bean 创建流程（以某个具体 Bean 为例）**

> 以下是容器开始正式创建某个 Bean 时的详细流程（重点）：

| **步骤**                                                     | **描述**                                           | **执行的处理器**                         |
| ------------------------------------------------------------ | -------------------------------------------------- | ---------------------------------------- |
| 1️⃣ 判断是否是单例，是则从缓存获取                             | -                                                  | -                                        |
| 2️⃣ 执行 InstantiationAwareBeanPostProcessor 的 **postProcessBeforeInstantiation** | 允许你自己创建一个 Bean 实例，常用于代理（如 AOP） | InstantiationAwareBeanPostProcessor      |
| 3️⃣ 如果上一步返回非 null，则跳过后续实例化流程（直接返回）    | -                                                  | -                                        |
| 4️⃣ 实例化 Bean（通过构造方法）                                | 相当于 new Xxx()                                   | -                                        |
| 5️⃣ 执行 InstantiationAwareBeanPostProcessor 的 **postProcessAfterInstantiation** | 是否继续进行依赖注入（返回 false 则跳过注入）      | InstantiationAwareBeanPostProcessor      |
| 6️⃣ 属性注入（setXxx）                                         | @Autowired, @Value, 构造注入等                     | AutowiredAnnotationBeanPostProcessor 等  |
| 7️⃣ 执行 InstantiationAwareBeanPostProcessor 的 **postProcessProperties**（Spring 5 新方法） | 可以修改属性值或自定义依赖注入                     | SmartInstantiationAwareBeanPostProcessor |
| 8️⃣ 执行 Aware 接口（如 BeanNameAware, BeanFactoryAware）      | 让 Bean 获取容器信息                               | -                                        |
| 9️⃣ 执行 BeanPostProcessor 的 **postProcessBeforeInitialization** | 初始化之前逻辑增强                                 | BeanPostProcessor                        |
| 🔟 调用初始化方法（顺序如下）                                 | -                                                  | -                                        |
| - @PostConstruct                                             | -                                                  | -                                        |
| - InitializingBean.afterPropertiesSet()                      | -                                                  | -                                        |
| - XML 或 Java Config 中配置的 init-method                    | -                                                  | -                                        |
| 1️⃣1️⃣ 执行 BeanPostProcessor 的 **postProcessAfterInitialization** | 初始化之后逻辑增强（AOP 代理多在此时）             | BeanPostProcessor                        |
| ✅ 最终返回 Bean，注册进一级缓存                              | 单例 Bean 进入容器                                 | -                                        |

## **图示：Spring Bean 生命周期及处理器执行位置**

```
容器启动阶段：
 ┌──────────────────────────────────────────────┐
 │ BeanDefinitionRegistryPostProcessor          │  --> 注册新 BeanDefinition
 │ BeanFactoryPostProcessor                     │  --> 修改 BeanDefinition 属性
 └──────────────────────────────────────────────┘

单个 Bean 创建流程：
 ┌─────────────┐
 │ 构造前（代理）│ --> InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation
 └──────▲──────┘
        │
        ▼
 ┌─────────────┐
 │ 构造 Bean    │ --> 通过构造方法 new 出 Bean 实例
 └──────▲──────┘
        │
        ▼
 ┌──────────────┐
 │ 属性注入     │ --> AutowiredAnnotationBeanPostProcessor 等
 └──────▲───────┘
        │
        ▼
 ┌─────────────────────────────┐
 │ postProcessBeforeInit       │ --> BeanPostProcessor
 └────────────▲────────────────┘
              │
              ▼
 ┌─────────────────────────────┐
 │ 初始化：@PostConstruct 等     │
 └────────────▲────────────────┘
              │
              ▼
 ┌─────────────────────────────┐
 │ postProcessAfterInit        │ --> BeanPostProcessor（常用于 AOP）
 └────────────▲────────────────┘
              │
              ▼
       ✅ Bean 注册完成，加入容器
```

## **小结：各类处理器对比**

| **接口名**                               | **执行阶段**         | **作用**                     |
| ---------------------------------------- | -------------------- | ---------------------------- |
| BeanDefinitionRegistryPostProcessor      | 容器刷新前           | 注册或修改 BeanDefinition    |
| BeanFactoryPostProcessor                 | 容器刷新前           | 修改已有 BeanDefinition 内容 |
| InstantiationAwareBeanPostProcessor      | 构造前/构造后/注入时 | 控制实例化/注入过程          |
| BeanPostProcessor                        | 初始化前后           | 增强 Bean，如代理            |
| SmartInstantiationAwareBeanPostProcessor | 属性注入时           | 精准处理构造器选择、依赖注入 |
| DestructionAwareBeanPostProcessor        | Bean 销毁前          | 清理资源                     |