---
title: Spring IOC
date: 2020-06-12 14:29:04
tags:
  - spring
  - IOC
---

#### 简介

IOC:Inversion Of Control控制反转

​	控制：资源的获取方式由主动的new变为被动的从容器接收，容器相当于婚介所

DI：依赖注入

​	容器能知道哪个组件运行的时候需要另一个组件；容器通过反射的形式，将容器准备好的对象注入（利用反射给属性赋值）

<!--more-->

![image-20200612150607383](C:\Users\75959\AppData\Roaming\Typora\typora-user-images\image-20200612150607383.png)

1.导入Spring开发的基本包坐标 2.编写Dao接口和实现类 3.创建Spring核心配置文件 4.在Spring配置文件中配置UserDaoImpl 5.使用Spring的API获得Bean实例

#### Bean属性

**scope**

```java
<bean id="userDao" class="com.it.dao.impl.UserDaoImpl" scope="singleton"></bean>
```

singleton单例的实例化是在Spring的xml文件加载时

```java
<bean id="userDao" class="com.it.dao.impl.UserDaoImpl" scope="prototype"></bean>
```

prototype多例的实例化是在每次getBean时

**init-method&destory-method**

```java
<bean id="userDao" class="com.it.dao.impl.UserDaoImpl" init-method="init" destroy-method="destory"></bean>
```

指定初始化和销毁方法

#### Bean初始化三种方法

**1.无参的构造方法**

**2.工厂静态方法**

```java
<bean id="userDao" class="com.it.factory.StaticFactory" factory-method="getUserDao"></bean>
```

全包名换成静态工厂的名，并指定factory-method属性，否则就找的是静态工厂的无参构造方法了

**3.工厂实例方法**

和静态方法的区别在于工厂的实例化方法去掉了static，这样就需要实例工厂对象，才能调用实例user的函数

```java
<bean id="factory" class="com.it.factory.DynamicFactory"></bean>
<bean id = "userDao" factory-bean="factory" factory-method="getUserDao"></bean>
```

先用一个bean去实例化工厂对象，再用bean指定userDao的factory-bean和factory-method

#### Bean的依赖注入方式

**set方法**

UserServiceImpl的实例的save方法需要调用UserDao的save方法，不是通过在userServiceImpl的save方法里

```java
ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
serDao userDao1 = (UserDao) app.getBean("userDao");
```

而是通过在UserServiceImpl类里加一个UserDao的对象，然后通过setUserDao(userDao)方法赋值，这里的userDao的通过Spring容器接收过来的

```java
<bean id="userService" class="com.it.service.impl.UserServiceImpl">
```

在userService的bean下面添加property属性

```java
<bean id="userService" class="com.it.service.impl.UserServiceImpl">
	<property name="userDao" ref="userDao"></property>//依赖注入
```

name的userDao是setUserDao的set后面的，第一个字母要小写

ref的userDao是上面的

```java
<bean id="userDao" class="com.it.dao.impl.UserDaoImpl" scope="singleton"></bean>
```

意思是UserServiceImpl有名为userDao的属性，对应setUserDao的UserDao，引用是userDao的Bean的id

**这里的UserServiceImpl内部就有UserDao了，但是UserServiceImpl必须从容器中拿，自己new内部的UserDao会是空指针，因为property是写在UserServiceImpl的Bean下面的，没有Bean的实例化就没有set方法的依赖注入**

set方法的简单方式是使用p命名空间

**构造方法**

在UserServiceImpl类里加有参构造方法

```java
public UserServiceImpl(UserDao userDao){
	this.userDao = userDao;
}
```

```java
<Bean id="userService" class="com.it.service.impl.UserServiceImpl">
	<constructor-arg name="userDao" ref="userDao"></constructor-arg>
</Bean>
```

constructor-arg的意思是构造参数，含义是在userServiceImpl里有一个名为userDao的构造参数，是个引用类型，引用的是XML里的id为userDao的Bean

#### Bean的依赖注入的数据类型

**引用数据类型**

上面的内容向UserServiceImpl注入的UserDao都是引用类型

**普通数据类型**

UserDao里添加String username，int age以及相应的Set方法

XML的Property的ref换成value，后面跟的是要赋的值 

**集合数据类型**

```java
<bean id="userDao" class="com.it.dao.impl.UserDaoImpl"></bean>
	<property name="strList">
		<list>
			<value>aaa</value>
			<value>bbb</value>
		</list>
	</property>
</bean>
```

通过使用property里的子标签，集合是List就用<List>,是Map就用<Map>。类似的是List<String>就用<value>，是List<userDao>就用<ref>

**依赖注入完整实例**

xml文件

```java
    <bean id = "userDao" factory-bean="factory" factory-method="getUserDao"></bean>
    <bean id="userService" class="com.it.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"></property>
    </bean>
```

dao-UserDao

```java
package com.it.dao;

public interface UserDao {
    public void save();

}
```

dao-impl-UserDaoImpl

```java
package com.it.dao.impl;

import com.it.dao.UserDao;

public class UserDaoImpl implements UserDao{
    @Override
    public void save() {
        System.out.println("save running");
    }
}
```

service-UserService

```java
package com.it.service;

public interface UserService {
    public void save();
}
```

service-impl-UserServiceImpl

```java
package com.it.service.impl;
import com.it.dao.UserDao;
import com.it.service.UserService;

public class UserServiceImpl implements UserService {
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save() {
        userDao.save();
    }
}
```

web-UserController

```java
package com.it.web;

import com.it.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserController {
    public static void main(String[] args) {
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = app.getBean(UserService.class);
        userService.save();
    }
}
```



#### Bean小结

```java
<bean>标签
	id属性：在容器中Bean实例的唯一标识，不允许重复
	class属性：要实例化的Bean的全限定名
	scope属性：Bean的作用范围，默认是Singleton
	<property>标签：属性注入
		name属性：属性名称
		value属性：普通类型的属性值
		ref属性：引用对象的Bean的id
		<list>标签
		<map>标签
    	...
		<properties>标签
	<constructor-arg>标签：带参构造注入
    	...
	<import>标签：导入其他的xml分文件
```

#### Spring相关API

**ApplicationContext的实现类**

1.ClassPathXmlApplicationContext：从类加载路径(resources)下加载配置文件

2.FilePathXmlApplicationContext：从磁盘路径下加载配置文件

3.AnnotationConfigApplicationContext：当使用注解配置容器对象时，需要用此类来创建spring容器，用来读取注解

**getBean()两种方法**

1.getBean(string)，例如getBean("userDao")

2.getBean(class)，例如getBean(UserDao.class)，仅限于xml中只有一个<bean ... class="...UserDao">

#### Spring配置数据源

**数据源/连接池作用**

使用连接资源时从数据源中获取，使用完后将连接资源还给数据源

常见的数据源：DBCP、C3P0、Druid

**整合c3p0数据源**

```java
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.32</version>
</dependency>

<dependency>
	<groupId>c3p0</groupId>
	<artifactId>c3p0</artifactId>
	<version>0.9.1.2</version>
</dependency>
```



```java
   //测试手动创建c3p0
    public void test1() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();

        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mybatis");
        dataSource.setUser("root");
        dataSource.setPassword("root");

        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

**整合druid数据源**

```java
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.32</version>
</dependency>

<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
	<version>1.1.10</version>
</dependency>
```

```java
    public void test2() throws Exception{
        DruidDataSource dataSource = new DruidDataSource();

        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("root");

        DruidPooledConnection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

这里面的DriverClass、Url、Username、Passoword是必须设置的，想连接数据库必须配置这些参数。但是在方法里set会产生耦合，解耦合的方法是在property里设置

**解耦合到配置文件**

在resources里新建file，名字为jdbc.properties，内容如下

```java
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.username=root
jdbc.password=root
```

```java
    public void test3() throws Exception{
        //读取配置文件
        ResourceBundle rb = ResourceBundle.getBundle("jdbc");
        String driver = rb.getString("jdbc.driver");
        String url = rb.getString("jdbc.url");
        String username = rb.getString("jdbc.username");
        String password = rb.getString("jdbc.password");

        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);

        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

getBundle需要的字符串名是类加载目录下(resources文件下的properties文件的前缀，即jdbc)

**spring配置**

spring的步骤是1.导入Spring开发的基本包坐标 2.编写接口和实现类 3.创建Spring核心配置文件 4.在Spring配置文件中配置 5.使用Spring的API获得Bean实例

这里的第2布已经由jar包完成了

```java
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
```

```java
public void test4()throws Exception{
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        DataSource dataSource = app.getBean(ComboPooledDataSource.class);
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

property name的值是set函数后面的部分，即对应

```java
dataSource.setDriverClass(driver);
dataSource.setJdbcUrl(url);
dataSource.setUser(username);
dataSource.setPassword(password);
```

但现在还是有个同样的问题就是value的值和xml耦合了，现在想办法让xml加载properties文件

**解耦合**

xml里引入properties命名空间

```java
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"//新增这一行context
```

引入约束路径

```java
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">//新增context
```

property标签改成

```java
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>
```

${}内对应的是jdbc.properties里的内容

#### Spring注解开发

注解代替xml配置，可以简化配置，提高开发效率

```java
@Component:使用在类上用于实例化Bean
@Controller:使用在web层上用于实例化Bean
@Service:使用在service层上用于实例化Bean
@Repositort:使用在Dao层类上用于实例化Bean
@Autowired:使用在字段上用于根据类型依赖注入
@Qualifier:结合@Autowired一起使用用于根据名称进行依赖注入
@Resource:相当于@Autowired+@Qualifier，按照名称进行注入
@Value:注入普通属性
@Scope:标注Bean的作用范围
@PostConstruct:标注方法为Bean的初始化方法
@PreDestory:标注方法为Bean的销毁方法
```

**原始注解**

**@Component|@Autowired|@Qualifier**

将上面依赖注入的实例改成原始注解开发

将xml中相应的Bean去掉，在class上加注解@Component

使用注解开发时，需要在applicationContext.xml中配置组件扫描，作用是指定哪个包的Bean需要进行扫描以扫的注解帮我创建对象，注解扫描是在Context命名空间下的

base-package写的是需要扫描的基本包，写上com.it会扫描com.it及其下面的所有包的内容有没有注解

```java
    <!-- applicationContext.xml文件配置组件扫描-->
    <context:component-scan base-package="com.it"></context:component-scan>
```

```java
package com.it.service.impl;
import com.it.dao.UserDao;
import com.it.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

//    <bean id="userService" class="com.it.service.impl.UserServiceImpl">
@Component("userService")
public class UserServiceImpl implements UserService {
//<property name="userDao" ref="userDao"></property>
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;

//    public void setUserDao(UserDao userDao) {
//        this.userDao = userDao;
//    }

    public void save() {
        userDao.save();
    }
}
```

```java
package com.it.dao.impl;

import com.it.dao.UserDao;
import org.springframework.stereotype.Component;

//    <bean id = "userDao" factory-bean="factory" factory-method="getUserDao"></bean>
@Component("userDao")
public class UserDaoImpl implements UserDao{
    @Override
    public void save() {
        System.out.println("save running");
    }
}

```

这里的@Autowired+@Qualifier就实现了依赖注入，Qualifier后面跟的内容是指定是Bean的id

使用注解可以不写set方法，但用xml配置必须用set方法

@Autowired：按照数据类型从Spring容器中进行匹配（类型注入）

@Qualifier：按照id值从容器中进行匹配，但必须和Autowired结合使用

```java
@Resource(name="userDao")//@Resource=@Autowired+@Qualifier
```

用@Controller@Service@Repositort和@Component的作用是一样的，但可读性强一点，因为全部用@Component看不出来是哪个层

**@Value**

```java
package com.it.service.impl;
import com.it.dao.UserDao;
import com.it.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

//    <bean id="userService" class="com.it.service.impl.UserServiceImpl">
@Component("userService")
public class UserServiceImpl implements UserService {
    
    //@Value("itcast")
    @Value("${jdbc.Driver}")
    private String Driver;
    
//<property name="userDao" ref="userDao"></property>
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save() {
        userDao.save();
    }
}
```

@Value("itcast")可以指定String Driver的值为itcast，因为xml可以通过加载jdbc.properties文件，所以在@Value注解里也可以添加表达式，使用properties文件的内容

**@Scope**

相当于Scope属性，可以指定单例或者多例

```java
package com.it.service.impl;
import com.it.dao.UserDao;
import com.it.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

//    <bean id="userService" class="com.it.service.impl.UserServiceImpl">
@Component("userService")
@Scope("prototype")
public class UserServiceImpl implements UserService {
    //@Value("itcast")
    @Value("${jdbc.driver}")
    private String Driver;
//<property name="userDao" ref="userDao"></property>
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save() {
        System.out.println(Driver);
        userDao.save();
    }
}
```

**@PostConstruct|@PreDestory**

相当于init-method属性和destory-method属性

```java
package com.it.dao.impl;

import com.it.dao.UserDao;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

//    <bean id = "userDao" factory-bean="factory" factory-method="getUserDao"></bean>
@Component("userDao")
public class UserDaoImpl implements UserDao{

    @PostConstruct
    public void init(){
        System.out.println("初始化方法");
    }
    @PreDestroy
    public void destory(){
        System.out.println("销毁方法");
    }
    @Override
    public void save() {
        System.out.println("save running");
    }
}
```

**Spring新注解**

```java
@Configuration:用于指定当前类是一个Spring配置类，当创建容器时会从该类上加载注释
@ComponentScan:用于指定Spring和初始化容器时要扫描的包
@Bean:使用在方法上，将该方法的返回值存储到容器中
@PropertySource:用于加载.properties配置文件
@Import：用于导入其他配置xml
```

```java
package com.it.config;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

import javax.sql.DataSource;
import javax.xml.crypto.Data;

//标志该类是Spring的核心配置类
@Configuration
//<context:component-scan base-package="com.it"></context:component-scan>
@ComponentScan("com.it")
//<context:property-placeholder location:"classpath:jdbc.properties"/>
@PropertySource("classpath:jdbc.properties")
public class SpringConfiguration {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean("dataSource")//Spring会将当前方法的返回值以指定名称(dataSource)存储到Spring容器当中
    public DataSource getDataSource() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();

        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

在xml和注解里可以使用表达式${}，但是在方法参数里不行，所以这里用了@Value作为传递

到这里xml文件就可以删除了，完全通过注解的方式替代了，核心配置换成了SpringConfiguration类

所以ApplicationContext也要改

```java
package com.it.web;

import com.it.config.SpringConfiguration;
import com.it.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserController {
    public static void main(String[] args) {
        //ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        ApplicationContext app = new AnnotationConfigApplicationContext(SpringConfiguration.class);
        UserService userService = app.getBean(UserService.class);
        userService.save();
    }
}
```

#### Spring整合Junit

Juint可以替代ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");UserService userService = app.getBean(UserService.class);这两句

让Juint负责创建Spring容器

**整合Juint步骤**

1.导入Spring集成Juint坐标	2.使用@Runwith注解替换原来的运行期	3.使用@ContextConfiguration指定配置文件	4.使用@Autowired注入需要测试的对象	5.创建测试方法测试

```java
package com.it.test;

import com.it.config.SpringConfiguration;
import com.it.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.sql.DataSource;

@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration("classpath:applicaionContext.xml")
@ContextConfiguration(classes = {SpringConfiguration.class})
public class SpringJuintTest {
    @Autowired
    private UserService userService;
    @Autowired
    private DataSource dataSource;
    @Test
    public void test1()throws Exception{
        userService.save();
        System.out.println(dataSource.getConnection());
    }
}
```

在<dependency>里导入Juint和Spring-test坐标后，用@RunWith指定SpringJuint4ClassRunner测试内核，用@ContextConfiguration指定.xml或者SpringConfiguration类作为核心配置文件，每次测试的时候用@Autowired注入Spring容器的对象，在底下创建测试方法测试即可

#### 源码解析

**IOC = 工厂模式 + 反射**

##### BeanFactory

ApplicationContext 继承自 BeanFactory，但是它不应该被理解为 BeanFactory  的实现类，而是说其内部持有一个实例化的 BeanFactory（DefaultListableBeanFactory）。以后所有的  BeanFactory 相关的操作其实是委托给这个实例来处理的。

**1.初始化BeanFactory（DefaultListableBeanFactory），因为在众多BeanFactory中功能最全**

**2.读取配置文件，解析成一个个 BeanDefinition，放到BeanFactory的beanDefinitionMap<beanName,beanDefinition>中。这里的beanName实际是beanId**

##### BeanDefinition

BeanDefinition的很多属性和方法都很熟悉，例如类名、scope、属性、构造函数参数列表、依赖的bean、是否是单例类、是否是懒加载等，其实就是将Bean的定义信息存储到这个BeanDefinition相应的属性中，后面对Bean的操作就直接对BeanDefinition进行，例如拿到这个BeanDefinition后，可以根据里面的类名、构造函数、构造函数参数，使用反射进行对象创建。

**3.调用各个 BeanFactoryPostProcessor后置处理器的postProcessBeanFactory(factory) 方法，可以管理、修改bean工厂内的beanDefinition数据**

**4.BeanFactory利用beanDefinition使用反射创建Bean实例**

**5.属性填充**

**6.如果Bean实现了BeanNameAware接口，调用setBeanName方法，传入Bean的名字**

**7.如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader方法，传入ClassLoader实例**

**8.如果实现了其他Aware接口就调用相应的方法**

**9.调用BeanPostProcessor后置处理器的postProcessBeforeInitialization方法**

**10.如果Bean实现了InitializingBean接口，调用afterPropertiesSet方法**

**11.如果Bean配置了init-method方法，调用相应的方法**

**12.调用BeanPostProcessor后置处理器的postProcessAfterInitialization方法**

**13.将Bean添加到单例池中SingletonObjects<beanName,bean>**

**14.ApplicationContext.getBean(beanName)从单例池中获取对应的Bean**

**15.销毁Bean，如果Bean实现了DisposableBean接口，调用destory方法**

**16.销毁Bean，如果Bean配置了destory-method方法，调用相应的方法**