---
title: Spring 事务控制
date: 2020-06-15 09:07:24
tags:
  - spring
  - 事务
---

编程式事务控制：自己编写代码实现事务控制

声明式事务控制：通过配置方式实现事务控制，解耦合

运用到了AOP思想，Spring声明式事务控制底层就是AOP，切点是业务方法，通知是事务控制方法，通过在xml中织入可以形成切面

<!--more-->

#### 基于XML的Spring事务控制

```java
package com.it.service.impl;

import com.it.dao.AccountDao;
import com.it.service.AccountService;

public class AccountServiceImpl implements AccountService {
    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao){this.accountDao = accountDao;}

    //outMan把钱转给inMan
    public void transfer(String outMan, String inMan, double money) {
        accountDao.out(outMan, money);
        accountDao.in(inMan, money);
    }
}
```

转账业务里，如果out执行后发生了异常，in得不到执行，会导致钱的不平衡

声明式事务控制：1.平台事务管理器配置 2.事务通知配置 3.事务aop织入配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

<!--配置数据源对象-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3306/test?serverTimezone=GMT"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
<!--配置Jdbc模板对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <bean id="accountDao" class="com.it.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>

<!--目标对象，切点-->
    <bean id="accountService" class="com.it.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>

<!--配置平台事务管理器-->
    <bean id="transactionManger" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 会从dataSource里getConnection()-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
<!--通知 事务的增强-->
    <tx:advice id="txAdvice" transaction-manager="transactionManger">
        <!--设置事务的属性信息-->
        <tx:attributes>
            <tx:method name="tranfer" isolation="DEFAULT" read-only="false"></tx:method>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

<!--配置事务AOP织入 事务的增强用aop:advisor-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.it.service.impl.*.*(..))"></aop:advisor>
    </aop:config>
</beans>
```

进行事务控制后，上面的转账代码出现异常，也不会出现outMan钱少InMan钱不变的情况，结果会是OutMan和InMan的钱都没动

平台事务管理器根据需求变，不同的Dao层用不同的事务管理器

<tx:attributes>里面用tx:method代表切点方法事务的具体参数配置，*代表所有方法，后面可以配置事务属性isolation隔离级别，read-only只读属性等等

配置AOP织入时，不用aop:aspect，用专门事务增强的ad:advisor

#### 基于注解的Spring事务控制

```java
package com.it.service.impl;

import com.it.dao.AccountDao;
import com.it.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Transactional;

@Service("accountService")
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;
    public void setAccountDao(AccountDao accountDao){this.accountDao = accountDao;}

    @Transactional(isolation = Isolation.DEFAULT,readOnly = false)
    //outMan把钱转给inMan
    public void transfer(String outMan, String inMan, double money) {
        accountDao.out(outMan, money);
        int i = 1/0;
        accountDao.in(inMan, money);
    }
}
```

@Transactional事务注解，写在切点方法上，可以指定事务属性

xml中要增加事务注解驱动，读事务注解

```java
<!-- 事务注解驱动-->
    <tx:annotation-driven transaction-manager="transactionManger"></tx:annotation-driven>
```

### Spring集成Web环境

应用上下文对象通过new ClassPathXmlApplicatonContext(Spring配置文件)方式获取的，但是每次从容器中获得Bean时都要编写这个语句，使得配置文件加载多次，应用上下文对象创建多次

在Web项目中，可以使用ServletContextListener监听Web项目的启动，因此我们可以在Web项目启动时就加载Spring配置文件，创建应用上下文对象ApplicationContext，再将其存储到最大的域servletContext，这样就可以在任意位置从域中获得应用上下文对象了

现在Spring提供了一个监听器ContextLoaderListener就是对上述功能的封装

1.在web.xml中配置ContextLoaderListener(导入spring-web坐标)

2.使用WebApplicationContextUtils获得应用对象上下文ApplicationContext

web.xml

```java
<!--全局初始化参数-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
<!--  配置监听器-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```

```java
package com.it.web;
import com.it.service.UserService;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class UserServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        ServletContext servletContext = this.getServletContext();
        //ApplicationContext app = (ApplicationContext) servletContext.getAttribute("app");
        //ApplicationContext app = WebApplicationContextUtils.getWebApplicationContext(servletContext);
        WebApplicationContext app = WebApplicationContextUtils.getWebApplicationContext(servletContext);
        UserService userService = app.getBean(UserService.class);
        userService.save();
    }
}
```

