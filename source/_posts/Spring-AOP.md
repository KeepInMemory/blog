---
title: Spring AOP
date: 2020-06-13 19:23:42
tags:
  - spring
  - AOP
---

作用：在程序运行期间，在不修改源码的情况下对方法进行功能增强

优势：减少重复代码，提高开发效率，便于维护

面向切面编程的切面：目标方法以及其增强方法结合在一起叫做一个切面

**现在实现松耦合都是使用配置文件的方式**

AOP的底层实现：通过spring提供的动态代理技术(基于JDK的动态代理+基于Cglib的动态代理)。在运行期间，Spring通过动态代理技术动态的生成代理对象，代理对象方法执行时进行增强功能的介入，在去调用目标对象的方法，从而完成功能的增强。

<!--more-->

### 简介

Target(目标对象)：代理的目标对象

Proxy(代理)：一个类被AOP织入增强后，就产生一个结果代理类

Joinpoint(连接点)：指可以被增强的方法，因为在Spring中只支持方法类型的连接点

Pointcut(切入点)：指真正被增强的方法

例如有100个目标对象，里面的方法都可以增强，都是连接点，但真正被增强的方法才是切入点

Advice(通知/增强)：拦截到连接点之后要做的就是通知

Aspect(切面)：切入点+通知

Weaving(织入)：将切点和通知结合的过程叫做织入过程

编写内容：编写核心业务代码（目标类的目标方法），编写切面类（通知），在配置文件中配置织入关系

AOP实现内容：一旦监控到Spring框架切入点方法的执行，使用代理技术，动态创建目标对象的代理对象，根据通知类别，在代理对象的对应位置，将通知对应的功能织入

### 基于XML的AOP开发

#### 简单例子

1.导入AOP相关坐标 2.创建目标接口和目标类 3.创建切面类（内部有增强方法） 4.将目标类和切面类交给Spring容器 5.在applicationContext.xml中配置织入关系

Spring本身有AOP的实现，第三方的配置是AspectJ，Spring发现AspectJ更好，主张使用AspectJ

1.导坐标

```java
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.2.7.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.4</version>
    </dependency>
```

2.目标接口和目标类

```java
package com.it.aop;

public interface TargetInterface {
    public void save();
}
```

```java
package com.it.aop;

public class Target implements TargetInterface{
    @Override
    public void save() {
        System.out.println("save running");
    }
}
```

3.切面类

```java
package com.it.aop;

public class MyAspect {
    public void before(){
        System.out.println("前置增强");
    }
}
```

4.交给Spring容器

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--目标对象-->
    <bean id="target" class="com.it.aop.Target"></bean>

    <!--切面对象-->
    <bean id="myAspect" class="com.it.aop.MyAspect"></bean>

    <!--配置织入：告诉Spring哪些方法需要哪些增强(前置增强/后置增强等等)-->
    <aop:config>
        <!--声明切面类是谁-->
        <aop:aspect ref="myAspect">
            <!--切面=切点+通知-->
            <aop:before method="before" pointcut="execution(public void com.it.aop.Target.save())"></aop:before>
        </aop:aspect>

    </aop:config>
</beans>
```

需要首先添加aop命名空间,切点表达式的格式是execution(修饰符 返回值类型 全限定名(参数)),这里面修饰符可以省略,全限定名里可以用*号表示任意,参数用..表示任意参数

很多时候切点表达式是一样的,所以进行切点表达式的抽取

使用<aop:pointcut id="myPointcut" expression="execution(* com.it.aop.*.*(..))">

<aop:before method="before" pointcut-ref="myPointcut">

5.测试

```java
package com.it.test;

import com.it.aop.TargetInterface;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class AOPTest {
    @Autowired
    private TargetInterface target;

    @Test
    public void test1(){
        target.save();
    }
}
```

输出为

前置增强
save running

#### 通知类型

|     名称     |         标签          |                  说明                  |
| :----------: | :-------------------: | :------------------------------------: |
|   前置通知   |     <aop:before>      |    指定增强方法在切入点方法之前执行    |
|   后置通知   | <aop:after-returning> |    指定增强方法在切入点方法之后执行    |
|   软绕通知   |     <aop:around>      | 指定增强方法在切入点方法之前之后都执行 |
| 异常抛出通知 |    <aop:throwing>     |      指定增强方法在出现异常时执行      |
|   最终通知   |      <aop:after>      |     无论是否有异常都会执行增强方法     |

### 基于注解的AOP开发

Target.java

```java
package com.it.anno;

import org.springframework.stereotype.Component;

@Component("target")
public class Target implements TargetInterface{
    @Override
    public void save() {
        System.out.println("save running");
    }
}
```

Aspect.java

```java
package com.it.anno;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Component("myAspect")//交给Spring容器
@Aspect//告诉Spring标注当前类是一个切面类
public class MyAspect {

    @Before("execution(public void com.it.anno.Target.*())")//配置前置通知,需要指定切点表达式
    public void before(){
        System.out.println("前置增强");
    }

    public void around(ProceedingJoinPoint pjp)throws Throwable{
        System.out.println("环绕前增强");
        pjp.proceed();
        System.out.println("环绕后增强");
    }
}
```

applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd">
<!--组件扫描-->
    <context:component-scan base-package="com.it.anno"></context:component-scan>
<!--Aop自动代理-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

加Aop自动代理才会去识别扫描包里的@Aspect@Before等等的注解

AnnoTest.java

```java
package com.it.test;

import com.it.anno.TargetInterface;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")

public class AnnoTest {
    @Autowired
    private TargetInterface target;

    @Test
    public void test(){
        target.save();
    }
}
```

**AnnoTest加载SpringJuint4ClassRunner的核心测试,加载applicationContext核心配置**

**在核心配置里去进行组件扫描,扫描相应的包的注解,将对象交给Spring容器**

**同时Aop的自动代理会识别包内的Aop注解**

#### 通知注解类型

前置通知:@Before

后置通知:@AfterReturning

环绕通知:@Around

异常抛出通知:AfterThrowing

最终通知:@After

### 