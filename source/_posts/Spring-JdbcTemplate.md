---
title: Spring JdbcTemplate
date: 2020-06-14 09:55:57
tags:
  - spring
  - Jdbc
---

Spring对Jdbc API的简单封装,还有很多相似的模板类,例如RedisTemplate

开发步骤:1.导入Spring-jdbc和Spring-tx坐标 2.创建数据库实体 3.创建JdbcTemplate对象 4.执行数据库操作

<!--more-->

```java
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.0.5.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.0.5.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.1.2</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.20</version>
    </dependency>
```

```java
package com.it.test;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.junit.Test;
import org.springframework.jdbc.core.JdbcTemplate;

import java.beans.PropertyVetoException;

public class JdbcTemplateTest {
    @Test
    //测试Jdbc开发步骤
    public void test1() throws PropertyVetoException {
        //创建数据源对象
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/test?serverTimezone=GMT");
        dataSource.setUser("root");
        dataSource.setPassword("root");
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //设置数据源对象
        jdbcTemplate.setDataSource(dataSource);
        //执行操作
        int row = jdbcTemplate.update("insert into account values(?,?)","tom",5000);
        System.out.println(row);
    }
}
```

jdbc:mysql://127.0.0.1:3306/test?serverTimezone=GMT里的test是数据库名,加?serverTimezone=GMT解决时区错误

### Spring产生JdbcTemplate

applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

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
</beans>
```

JdbcTemplateTest.java

```java
package com.it.test;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.beans.PropertyVetoException;

public class JdbcTemplateTest {
    @Test
    //测试Jdbc开发步骤
    public void test1() throws PropertyVetoException {
        //创建数据源对象
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/test?serverTimezone=GMT");
        dataSource.setUser("root");
        dataSource.setPassword("root");
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //设置数据源对象
        jdbcTemplate.setDataSource(dataSource);
        //执行操作
        int row = jdbcTemplate.update("insert into account values(?,?)","tom",5000);
        System.out.println(row);
    }
    @Test
    //测试Spring产生Jdbc模板对象
    public void test2(){
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        DataSource dataSource = (DataSource) app.getBean("dataSource");
        JdbcTemplate jdbcTemplate = (JdbcTemplate)app.getBean(JdbcTemplate.class);
        int row = jdbcTemplate.update("insert into account values(?,?)","amy",3000);
        System.out.println(row);
    }
}
```

### JdbcTemplate常用操作

#### update更新操作

```java
package com.it.test;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class JdbcTemplateCRUDTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Test
    public void testUpdate(){
        jdbcTemplate.update("update account set money= ? where name = ?",10000,"amy");
    }
    @Test
    public void testDelete(){
        jdbcTemplate.update("delete from account where name = ?","tom");
    }
}
```

用Spring整合Juint测试,测试update和delete操作

#### query查询操作

```java
package com.it.test;

import com.it.domain.Account;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class JdbcTemplateCRUDTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Test
    public void testUpdate(){
        jdbcTemplate.update("update account set money= ? where name = ?",10000,"amy");
    }
    @Test
    public void testDelete(){
        jdbcTemplate.update("delete from account where name = ?","tom");
    }
    @Test
    public void testQueryAll(){
        List<Account> accountList = jdbcTemplate.query("select * from account",new BeanPropertyRowMapper<Account>(Account.class));
        System.out.println(accountList);
    }
    @Test
    public void testQueryOne(){
        Account account = jdbcTemplate.queryForObject("select * from account where name = ?",new BeanPropertyRowMapper<Account>(Account.class),"amy");
        System.out.println(account);
    }
    @Test
    public void testQueryCount(){
        Long count = jdbcTemplate.queryForObject("select count(*) from count",Long.class);
        System.out.println(count);
    }
}
```

如果要返回的值是个实体,就需要BeanPropertyRowMapper<Account>(Account.class)进行映射,让方法帮我封装实体

如果要返回的值是个简单值,就直接传Long.class,不用上面那么麻烦

queryForObject是用来查询单个结果,query查询多个结果