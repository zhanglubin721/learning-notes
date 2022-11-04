# SpringBoot工作原理简谈

**1，为什么依赖的依赖变少了？SpringBoot是如何管理这些依赖的？**

我们分两个点来看起

**1.1 从pom文件出发**

首先，是有一个父工程的引用

![img](image/v2-2e85e0c4c2ccf8d0dfdc658c434f1f92_1440w.jpg)

我们继续往里面跟踪，发现父工程又依赖于另一个父工程

![img](image/v2-49addf33aeee75a6c12fc74872ecd2d2_1440w.jpg)

继续跟踪，发现这是一个pom工程，统一控制版本

![img](image/v2-f0c8a73b066a30554562ef92b59882fa_1440w.jpg)

定义了一堆第三方jar包的版本

![img](image/v2-01e3285928e9826d1491811b72e9c173_1440w.jpg)

**结论：**

所有我们使用了SpringBoot之后，由于父工程有对版本的统一控制，所以大部分第三方包，我们无需关注版本，个别没有纳入SpringBoot管理的，才需要设置版本号

**1.2 SpringBoot将所有的常见开发功能，分成了一个个场景启动器（starter），这样我们需要开发什么功能，就导入什么场景启动器依赖即可。**

比如，我们现在要开发web项目，所以我们导入了spring-boot-starter-web

![img](image/v2-d9979747317f80869df08bdb94d090e9_1440w.jpeg)

我们来跟踪看看，内部也复用一些starter

![img](image/v2-4e3487f5ee08f0a26ebf82bd058edae7_1440w.webp)

还有Springweb和SpringMVC，这也就是为什么，我们就可以开发SpringWeb程序的原因

![img](image/v2-0e88033da25570547daa94950d308d44_1440w.webp)

**结论：**

- 大家会发现，SpringBoot是通过定义各种各样的Starter来管理这些依赖的
- 比如，我们需要开发web的功能，那么引入spring-boot-starter-web
- 比如，我们需要开发模板页的功能，那么引入spring-boot-starter-thymeleaf
- 我们需要整合redis，那么引入spring-boot-starter-data-redis
- 我们需要整合amqp，实现异步消息通信机制，那么引入spring-boot-starter-amqp
- 等等，就是这么方便

**2，为什么我们不需要配置？**

我们来看看SpringBoot的启动类代码，除了一个关键的注解，其他都是普通的类和main方法定义

![img](image/v2-8d261b9d935e98a8bf910239931546ed_1440w.jpeg)

那么，我们来观察下这个注解背后的东西，发现，这个注解是一个复合注解，包含了很多的信息

![img](image/v2-a67886bf1ee067d6f9939bfd36546a7d_1440w.webp)

其他注解都是一个注解的常规配置，所以关键看圈中的这两个

**我们来分析第一个关键注解：@SpringBootConfiguration**

我们可以看到，内部是包含了@Configuration，这是Spring定义配置类的注解

而@Configuration实际上就是一个@Component，表示一个受Spring管理的组件

![img](image/v2-7f8a5d11c5a46ed6697b91de274d92a0_1440w.jpeg)

**结论：@SpringBootConfiguration这个注解只是更好区分这是SpringBoot的配置注解，本质还是用了Spring提供的@Configuration注解**

**我们再来探讨下一个注解：@EnableAutoConfiguration**

这个注解的作用是告诉SpringBoot开启自动配置功能，这样就减少了我们的配置

那么具体是怎么实现自动配置的？

我们先来观察这个注解背后的内容

![img](image/v2-c7af23a595c1d301907d8abfe34c2689_1440w.webp)

所以，又到了分析圈中的两个注解了

**先来分析@AutoConfigurationPackage**

观察其内部实现，内部是采用了@Import，来给容器导入一个Registrar组件

![img](image/v2-f4fc91431709815748f4743160bf626a_1440w.jpeg)

所以，我们继续往下跟踪，来看Registrar内部是什么情况？

![img](image/v2-a93f83f34cd21c13701ad55ea8b7212e_1440w.webp)

我们可以跟踪源码看看这段是什么信息

![img](image/v2-260515784d191c4e149a0b8e87151025_1440w.webp)

**结论：**

通过源码跟踪，我们知道，程序运行到这里，会去加载启动类所在包下面的所有类

这就是为什么，默认情况下，我们要求定义的类，比如controller，service必须在启动类的同级目录或子级目录的原因

**再来分析@Import(AutoConfigurationImportSelector.class)**

这个的关键是来看AutoConfigurationImportSelector.class内部的细节

在这个类的内部，有一个关键的方法，我们可以调试来看看结果

![img](image/v2-37063c2b7b7e2666841eaef6ca79b4b0_1440w.webp)

发现默认加载了好多的自动配置类，这些自动配置类，会自动给我们加载每个场景所需的所有组件，并配置好这些组件，这样就省去了很多的配置

![img](image/v2-c177a7dd2b53dea8c74d82a28afcac87_1440w.webp)