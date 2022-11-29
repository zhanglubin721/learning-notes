# SpringBoot自动配置

**说起springboot，大家第一个想到的肯定就是它“豪横”的功能——自动配置，那么springboot的自动配置原理是什么呢？今天我们就来探究一下。**

首先，我们建立一个springboot项目，打开这个项目的主配置类

 ![在这里插入图片描述](<image/363f639aa0294385b6bd3adac6868f36.png>)

## 一 @SpringBootApplication

我们可以看到@SpringBootApplication注解，这个注解标注在哪个类上，就代表哪个类是这个项目的主配置类，SpringBoot 就应该运行这个类的main方法来启动 SpringBoot ；它的本质是一个组合注解，我们点进去查看该类的元信息主要包含3个注解：<br>

![在这里插入图片描述](<image/f6c2a390077c41eb89caee1a4edc12fe.png>)<br>

 我用红色框起来的@SpringBootConfiguration（里面就是@[Configuration](<https://so.csdn.net/so/search?q=Configuration&spm=1001.2101.3001.7020>)，标注当前类为配置类，其实只是做了一层封装改了个名字而已）

 蓝色框起来的@EnableAutoConfiguration（开启自动配置）

 绿色框起来的@ComponentScan（包扫描）

### @SpringBootConfiguration

我们按住ctrl进入这个注解去查看：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```


@Configuration标注在某个类上，表示这是一个 springboot的配置类。可以向容器中注入组件。

### @EnableAutoConfiguration

@EnableAutoConfiguration顾名思义就是：开启自动导入配置<br>

 这个注解是SpringBoot的重点，下面会详细讲解

### @ComponentScan

@ComponentScan：

 配置用于 Configuration 类的组件扫描指令。

 提供与 Spring XML 的 < context:component-scan> 元素并行的支持。

 可以 basePackageClasses 或basePackages 来定义要扫描的特定包。 如果没有定义特定的包，就从声明该注解的类的包开始向下(子包)扫描。

## 二 @EnableAutoConfiguration

springboot自动配置主要都是这个注解来定义的，我们先点进来看看这个注解都包含了什么

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

点击@AutoConfigurationPackage注解

### 2\.1 @AutoConfigurationPackage

自动导入配置包

 点进去查看代码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

@Import 为spring的注解，导入一个配置文件，在springboot中为给容器导入一个组件，而导入的组件由 AutoConfigurationPackages.class的内部类Registrar.class 执行逻辑来决定是如何导入的。

#### 2\.1.1 @Import({Registrar.class})

点Registrar.class看一下源码

![在这里插入图片描述](<image/554f4a3dd6444d0485f83440b3ea842c.png>)

 Registrar实现了ImportBeanDefinitionRegistrar类，就可以被注解@Import导入到spring容器里。

![在这里插入图片描述](<image/ad3ee46c392e403aa04b94d176f16763.png>)<br>

![在这里插入图片描述](<image/5b7e3605850849648cb3195da1d46205.png>)在这个地方打一断点，运行可以看到

 (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0])的值为com.apesource.graduation\_project(当前启动类所在的包名)

**这样我们就得到一个结论：@AutoConfigurationPackage 就是将主配置类（@SpringBootApplication 标注的类）所在的包下面所有的组件都扫注册到 spring 容器中。**

### 2\.2 @Import({AutoConfigurationImportSelector.class})

作用：AutoConfigurationImportSelector开启自动配置类的导包的选择器，即是带入哪些类，有选择性的导入

点AutoConfigurationImportSelector.class进入查看源码，这个类中有两个方法一目了然：

![在这里插入图片描述](<image/acd92a5183824c4eb0ff468e38107645.png>)

 1、selectImports：选择需要导入的组件

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

2、getAutoConfigurationEntry：根据导入的@Configuration类的AnnotationMetadata返回AutoConfigurationImportSelector.AutoConfigurationEntry

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
    } else {
      AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
      List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
      configurations = this.removeDuplicates(configurations);
      Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
      this.checkExcludedClasses(configurations, exclusions);
      configurations.removeAll(exclusions);
      configurations = this.getConfigurationClassFilter().filter(configurations);
      this.fireAutoConfigurationImportEvents(configurations, exclusions);
      return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```


this.getCandidateConfigurations(annotationMetadata, attributes)这里断点查看

![在这里插入图片描述](<image/78508c6721bd44f1b348a7f405917ddc.png>)<br>

 configurations数组长度为133，并且文件后缀名都为 AutoConfiguration

**看到这里我们又的到一个结论： 这些都是候选的配置类，经过去重，去除需要的排除的依赖，最终的组件才是这个环境需要的所有组件。有了自动配置，就不需要我们自己手写配置的值了，配置类有默认值的。**

 我们继续往下看看是如何返回需要配置的组件的

#### 2\.2.1 getCandidateConfigurations(annotationMetadata, attributes)

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```


这里有一句断言：“No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.”

 意思是：“在 META-INF/spring.factories 中没有找到自动配置类。如果您使用自定义包装，请确保该文件是正确的。”

**结论： 即是要loadFactoryNames()方法要找到自动的配置类返回才不会报错**

#### 2\.2.2 getSpringFactoriesLoaderFactoryClass()

我们点进去发现：this.getSpringFactoriesLoaderFactoryClass()返回的是EnableAutoConfiguration.class这个注解。这个注解和@SpringBootApplication下标识注解是同一个注解。

```java
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
  	return EnableAutoConfiguration.class;
}
```


**结论：获取一个能加载自动配置类的类，即SpringBoot默认自动配置类为EnableAutoConfiguration**

#### 2\.2.3 SpringFactoriesLoader

SpringFactoriesLoader工厂加载机制是Spring内部提供的一个约定俗成的加载方式，只需要在模块的META-INF/spring.factories文件，这个Properties格式的文件中的key是接口、注解、或抽象类的全名，value是以逗号 “ , “ 分隔的实现类，使用SpringFactoriesLoader来实现相应的实现类注入Spirng容器中。

*会加载所有jar包下的classpath路径下的META-INF/spring.factories文件，这样文件不止一个。*

#### 2\.2.4 loadFactoryNames()

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
   ClassLoader classLoaderToUse = classLoader;
   if (classLoaderToUse == null) {
      classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
   }
   String factoryTypeName = factoryType.getName();
   return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

![在这里插入图片描述](<image/7950ed1c71534678a14f65d1b9f5a278.png>)

先是将 EnableAutoConfiguration.class 传给了 factoryType

然后String factoryTypeName = factoryType.getName();，所以factoryTypeName 值为 org.springframework.boot.autoconfigure.EnableAutoConfiguration

#### 2\.2.5 loadSpringFactories()

接着查看loadSpringFactories方法的作用

```java
 private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
   //断点查看
   MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
   if (result != null) {
     return result;
   } else {
     try {
       //注意这里：META-INF/spring.factories
       Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
       LinkedMultiValueMap result = new LinkedMultiValueMap();
       while(urls.hasMoreElements()) {
         URL url = (URL)urls.nextElement();
         UrlResource resource = new UrlResource(url);
         Properties properties = PropertiesLoaderUtils.loadProperties(resource);
         Iterator var6 = properties.entrySet().iterator();
         while(var6.hasNext()) {
           Entry<?, ?> entry = (Entry)var6.next();
           String factoryTypeName = ((String)entry.getKey()).trim();
           String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
           int var10 = var9.length;
           for(int var11 = 0; var11 < var10; ++var11) {
             String factoryImplementationName = var9[var11];
             result.add(factoryTypeName, factoryImplementationName.trim());
           }
         }
       }

       cache.put(classLoader, result);
       return result;
     } catch (IOException var13) {
       throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var13);
     }
   }
 }
```

**这里的 FACTORIES\_RESOURCE\_LOCATION 在上面有定义：META-INF/spring.factories**

```java
public final class SpringFactoriesLoader {

    /**
      * The location to look for factories.
      * <p>Can be present in multiple JAR files.
      */
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
}
```

META-INF/spring.factories文件在哪里呢？—— 在所有引入的java包的当前类路径下的META-INF/spring.factories文件都会被读取，如：

![在这里插入图片描述](<image/13994b89f5ca4d7392262bfaa16ff491.png>)

 断点查看result值如下：

![在这里插入图片描述](<image/07fda268f2bb4aae80681f0de9c3fa79.png>)

该方法作用是加载所有依赖的路径META-INF/spring.factories文件，通过map结构保存，key为文件中定义的一些标识工厂类，value就是能自动配置的一些工厂实现的类，value用list保存并去重。

在回看 loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());

因为 loadFactoryNames 方法携带过来的第一个参数为 EnableAutoConfiguration.class，所以 factoryType 值也为 EnableAutoConfiguration.class，那么 factoryTypeName 值为 EnableAutoConfiguration。拿到的值就是META-INF/spring.factories文件下的key为 org.springframework.boot.autoconfigure.EnableAutoConfiguration的值

getOrDefault 当 Map 集合中有这个 key 时，就使用这个 key值，如果没有就使用默认值空数组

**结论：**

loadSpringFactories()该方法就是从“META-INF/spring.factories”中加载给定类型的工厂实现的完全限定类名放到map中

loadFactoryNames()是根据SpringBoot的启动生命流程，当需要加载自动配置类时，就会传入org.springframework.boot.autoconfigure.EnableAutoConfiguration参数，从map中查找key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的值，这些值通过反射加到容器中，之后的作用就是用它们来做自动配置，这就是Springboot自动配置开始的地方

只有这些自动配置类进入到容器中以后，接下来这个自动配置类才开始进行启动

当需要其他的配置时如监听相关配置：listenter，就传不同的参数，获取相关的listenter配置。

## 最后 再给大家附上两张总结流程图

![在这里插入图片描述](<image/7b737cc920184a82b3a3be1e5fc85680.png>)

![在这里插入图片描述](<image/d54db2feb293450bbf93ba4641e2a51f.png>)