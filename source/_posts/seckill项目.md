---
title: seckill项目
date: 2020-08-01 17:42:56
tags:
  - 项目
---

<!--more-->

## 项目创建

项目的创建通过了Spring Boot项目的快速创建方式，从Spring官网选择Starter进行创建

![1](seckill项目/1.png)

选中Web模块直接创建，查看pom文件

```xml
<!--指定项目的父pom文件为spring-boot-starter-parent-->    
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.2.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

### 引入依赖包

```xml
<!-- mybatis启动器 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
<!-- mysql 版本要与本机安装的版本一致 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.20</version>
</dependency>
<!-- 德鲁伊数据库连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.20</version>
</dependency>
```

在application.properties里配置mybatis的mapperLocation，为接下来的mybatis-generator自动生成mapper后指定mapper.xml的路径，**mybatis.mapper-locations在SpringBoot配置文件中使用，作用是扫描Mapper接口对应的XML文件，如果全程使用@Mapper注解代替Mapper.xml文件，可以不使用该配置。**

@Mapper/@MapperScan和 mapper-locations的区别见这里https://blog.csdn.net/weixin_43963583/article/details/105653333?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param

```properties
mybatis.mapper-locations=classpath:mapping/*.xml
```

## Mybatis自动生成器

### 创建数据库

#### 创建用户表

创建数据库seckill并且创建用户表user_info

![2](seckill项目/2.png)

gender的数据类型可以为int，设置为tinyint是出于性能考虑

register_mode注册方式，有三种注册方式：byphone通过手机号、bywechat通过微信、byalipay通过支付宝

third_party_id第三方登录的id，系统默认是通过手机号登录，如果通过微信或者支付宝登录则在这里记录微信支付宝的id，如果这个值为空则通过手机号telphone做一个otp校验码注册后再登录

#### 创建用户密码表

user的password是加密的字符串，一般不跟主表放在一起，企业开发里放在另一台系统上进行，这里为了简单起见不去模拟几台机器的操作，而是至少在数据库层面密码和用户主表是分开存储的

![3](seckill项目/3.png)

encrpt_passoword，密码会经过encrpt的加密方式，以密文方式存在数据库中

user_id是一个外键，用于关联到用户主表

### 导入Mybatis自动生成插件

在pom.xml中导入mybatis-generator-maven-plugin插件

```xml
<!--自动生成文件插件,mybatis generator-->
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.5</version>

    <!--插件的依赖-->
    <dependencies>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.5</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
    </dependencies>

    <executions>
        <execution>
            <id>mybatis generator</id>
            <phase>package</phase>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>

    <configuration>
        <!--允许移动生成的文件-->
        <verbose>true</verbose>
        <!--允许自动覆盖文件，这里设置为false，重新运行generator之后不会覆盖之前的Mapper等文件-->
        <overwrite>true</overwrite>
        <!--配置文件路径-->
        <configurationFile>
            src/main/resources/mybatis-generator.xml
        </configurationFile>
    </configuration>
</plugin>
```

### Mybatis自动生成器配置文件

mybatis-generator.xml可以从官网上下载，网上其他地方也有很多，这里附上[官网下载地址](http://mybatis.org/generator/configreference/xmlconfig.html)

mybatis-generator.xml的内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!--数据库连接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/seckill"
                        userId="root"
                        password="root">
        </jdbcConnection>


        <!--生成Model/DataObject类存放的位置，数据库到Java类-->
        <javaModelGenerator targetPackage="com.seckillproject.dataobject" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!--生成映射文件存放的位置-->
        <sqlMapGenerator targetPackage="mapping"  targetProject="src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--生成Dao类存放的位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.seckillproject.dao"  targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!--生成对应表及类名-->
        <table tableName="user_info" domainObjectName="UserDO"></table>
        <table tableName="user_password" domainObjectName="UserPasswordDO"></table>
    </context>
</generatorConfiguration>
```

修改其中的配置，在Run-Edit Configurations里新建maven命令，在Command line里添加

```
mybatis-generator:generate
```

上面这行命令就会解析项目文件里指定的mybatis-generator插件，并且执行它

![5](seckill项目/5.png)

添加命令后Run项目会生成一些文件，文件目录的配置都在mybatis-generator.xml中

![6](seckill项目/6.png)

#### UserDoMapper.xml

打开UserDoMapper.xml

```xml
<sql id="Example_Where_Clause">
<!--
  WARNING - @mbg.generated
  This element is automatically generated by MyBatis Generator, do not modify.
  This element was generated on Sat Aug 01 19:56:02 CST 2020.
-->
<where>
  <foreach collection="oredCriteria" item="criteria" separator="or">
    <if test="criteria.valid">
      <trim prefix="(" prefixOverrides="and" suffix=")">
        <foreach collection="criteria.criteria" item="criterion">
          <choose>
            <when test="criterion.noValue">
              and ${criterion.condition}
            </when>
            <when test="criterion.singleValue">
              and ${criterion.condition} #{criterion.value}
            </when>
            <when test="criterion.betweenValue">
              and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
            </when>
            <when test="criterion.listValue">
              and ${criterion.condition}
              <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                #{listItem}
              </foreach>
            </when>
          </choose>
        </foreach>
      </trim>
    </if>
  </foreach>
</where>
</sql>
```

可以看到里面有自动生成的复杂查询语句，通过在mybatis-generator.xml中配置可以避免，只生成简单查询语句

```xml
<!--生成对应表及类名-->
<table tableName="user_info" domainObjectName="UserDO" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
<table tableName="user_password" domainObjectName="UserPasswordDO" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
```

#### UserDOExample

```java
public class UserDOExample {
    /**
     * This field was generated by MyBatis Generator.
     * This field corresponds to the database table user_info
     *
     * @mbg.generated Sat Aug 01 19:56:02 CST 2020
     */
    protected String orderByClause;

    /**
     * This field was generated by MyBatis Generator.
     * This field corresponds to the database table user_info
     *
     * @mbg.generated Sat Aug 01 19:56:02 CST 2020
     */
    protected boolean distinct;

    /**
     * This field was generated by MyBatis Generator.
     * This field corresponds to the database table user_info
     *
     * @mbg.generated Sat Aug 01 19:56:02 CST 2020
     */
    protected List<Criteria> oredCriteria;
```

UserDOExample是自动生成器帮我们自动生成的用于复杂查询的类，复杂查询一般自己编写，不会使用它生成的这个，所以直接删除它，删除后的结构如下

![5](线程池原理/5.png)

#### 遇到的问题

运行的时候报错

The server time zone value '?й???????' is unrecognized or represents more than one time zone

原因是数据库的时区不正确

**解决方案：**

1.在项目代码-数据库连接URL后，加上 ?serverTimezone=UTC

2.在mysql中设置时区，默认为SYSTEM
set global time_zone=’+8:00’

```sql
mysql> set global time_zone='+8:00';
Query OK, 0 rows affected (0.01 sec)
```

## 接入数据源

Mybatis是ORM层框架，对应MySQL数据库要有一个数据源

### 数据源作用

数据源建立多个数据库连接，这些数据库连接会保存在数据库连接池中

当需要访问数据库时，只需要从数据库连接池中，获取空闲的数据库连接

当程序访问数据库结束时，数据库连接会放回数据库连接池中

### 数据源配置

在application.properties中配置

```properties
spring.datasource.name=seckill
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/seckill
spring.datasource.username=root
spring.datasource.password=root

#使用druid数据源
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

### 主配置类的更改

主配置类指定@SpringBootApplication(scanBasePackages)和@MapperScan，这里配置的scanBasePackages其实还没有用到，主要是和@Controller、@Service、@Component、@Repository配合用

```java
package com.seckillproject.seckill;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication(scanBasePackages = {"com.seckillproject"})
@MapperScan("com.seckillproject.dao")
public class SeckillApplication {


    public static void main(String[] args) {
        SpringApplication.run(SeckillApplication.class, args);
    }

}
```

#### @SpringBootApplication(scanBasePackages)

@SpringBootApplication是一个复合注解，包含了@SpringBootConfiguration，@EnableAutoConfiguration和@ComponentScan三个注解，scanBasePackages是@ComponentScan的属性，作用是扫描指定包下所有的组件，放入容器

#### @MapperScan和@Mapper

@Mapper注解：
	作用：在接口类上添加了@Mapper，在编译之后会生成相应的接口实现类

​	添加位置：Dao层的接口类上面

@MapperScan
	 作用：指定要变成实现类的接口所在的包，然后包下面的所有接口在编译之后都会生成相应的实现类

 	添加位置：是在Springboot启动类上面添加

mybatis支持的映射方式有基于xml的mapper.xml文件，和基于注解的@Insert、@Update、@Delete

两者都需要使用@Mapper或者@MapperScan

## 完整SpringMVC 查询用户信息

### Service层

service层下有UserService接口

```java
package com.seckillproject.service;

public interface UserService {
    //通过用户ID获取用户对象的方法
    void getUserById(Integer id);
}
```

还有Implement包，impl包下有UserServiceImpl实现类

```java
package com.seckillproject.service.impl;

import com.seckillproject.dao.UserDOMapper;
import com.seckillproject.dataobject.UserDO;
import com.seckillproject.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDOMapper userDOMapper;
    @Override
    public void getUserById(Integer id) {
        UserDO userDO = userDOMapper.selectByPrimaryKey(id);
    }
}

```

#### model层

##### UserModel

因为企业级开发里是不能从Service层里传出UserDo对象的，所以还需要在Service里加一个model包，用来定义业务逻辑交互，实现UserModel

```java
package com.seckillproject.service.model;

public class UserModel {
    private Integer id;
    private String name;
    private Byte gender;
    private String age;
    private String telphone;
    private String registerMode;
    private String thirdPartyId;

    private String encrptPassword;

    public String getEncrptPassword() {
        return encrptPassword;
    }

    public void setEncrptPassword(String encrptPassword) {
        this.encrptPassword = encrptPassword;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Byte getGender() {
        return gender;
    }

    public void setGender(Byte gender) {
        this.gender = gender;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getTelphone() {
        return telphone;
    }

    public void setTelphone(String telphone) {
        this.telphone = telphone;
    }

    public String getRegisterMode() {
        return registerMode;
    }

    public void setRegisterMode(String registerMode) {
        this.registerMode = registerMode;
    }

    public String getThirdPartyId() {
        return thirdPartyId;
    }

    public void setThirdPartyId(String thirdPartyId) {
        this.thirdPartyId = thirdPartyId;
    }
}

```

encrptPassword是属于用户对象的，仅仅是因为数据模型层的关系，把它和用户主表设计成两个表，但是对对象来说就是属于UserModel的

UserModel才是真正意义上处理业务逻辑的模型，DataObject仅仅是一个数据库的映射

##### convertFromDataObject

在UserServiceImpl中增加convertFromDataObject，用来使用UserDO和UserPasswordDO去构造返回的UserModel

```java
private UserModel convertFromDataObject(UserDO userDO, UserPasswordDO userPasswordDO) {
    if (userDO == null) {
        return null;
    }
    UserModel userModel = new UserModel();
    //copyProperties要求两个对象的要赋值的字段，类型和名称都要相同
    BeanUtils.copyProperties(userDO,userModel);

    //不能用copyProperties去设置password，因为有一个id字段是重复的
    if(userPasswordDO != null) {
        userModel.setEncrptPassword(userPasswordDO.getEncrptPassword());
    }
    return userModel;
}
```

##### getUserById

要实现getUserById方法，需要引入userPasswordDOMapper

```java
@Autowired
private UserPasswordDOMapper userPasswordDOMapper;
```

但是自动生成器仅仅帮我们实现了selectByPrimaryKey，我们需要根据用户id查找他的password

![7](seckill项目/7.png)

所以在UserPasswordDOMapper.xml下增加SQL语句

```xml
<select id="selectByUserId" parameterType="java.lang.Integer" resultMap="BaseResultMap">
select
<include refid="Base_Column_List" />
from user_password
where user_id = #{userId,jdbcType=INTEGER}
</select>
```

在UserPasswordDOMapper接口中增加实现

```java
UserPasswordDO selectByUserId(Integer id);
```

最后完成UserServiceImpl的getUserById

```java
public UserModel getUserById(Integer id) {
    //调用userDOMapper获取到对应用户的dataobject
    UserDO userDO = userDOMapper.selectByPrimaryKey(id);

    if(userDO == null) {
        return null;
    }
    //通过用户Id获取对应的用户加密密码信息
    UserPasswordDO userPasswordDO = userPasswordDOMapper.selectByUserId(userDO.getId());
    return convertFromDataObject(userDO,userPasswordDO);
}
```

### Controller层

创建Controller包下的UserController

```java
package com.seckillproject.controller;

import com.seckillproject.service.UserService;
import com.seckillproject.service.model.UserModel;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller("user")
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/get")
    @ResponseBody
    public UserModel getUser(@RequestParam(name="id") Integer id) {
        //调用service服务器获取对应id的用户对象并返回给前端
        UserModel userModel = userService.getUserById(id);
        return userModel;
    }
}

```

#### 测试结果

![8](seckill项目/8.png)

结果发现用户经过加密的密码也会展示出来，问题出在Service层返回了完整的UserModel，Controller层返回了反正的UserModel，只需要让Controller返回前端需要展示的数据即可

因此在Controller里加一层ViewObject

#### ViewObject层

ViewObject里创建UserVO

```java
package com.seckillproject.controller.viewobject;

public class UserVO {
    private Integer id;
    private String name;
    private Byte gender;
    private String age;
    private String telphone;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Byte getGender() {
        return gender;
    }

    public void setGender(Byte gender) {
        this.gender = gender;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getTelphone() {
        return telphone;
    }

    public void setTelphone(String telphone) {
        this.telphone = telphone;
    }
}
```

UserVO里封装只想让前端展示的数据

和UserModel一样将UserController增加convertFromUserModel方法

```java
private UserVO convertFromUserModel(UserModel userModel) {
    if(userModel == null) {
        return null;
    }
    UserVO userVO = new UserVO();
    BeanUtils.copyProperties(userModel,userVO);
    return userVO;
}
```

将getUser返回UserModel改为返回UserVO

```java
@RequestMapping("/get")
@ResponseBody
public UserVO getUser(@RequestParam(name="id") Integer id) {
    //调用service服务器获取对应id的用户对象并返回给前端
    UserModel userModel = userService.getUserById(id);

    //将核心领域模型用户对象转化为可提供UI使用的viewObject
    return convertFromUserModel(userModel);
}
```

经过再次测试结果如下，没有返回密码等不应该返回的数据

![9](seckill项目/9.png)

### 总结

至此做了完整的读操作展示，从Controller层到Service层到DataObject层，DataObject层负责数据存储和到Service的传输，在Service层组装了需要的一个核心领域模型做下一步的处理，Controller层做了ViewObject，保证了UI只使用到它需要展示的字段

