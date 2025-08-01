## 容器初始化与加载

```
【构造入口】
AnnotationConfigApplicationContext(AppConfig.class)
    ↓
【1】创建 DefaultListableBeanFactory
    - 容器的核心数据结构
    - 注册系统默认的 bean 解析器、类型转换器等工具

【2】注册配置类（register(AppConfig.class)）
    - 将 @Configuration 类作为 BeanDefinition 注册进 BeanFactory
    - 若有多个配置类，一并注册

【3】执行 refresh()，开始容器刷新流程：
      ↓
  【3.1】prepareRefresh()
      - 初始化容器状态
      - 设置启动时间、标志位、初始化资源占位符等

  【3.2】obtainFreshBeanFactory()
      - 获取 BeanFactory（前面已创建）

  【3.3】prepareBeanFactory(beanFactory)
      - 注册常用工具类（如 environment、systemProperties）
      - 添加 ClassLoader、ExpressionResolver、TypeConverter 等

  【3.4】postProcessBeanFactory()
      - 钩子方法（空实现，供子类扩展）

【4】执行 BeanFactoryPostProcessor（扩展点！）
      ↓
  【4.1】调用 ConfigurationClassPostProcessor
      - 解析 @ComponentScan、@Import、@Bean 等注解
      - 注册所有解析到的 BeanDefinition
      - 包括注解方式的 @Component、@Repository、@Service、@Controller 等

【5】注册 BeanPostProcessor（扩展点！）
      ↓
  - 注册所有实现 BeanPostProcessor 接口的 Bean
  - 比如：AutowiredAnnotationBeanPostProcessor、AopProxyCreator 等
  - AOP、事务、异步等核心功能都依赖于这里

【6】初始化所有非懒加载的单例 Bean（核心！）
      ↓
  - 调用 getBean → doCreateBean：
      - 实例化
      - 属性注入（依赖注入）
      - 初始化（init-method、@PostConstruct）
      - BeanPostProcessor 前后置处理
      - AOP 动态代理（在 postProcessAfterInitialization）

【7】容器刷新完成：
      ↓
  - 调用 finishRefresh()
      - 发布 ContextRefreshedEvent 事件
      - 初始化事件广播器和监听器（ApplicationListener）
      - 执行所有 SmartInitializingSingleton Bean 的 afterSingletonsInstantiated 方法

【容器已启动完成，可对外提供服务】
```

## Spring Bean生命周期

```
创建阶段：
    ↓
【1】实例化（构造器）
    ↓
【2】依赖注入（populate）
    ↓
【3】Aware 接口回调（*Aware）
    ↓
【4】BeanPostProcessor.postProcessBeforeInitialization()
    ↓
【5】初始化：
       @PostConstruct
       InitializingBean.afterPropertiesSet()
       自定义 init-method
    ↓
【6】BeanPostProcessor.postProcessAfterInitialization()
    ↓
Bean 可使用
    ↓
容器关闭：
    ↓
【7】销毁：
       @PreDestroy
       DisposableBean.destroy()
       自定义 destroy-method
```



## Spring Boot启用流程

```
SpringApplication.run(MyApp.class, args)
    ↓
【1】创建 SpringApplication 对象
    - 推断应用类型（Web/非Web）
    - 加载启动类资源（SpringFactoriesLoader）
    - 初始化 ApplicationContext 类型（如 AnnotationConfigServletWebServerApplicationContext）
    - 加载所有 ApplicationContextInitializer、ApplicationListener
    ↓
【2】执行 run() 方法（正式启动）
    ↓
【3】准备环境 prepareEnvironment()
    - 加载 application.properties、application.yml
    - 解析命令行参数、环境变量
    - 构建 ConfigurableEnvironment
    ↓
【4】创建并准备 ApplicationContext
    - 创建 IOC 容器（AnnotationConfigServletWebServerApplicationContext）
    - 回调所有 ApplicationContextInitializer
    ↓
【5】加载主配置类（@SpringBootApplication）
  	- 自动装配逻辑触发（详见后面）
    ↓
【6】刷新容器（context.refresh()）
    - 触发 Spring 容器完整生命周期流程（BeanFactoryPostProcessor、BeanPostProcessor、生单例 Bean 等）
    	↓
    【6.1】invokeBeanFactoryPostProcessors()
    	↓
    【6.2】registerBeanPostProcessors()
    	↓
    【6.3】finishBeanFactoryInitialization()
    	↓
    【6.4】onRefresh()                        <-- 关键！
           	↓
           【6.4.1】createWebServer()         <-- 获取 ServletWebServerFactory 并 new Tomcat()
                    	↓
                    【6.4.2】webServer.start() // 启动 Tomcat
    ↓
【7】完成启动后事件广播 listeners.running()
    - 回调 ApplicationRunner、CommandLineRunner
    - 广播 ApplicationReadyEvent，标志启动完成
```



### 自动装配逻辑

对应上面的【5】的内容

```
@SpringBootApplication
    ↓
= @Configuration
  + @ComponentScan
  + @EnableAutoConfiguration
    ↓
@EnableAutoConfiguration
    ↓
@Import(AutoConfigurationImportSelector.class)
    ↓
AutoConfigurationImportSelector#selectImports()
    ↓
【1】读取 META-INF/spring.factories 中的所有自动配置类
     - key: org.springframework.boot.autoconfigure.EnableAutoConfiguration
     - value: xxxAutoConfiguration 全限定类名列表（比如 WebMvcAutoConfiguration、RedisAutoConfiguration）
    ↓
【2】根据 @Conditional 系列注解做条件过滤：
     - @ConditionalOnClass（类是否存在）
     - @ConditionalOnMissingBean（是否已注册某类）
     - @ConditionalOnProperty（配置属性是否满足）
     - @ConditionalOnWebApplication（当前是否 Web 应用）
    ↓
【3】将满足条件的配置类作为 BeanDefinition 注册到 IOC 容器中
    - 注意：此时只是注册，还没创建实例
    - 注册后等 Spring 容器 refresh() 时统一创建和初始化这些 Bean
```



Tomcat启动流程

```
SpringApplication.run()
    ↓
【1】创建 SpringApplication & 推断为 Web 应用
    - 设置应用类型：WebApplicationType.SERVLET
    - 选择 ApplicationContext 类型：AnnotationConfigServletWebServerApplicationContext

【2】创建 IOC 容器：AnnotationConfigServletWebServerApplicationContext
    - 是一个 WebApplicationContext，具备 Web 环境能力

【3】刷新容器（refresh()） → createWebServer()
    - ServletWebServerApplicationContext 中重写了 onRefresh()
    - 在 onRefresh() 中调用 createWebServer()

【4】获取 ServletWebServerFactory Bean
    - 默认是 TomcatServletWebServerFactory（由自动装配提供）
    - 由 spring-boot-autoconfigure 中的 EmbeddedWebServerFactoryCustomizerAutoConfiguration 注册

【5】创建 Tomcat 实例（new Tomcat()）
    - 设置端口（默认为 8080）
    - 注册 Context（Servlet 容器级别的根目录）
    - 设置连接器、Host、Engine、BaseDir 等

【6】将 DispatcherServlet 添加为默认 Servlet
    - 加载 WebMvcAutoConfiguration 中的 DispatcherServletRegistrationBean
    - 注册路径为 "/" 的 DispatcherServlet，作为前端控制器

【7】调用 tomcat.start()
    - 启动 Tomcat 监听端口
    - 输出 `Tomcat started on port(s): 8080` 日志
```

