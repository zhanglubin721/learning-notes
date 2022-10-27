# Spring中bean的注入方式

平常的Java开发中，程序员在某个类中需要依赖其它类的方法。 通常是new一个依赖类的实例再调用该实例的方法，这种开发存在的问题是new的类实例不好统一管理。

Spring提出了[依赖注入](<https://so.csdn.net/so/search?q=%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5&spm=1001.2101.3001.7020>)的思想，即依赖类不由程序员实例化，而是通过Spring容器帮我们new指定实例并且将实例注入到需要该对象的类中。

依赖注入的另一种说法是”[控制反转](<https://so.csdn.net/so/search?q=%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC&spm=1001.2101.3001.7020>)”。通俗的理解是：平常我们new一个实例，这个实例的控制权是我们程序员。而控制反转是指new实例工作不由我们程序员来做而是交给Spring容器来做。

Spring有多种依赖注入的形式，本篇文章仅介绍Spring通过xml进行IOC配置的方式。

**1、Set()注入：**

这是最简单的注入方式，假设有一个SpringAction，类中需要实例化一个SpringDao对象，那么就可以定义一个private的SpringDao成员变量，然后创建SpringDao的set方法（这是ioc的注入入口）：

```java
package com.bless.springdemo.action; 
public class SpringAction { 
    //注入对象springDao 
    private SpringDao springDao; 
    //一定要写被注入对象的set方法 
    public void setSpringDao(SpringDao springDao) { 
    this.springDao = springDao; 
} 
 
public void ok(){ 
    springDao.ok(); 
}
```

随后编写spring的xml文件中，name属性是class属性的一个别名，class属性指类的全名，因为在SpringAction中有一个公共属性Springdao，所以要在标签中创建一个标签指定SpringDao。标签中的name就是SpringAction类中的SpringDao属性名，ref指下面，这样其实是spring将SpringDaoImpl对象实例化并且调用SpringAction的setSpringDao方法将SpringDao注入：

```xml
<!--配置bean,配置后该类由spring管理--> 
<bean name="springAction" class="com.bless.springdemo.action.SpringAction"> 
<!--(1)依赖注入,配置当前类中相应的属性--> 
<property name="springDao" ref="springDao"></property> 
</bean> 
<bean name="springDao" class="com.bless.springdemo.dao.impl.SpringDaoImpl"></bean>
```

**2、构造器注入：**

这种方式的注入是指带有参数的构造函数注入，看下面的例子，我创建了两个成员变量SpringDao和User，但是并未设置对象的set方法，所以就不能支持第一种注入方式，这里的注入方式是在SpringAction的构造函数中注入，也就是说在创建SpringAction对象时要将SpringDao和User两个参数值传进来：

```java
public class SpringAction { 
    //注入对象springDao 
    private SpringDao springDao; 
    private User user; 
 
    public SpringAction(SpringDao springDao,User user){ 
    this.springDao = springDao; 
    this.user = user; 
    System.out.println("构造方法调用springDao和user"); 
} 
 
public void save(){ 
    user.setName("卡卡"); 
    springDao.save(user); 
}
```

在XML文件中同样不用的形式，而是使用标签，ref属性同样指向其它标签的name属性：

```xml
<!--配置bean,配置后该类由spring管理--> 
<bean name="springAction" class="com.bless.springdemo.action.SpringAction"> 
<!--(2)创建构造器注入,如果主类有带参的构造方法则需添加此配置--> 
<constructor-arg ref="springDao"></constructor-arg> 
<constructor-arg ref="user"></constructor-arg> 
</bean> 
<bean name="springDao" class="com.bless.springdemo.dao.impl.SpringDaoImpl"></bean> 
<bean name="user" class="com.bless.springdemo.vo.User"></bean> 
```

解决构造方法参数的不确定性：你可能会遇到构造方法传入的两参数都是同类型的，为了分清哪个该赋对应值，则需要进行一些小处理：

下面是设置index，就是参数位置：

```xml
<bean name="springAction" class="com.bless.springdemo.action.SpringAction"> 
		<constructor-arg index="0" ref="springDao"></constructor-arg> 
		<constructor-arg index="1" ref="user"></constructor-arg> 
</bean> 
```

另一种是设置参数类型：

```xml
<constructor-arg type="java.lang.String" ref=""/>
```

**3、静态工厂的方法注入：**

静态工厂顾名思义，就是通过调用静态工厂的方法来获取自己需要的对象，为了让spring管理所有对象，我们不能直接通过”工程类.静态方法()”来获取对象，而是依然通过spring注入的形式获取：

```java
package com.bless.springdemo.factory; 
 
import com.bless.springdemo.dao.FactoryDao; 
import com.bless.springdemo.dao.impl.FactoryDaoImpl; 
import com.bless.springdemo.dao.impl.StaticFacotryDaoImpl; 
 
public class DaoFactory { 
//静态工厂 
public static final FactoryDao getStaticFactoryDaoImpl(){ 
    return new StaticFacotryDaoImpl(); 
    } 
}
```

同样看关键类，这里我需要注入一个FactoryDao对象，这里看起来跟第一种注入一模一样，但是看随后的xml会发现有很大差别:

```java
public class SpringAction { 
    //注入对象 
    private FactoryDao staticFactoryDao; 

    public void staticFactoryOk(){ 
        staticFactoryDao.saveFactory(); 
    } 
    //注入对象的set方法 
    public void setStaticFactoryDao(FactoryDao staticFactoryDao) { 
        this.staticFactoryDao = staticFactoryDao; 
    }
}
```

Spring的IOC配置文件，注意看指向的class并不是FactoryDao的实现类，而是指向静态工厂DaoFactory，并且配置 factory-method=”getStaticFactoryDaoImpl”指定调用哪个工厂方法：

```xml
<!--配置bean,配置后该类由spring管理--> 
<bean name="springAction" class="com.bless.springdemo.action.SpringAction" > 
    <!--(3)使用静态工厂的方法注入对象,对应下面的配置文件(3)--> 
    <property name="staticFactoryDao" ref="staticFactoryDao"></property> 
    </property> 
</bean> 
<!--(3)此处获取对象的方式是从工厂类中获取静态方法--> 
<bean name="staticFactoryDao" class="com.bless.springdemo.factory.DaoFactory" factory-method="getStaticFactoryDaoImpl">
</bean> 
```

**4、实例工厂的方法注入：**

实例工厂的意思是获取对象实例的方法不是静态的，所以你需要首先new工厂类，再调用普通的实例方法：

```java
public class DaoFactory { 
    //实例工厂 
    public FactoryDao getFactoryDaoImpl(){ 
        return new FactoryDaoImpl(); 
    } 
}
```

那么下面这个类没什么说的，跟前面也很相似，但是我们需要通过实例工厂类创建FactoryDao对象：

```java
public class SpringAction { 
		//注入对象 
    private FactoryDao factoryDao; 

    public void factoryOk(){ 
        factoryDao.saveFactory(); 
    } 

    public void setFactoryDao(FactoryDao factoryDao) { 
    		this.factoryDao = factoryDao; 
    } 
} 
```

最后看spring配置文件：

```xml
<!--配置bean,配置后该类由spring管理--> 
<bean name="springAction" class="com.bless.springdemo.action.SpringAction"> 
		<!--(4)使用实例工厂的方法注入对象,对应下面的配置文件(4)--> 
		<property name="factoryDao" ref="factoryDao"></property> 
</bean> 
 
 
<!--(4)此处获取对象的方式是从工厂类中获取实例方法--> 
<bean name="daoFactory" class="com.bless.springdemo.factory.DaoFactory"></bean> 
<bean name="factoryDao" factory-bean="daoFactory" factory-method="getFactoryDaoImpl"></bean> 
```

**5\.总结：**

Spring IOC注入方式用得最多的是(1)(2)种，多写多练就会非常熟练。

另外注意：通过Spring创建的对象默认是单例的，如果需要创建多实例对象可以在标签后面添加一个属性：

```java
<bean name="..." class="..." scope="prototype">
```

