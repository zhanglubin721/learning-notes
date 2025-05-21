# Spring Boot做了什么

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

# 自定义 Spring Boot Starter Demo

太好了，我们就一步一步来，从零构建一个 **自定义 Spring Boot Starter**，体验 spring.factories 的自动配置机制。你将会亲手实现一个能被 Spring Boot 自动识别的组件。

## **项目结构预览**

我们会创建两个模块：

```
my-spring-boot-starter-demo/
├── my-spring-boot-starter-api        # 使用者项目（模拟业务系统）
└── my-spring-boot-starter-core       # starter 模块（封装配置逻辑）
```

## **第一步：创建 starter 模块**

**my-spring-boot-starter-core**

### **1.1 添加依赖（只依赖 Spring Boot）**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```

### **1.2 编写你要提供的服务类**

```java
package com.example.starter;

public class HelloService {
    public String sayHello(String name) {
        return "Hello, " + name + "!";
    }
}
```

### **1.3 编写自动配置类**

```java
package com.example.starter;

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HelloServiceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean  // 如果容器里没有这个 Bean，才注入
    public HelloService helloService() {
        return new HelloService();
    }
}
```

### **1.4 添加spring.factories**

创建文件：

```
src/main/resources/META-INF/spring.factories
```

内容：

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.starter.HelloServiceAutoConfiguration
```

## **第二步：创建使用者模块**

**my-spring-boot-starter-api**

这是模拟实际使用你 starter 的业务项目。

### **2.1 添加依赖（引用你刚才写的 starter 模块）**

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter-core</artifactId>
    <version>1.0.0</version>
</dependency>
```

### **2.2 正常写 Spring Boot 启动类**

```java
@SpringBootApplication
public class StarterDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(StarterDemoApplication.class, args);
    }
}
```

### **2.3 使用 HelloService**

```java
@RestController
public class TestController {

    @Autowired
    private HelloService helloService;

    @GetMapping("/hello")
    public String hello(@RequestParam String name) {
        return helloService.sayHello(name);
    }
}
```

### **最终效果**

你启动 my-spring-boot-starter-api 项目后，访问：

```
GET http://localhost:8080/hello?name=Tom
```

输出：

```
Hello, Tom!
```

说明你的 HelloService 是 **完全由 spring.factories 驱动自动配置注入的**。

**补充：你也可以加 @ConfigurationProperties**

你可以让 starter 支持配置项，例如：

```java
@ConfigurationProperties(prefix = "hello")
public class HelloProperties {
    private String prefix = "Hello";
    // ...getter/setter
}
```

结合 @EnableConfigurationProperties(HelloProperties.class)，即可自动加载配置。

是的，你说得非常准确！Spring Boot 的自动装配机制**核心是根据 spring.factories 文件中声明的类来加载自动配置类，而不是依赖某个注解去扫描的**。

### **核心机制总结**

```java
@EnableAutoConfiguration
```

这是 Spring Boot 自动配置的入口注解，它底层会触发：

```
AutoConfigurationImportSelector
```

这个类的作用就是：



> 读取所有依赖 jar 包中 META-INF/spring.factories 文件里 EnableAutoConfiguration 对应的配置类，然后将这些类通过 @Import 的方式注入到 Spring 容器中。

# 以SpringBoot自动连接数据库为例

## **结论导读：**

Spring Boot 能自动连接 MySQL，靠的就是：

### **自动配置类：**

```java
@AutoConfiguration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    // 内部会引入 DataSourceConfiguration、DataSourceInitializer 等类
}
```

### **自动读取的配置（application.yml / properties）：**

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

## **自动装配背后的流程详解：**

### **①** @EnableAutoConfiguration启动自动配置（通过 spring.factories）

spring-boot-autoconfigure 模块中有：

```properties
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

这条配置表示：**Spring Boot 会加载 DataSourceAutoConfiguration 自动配置类。**

### **②** DataSourceAutoConfiguration是个启动类，内部会引入更多配置

它实际导入了：

```java
@Import({
    DataSourceConfiguration.class,           // 核心：根据条件装配不同类型的 DataSource
    DataSourceInitializerInvoker.class       // 初始化数据库（schema.sql / data.sql）
})
```

### ③ DataSourceProperties是用来读取配置的类

```java
@ConfigurationProperties("spring.datasource")
public class DataSourceProperties {
    private String url;
    private String username;
    private String password;
    private Class<? extends DataSource> type;
    ...
}
```

@EnableConfigurationProperties(DataSourceProperties.class) 使其自动绑定你写在 application.yml 里的配置。

### ④ DataSourceConfiguration负责真正创建 DataSource Bean

它有多个静态内部类，根据你的依赖条件加载不同的数据源（HikariDataSource、Druid、Tomcat 等）：

```java
@Configuration
@ConditionalOnClass(HikariDataSource.class)
@ConditionalOnMissingBean(DataSource.class)
static class Hikari {
    @Bean
    public HikariDataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
    }
}
```

如果你依赖了 Hikari，那么就会走这一分支。

### **⑤ 最终效果**

经过上述步骤，Spring Boot 自动：

- 加载配置（spring.datasource.*）
- 自动装配对应的 DataSource 实现（比如 Hikari）
- 注册进 Spring 容器，供 JPA / MyBatis / JDBC template 使用
- 自动执行 schema.sql / data.sql 初始化（如存在）

### **总结一句话：**

> @Configuration（如 DataSourceAutoConfiguration）+ @Bean 是 Spring Boot 自动连接数据库的核心，它通过读取 application.yml 中的 spring.datasource.* 配置项，自动选择并注入 DataSource，并注册为 Spring 管理的 Bean，实现了“开箱即用”的体验。