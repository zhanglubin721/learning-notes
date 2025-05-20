## **一、Spring Boot 是什么？（宏观视角）**

Spring Boot 是基于 Spring 的**快速开发框架**，本质上是对 Spring 的封装和扩展，其目标是：

- **零配置启动**
- **快速开发 Web 项目**
- **内嵌容器（Tomcat）**
- **自动装配（@EnableAutoConfiguration）**
- **统一的配置中心（application.yml/properties）**

## **二、Spring Boot 核心启动流程**

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

### **核心入口：**

### **SpringApplication.run()**

```java
public static ConfigurableApplicationContext run(...) {
    SpringApplication app = new SpringApplication(...);
    return app.run(args);
}
```

### **【流程总览】Spring Boot 启动关键步骤：**

1. **构造 SpringApplication 对象**
2. **准备运行环境（Environment）**
3. **准备 ApplicationContext 容器**
4. **加载 BeanDefinition（含自动配置类）**
5. **调用所有初始化器（ApplicationContextInitializer）**
6. **调用所有监听器（ApplicationListener）**
7. **完成 Spring 容器刷新 & 启动应用**

## **三、Spring Boot 核心机制详解**

### **① @SpringBootApplication 注解解构**

```
@SpringBootApplication = @Configuration
                       + @ComponentScan
                       + @EnableAutoConfiguration
```

#### **-** 

#### **@EnableAutoConfiguration**

####  **是 Spring Boot 的灵魂**

背后是 AutoConfigurationImportSelector：

```
@Import(AutoConfigurationImportSelector.class)
```

它会读取 META-INF/spring.factories 中的自动配置类：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.xxx.XxxAutoConfiguration,\
com.yyy.YyyAutoConfiguration
```

### **② 自动装配（AutoConfiguration）原理**

通过 spring.factories 加载所有自动配置类（如 Web、DataSource、Redis、Kafka 等）：

- 每个 xxxAutoConfiguration 都是一个 @Configuration 类
- 内部用 @ConditionalOn... 系列注解做条件判断加载：

```java
@ConditionalOnClass(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.url")
@Bean
public DataSource dataSource() {
    return ...
}
```

### **③ ApplicationContextInitializer、ApplicationListener 的支持**

你可以在 resources/META-INF/spring.factories 中注册自定义启动扩展逻辑：

```
org.springframework.context.ApplicationContextInitializer=\
com.example.MyInitializer

org.springframework.context.ApplicationListener=\
com.example.MyListener
```

### **④ 内嵌容器（Tomcat/Jetty/Undertow）**

Spring Boot 自动配置类：

```
EmbeddedWebServerFactoryAutoConfiguration
```

默认配置了 TomcatServletWebServerFactory，会创建 Tomcat 实例并绑定到端口。

### **⑤ 全局配置中心** 

### **application.yml**

加载逻辑来自：

```
ConfigFileApplicationListener → EnvironmentPostProcessor → PropertySourcesLoader
```

Spring Boot 会优先加载以下配置顺序：

```
application.properties/yml → profile-specific 文件 → 系统环境变量
```

并将其加载到 Environment 对象中供所有 Bean 使用。

## **四、Spring Boot Starter 原理**

比如 spring-boot-starter-web：

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

本质上只是一个**依赖聚合模块**，引入了：

- Spring MVC
- Jackson
- 内嵌 Tomcat
- 日志系统（logback）

**Starter 并不包含代码，只是组合一组常用依赖。**

## **五、总结：Spring Boot 的核心价值**

| **能力**         | **实现原理**                        |
| ---------------- | ----------------------------------- |
| 自动配置         | @EnableAutoConfiguration + 条件注解 |
| 内嵌 Web 容器    | EmbeddedServletContainerFactory     |
| Starter 依赖管理 | 依赖聚合 + 自动配置类               |
| 配置统一         | application.yml + Environment       |
| 启动扩展         | Initializer / Listener / Runner     |
| 减少 XML 配置    | 基于注解的 Java 配置                |