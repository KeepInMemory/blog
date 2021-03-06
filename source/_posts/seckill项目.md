---
title: seckill项目
date: 2020-08-01 17:42:56
tags:
  - 项目
---

# 基础能力建设

## 项目创建

项目的创建通过了Spring Boot项目的快速创建方式，从Spring官网选择Starter进行创建

![1](seckill项目/1.png)

<!--more-->

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

### service层

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

##### UserModel类

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

### controller层

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

#### viewobject层

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

## 定义通用的返回对象

项目中如果出现了异常，异常会抛到Tomcat容器层中，Tomcat帮我们处理，前端会出现自带的Error Page，我们需要将成功和异常统一起来

### response层

创建response包，用来处理HTTP返回

新建CommonReturnType类

```java
package com.seckillproject.response;

public class CommonReturnType {

    //表明对应请求的返回处理"success"或者"fail"
    private String status;
    //若status="success",data返回前端需要数据
    //若Status="fail",data使用通用的错误码格式
    private Object data;

    //定义一个通用的创建方法
    public static CommonReturnType create(Object result) {
        return CommonReturnType.create(result,"success");
    }
    public static CommonReturnType create(Object result,String status) {
        CommonReturnType type = new CommonReturnType();
        type.setData(result);
        type.setStatus(status);
        return type;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}
```

UserController的getUser方法也要更改，返回通用对象

```java
    @RequestMapping("/get")
    @ResponseBody
    public CommonReturnType getUser(@RequestParam(name="id") Integer id) {
        //调用service服务器获取对应id的用户对象并返回给前端
        UserModel userModel = userService.getUserById(id);

        //将核心领域模型用户对象转化为可提供UI使用的viewObject
        UserVO userVO =  convertFromUserModel(userModel);
        //返回通用对象
        return CommonReturnType.create(userVO);
    }
```

测试发现返回了通用对象

![10](seckill项目/10.png)

### error层

#### CommonError接口

项目下创建error层，创建CommonError接口

```java
package com.seckillproject.error;

public interface CommonError {
    public int getErrCode();
    public String getErrMsg();

    public CommonError setErrMsg(String errMsg);
}
```

#### EmBusinessError类

创建枚举类型的EmBusinessError去实现CommonError接口

定义一个简单的错误码USER_NOT_EXIST，值为10001。一般企业开发需要一个统一的全局错误码，所以让10000开头为用户信息相关错误定义。

定义了通用错误类型，比如值为00001，含义参数不合法。这样可以省去很多地方都定义错误类型，比如传入方法里邮箱不合法、手机号不合法等等，都使用通用错误类型。它们的errCode虽然是一样的，但是errMsg却根据场景不同要有变化，所以接口中定义的public CommonError setErrMsg(String errMsg);就是为了改变不同场景下的errMsg

```java
package com.seckillproject.error;

public enum EmBusinessError implements CommonError{
    //通用错误类型00001
    PARAMETER_VALIDATION_ERROR(00001,"参数不合法"),

    //10000开头为用户信息相关错误定义
    USER_NOT_EXIST(10001,"用户不存在")
    ;
    private int errCode;
    private String errMsg;

    private EmBusinessError(int errCode,String errMsg) {
        this.errCode = errCode;
        this.errMsg = errMsg;
    }
    @Override
    public int getErrCode() {
        return this.errCode;
    }

    @Override
    public String getErrMsg() {
        return this.errMsg;
    }

    @Override
    public CommonError setErrMsg(String errMsg) {
        this.errMsg = errMsg;
        return this;
    }
}
```

#### BusinessException类

一般程序出了异常，统一抛出Exception，直到Controller层的SpringBootHandler捕获处理

创建BusinessException类

```java
package com.seckillproject.error;

//包装器业务异常类实现
public class BusinessException extends Exception implements CommonError{
    private CommonError commonError;

    //直接接收EmBusinessError的传参用于构造业务异常
    public BusinessException(CommonError commonError) {
        super();
        this.commonError = commonError;
    }

    //接收自定义errMsg的方式构造业务异常
    public BusinessException(CommonError commonError,String errMsg) {
        super();
        this.commonError = commonError;
        this.commonError.setErrMsg(errMsg);
    }

    @Override
    public int getErrCode() {
        return this.commonError.getErrCode();
    }

    @Override
    public String getErrMsg() {
        return this.commonError.getErrMsg();
    }

    @Override
    public CommonError setErrMsg(String errMsg) {
        this.commonError.setErrMsg(errMsg);
        return this;
    }
}
```

BusinessException和EmBusinessError枚举类都实现了CommonError接口，以至于外部可以通过new EmBusinessError和new BusinessException都可以由errCode和errMsg的组装定义，这两个类都要实现setErrMsg方法，将原本的errMsg给覆盖掉

### controller层

#### exceptionHandler

对于Web系统来说e，Controller层是业务处理的最后一道关口，如果Controller层不处理异常，抛给Tomcat，就会出现自带的Error Page页，所以需要我们在Controller统一处理

在Controller层写handlerException方法

```java
//定义exceptionhandler解决未被controller层吸收的exception
@ExceptionHandler(Exception.class)
@ResponseStatus(HttpStatus.OK)
@ResponseBody//不加ResponseBody就会返回链接地址，而不是对象
public Object handlerException(HttpServletRequest httpServletRequest,Exception exception) {
    CommonReturnType commonReturnType = new CommonReturnType();
    commonReturnType.setStatus("fail");
    commonReturnType.setData(exception);
    return commonReturnType;
}
```

使用@ExceptionHandler指明处理的是哪个异常，@ResponseStatus指明方法返回的是什么HttpStatus，返回OK不会出现自带错误页面

运行项目，输入http://localhost:8080/user/get?id=2，返回了通用返回对象

![11](seckill项目/11.png)

但是这些返回的Json里面还有我们不需要的东西，对handlerException的代码进一步改造

```java
@ExceptionHandler(Exception.class)
@ResponseStatus(HttpStatus.OK)
@ResponseBody
public Object handlerException(HttpServletRequest httpServletRequest,Exception exception) {
    BusinessException businessException = (BusinessException)exception;

    Map<String,Object> responseData = new HashMap<>();
    responseData.put("errCode",businessException.getErrCode());
    responseData.put("errMsg",businessException.getErrMsg());
    return CommonReturnType.create(responseData,"fail");
}
```

再次运行测试，输入http://localhost:8080/user/get?id=2

![12](seckill项目/12.png)

可以看到只返回了我们需要的字段，即status为fail的时候，返回的data为errCode+errMsg的类型

如果传过来的exception不是BusinessException的类型，需要定义一个新的类型错误，未知错误

```java
//通用错误类型10001
PARAMETER_VALIDATION_ERROR(10001,"参数不合法"),
UNKNOWN_ERROR(10002,"未知错误"),
//20000开头为用户信息相关错误定义
USER_NOT_EXIST(20001,"用户不存在")
;
```

这里本来是下面这样的

```java
//通用错误类型00001
PARAMETER_VALIDATION_ERROR(00001,"参数不合法"),
UNKNOWN_ERROR(00002,"未知错误"),
//10000开头为用户信息相关错误定义
USER_NOT_EXIST(10001,"用户不存在")
;
```

但是int 类型的00002传到前端转成Json格式就变成了2

#### BaseController

这里Controller层的代码如下

```java
package com.seckillproject.controller;

import com.seckillproject.controller.viewobject.UserVO;
import com.seckillproject.error.BusinessException;
import com.seckillproject.error.EmBusinessError;
import com.seckillproject.response.CommonReturnType;
import com.seckillproject.service.UserService;
import com.seckillproject.service.model.UserModel;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;
import java.util.Map;

@Controller("user")
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/get")
    @ResponseBody
    public CommonReturnType getUser(@RequestParam(name="id") Integer id) throws BusinessException {
        //调用service服务器获取对应id的用户对象并返回给前端
        UserModel userModel = userService.getUserById(id);
        //若获取的用户信息不存在
        if(userModel == null){
            throw new BusinessException(EmBusinessError.USER_NOT_EXIST);
        }

        //将核心领域模型用户对象转化为可提供UI使用的viewObject
        UserVO userVO =  convertFromUserModel(userModel);
        //返回通用对象
        return CommonReturnType.create(userVO);
    }

    private UserVO convertFromUserModel(UserModel userModel) {
        if(userModel == null) {
            return null;
        }
        UserVO userVO = new UserVO();
        BeanUtils.copyProperties(userModel,userVO);
        return userVO;
    }

    //定义exceptionhandler解决未被controller层吸收的exception
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Object handlerException(HttpServletRequest httpServletRequest,Exception exception) {
        Map<String,Object> responseData = new HashMap<>();
        if(exception instanceof BusinessException) {
            BusinessException businessException = (BusinessException)exception;
            responseData.put("errCode",businessException.getErrCode());
            responseData.put("errMsg",businessException.getErrMsg());
        } else {
            responseData.put("errCode",EmBusinessError.UNKNOWN_ERROR.getErrCode());
            responseData.put("errMsg",EmBusinessError.UNKNOWN_ERROR.getErrMsg());
        }
        return CommonReturnType.create(responseData,"fail");
    }
}
```

在这里异常处理类定义在了UserController里，但其实handlerException是所有controller公用的逻辑

所以抽象一层BaseController层，将异常处理放在基类里

```java
package com.seckillproject.controller;

import com.seckillproject.error.BusinessException;
import com.seckillproject.error.EmBusinessError;
import com.seckillproject.response.CommonReturnType;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;
import java.util.Map;

public class BaseController {

    //定义exceptionhandler解决未被controller层吸收的exception
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Object handlerException(HttpServletRequest httpServletRequest, Exception exception) {
        Map<String,Object> responseData = new HashMap<>();
        if(exception instanceof BusinessException) {
            BusinessException businessException = (BusinessException)exception;
            responseData.put("errCode",businessException.getErrCode());
            responseData.put("errMsg",businessException.getErrMsg());
        } else {
            responseData.put("errCode", EmBusinessError.UNKNOWN_ERROR.getErrCode());
            responseData.put("errMsg",EmBusinessError.UNKNOWN_ERROR.getErrMsg());
        }
        return CommonReturnType.create(responseData,"fail");
    }
}
```

同时让UserController去继承BaseController

这样UserController抛出的异常就能由BaseController内的handlerException处理，因为handlerException方法也得到了继承

运行测试，http://localhost:8080/user/get?id=2

![13](seckill项目/13.png)

可以看到异常被正确的处理了，注意这里errCode已经改成了最新的

### 总结

定义了CommonReturnType，能够用对应的String status和Object data返回所有的Json序列化的一个固定对象，供前端解析使用，摒弃了HttpStatusCode+内嵌Tomcat自带的ErrorPage页的方式处理异常

定义了一个Common的BussinessError的方式，去统一管理我们所需要的错误码

在BaseController基类里，定义了一个通用的exceptionhandler解决未被controller层吸收的exception。并且使用errorCode+errorMsg的统一定义方式吃掉了所有内部未知的异常

## 基础能力总结

到目前位置完成了SpringBoot+Mybatis+SpringMVC框架+常态的错误定义等基础能力

# 模型能力管理

模型管理就是去完成领域模型，会有用户模型商品模型下单模型

## 用户模型管理

### otp短信获取

#### 用户获取otp短信接口

在UserController中加入getOtp方法作为用户获取otp短信的接口

```java
@Autowired
private HttpServletRequest httpServletRequest;
//用户获取otp短信接口
@RequestMapping("/getotp")
@ResponseBody
public CommonReturnType getOtp(@RequestParam(name="telphone") String telphone) {
    //需要按照一定规则生成OTP验证码
    Random random = new Random();
    int randomInt = random.nextInt(99999);
    randomInt += 10000;
    String otpCode = String.valueOf(randomInt);

    //将OTP验证码同对应用户的手机号关联，使用httpsession的方式绑定他的手机号与OTPCODE
    httpServletRequest.getSession().setAttribute(telphone,otpCode);

    //将OTP验证码通过短信通道发送给用户，这里省略
    System.out.println("telphone=" + telphone + "otpCode=" + otpCode);
    return CommonReturnType.create(null);
}
```

将OTP验证码同对应用户手机号关联最好使用key-value对的形式，企业采用Redis的去存储，因为Redis天然支持key-value对的形式，可以设置生存时间，也可以反复点击覆盖，但是这里没讲到Redis，先用httpsession绑定手机号和otpcode

HttpServletRequest是通过容器注入的，HttpServletRequest是一个单例的模式，怎么能支持多个用户的并发访问？

通过Bean包装的HttpServletRequest内部拥有ThreadLocalMap，能让不同用户在自己的线程中处理自己的request，并且有ThreadLocal清除机制

#### otp注册用户

接下来做用户注册的界面，下载的Static静态资源包里的内容是基于BootStrap的Metronic框架模板

![14](seckill项目/14.png)

##### getotp.html

新建一个getotp.html作为获取otp短信界面

```html
<html>
<head>
<meta charset="UTF-8">
<script src = "static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>
</head>

<body>
	<div>
		<h3>获取otp信息</h3>
		
		<div>
			<label>手机号</label>
			<div>
				<input type="text" placeholder="手机号" name="telphone" id="telphone"/>
			</div>
		</div>
		
		<div>
			<button id= "getotp" type="submit">
			获取otp短信
			</button>
		</div>
	</div>
</body>


<script>
    jQuery(document).ready(function () {
        $("#getotp").on("click",function () {
            var telphone = $("#telphone").val();
            if (telphone == null || telphone === "") {
                alert("手机号不能为空");
                return false;
            }
            $.ajax({
                type:"POST",
                contentType:"application/x-www-form-urlencoded",
                url:"http://localhost:8080/user/getotp",
                data:{
                    "telphone":telphone,
                },
                success:function (data) {
                    if (data.status === "success") {
                        alert("短信发送成功，请查收");
                    }else {
                        alert("短信发送失败，原因是"+data.data.errMsg);
                    }
                },
                error:function (data) {
                    alert("短信发送失败"+data.responseText);
                }
            })
        });
    });

</script>
</html>
```

这里ajax的type、contentType都对应下面的设置

在UserController的getOtp上增加method = {RequestMethod.POST}，让getotp方法必须映射到http的POST请求才处理，comsumes对应后端需要消费的Content Type的名字，指明为传统的HTTP的url form encoded方式"application/x-www-form-urlencoded"

```java
//用户获取otp短信接口
@RequestMapping(value = "/getotp",method = {RequestMethod.POST},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
public CommonReturnType getOtp(@RequestParam(name="telphone") String telphone) {
    //需要按照一定规则生成OTP验证码
    Random random = new Random();
    int randomInt = random.nextInt(99999);
    randomInt += 10000;
    String otpCode = String.valueOf(randomInt);

    //将OTP验证码同对应用户的手机号关联，使用httpsession的方式绑定他的手机号与OTPCODE
    httpServletRequest.getSession().setAttribute(telphone,otpCode);

    //将OTP验证码通过短信通道发送给用户，这里省略
    System.out.println("telphone=" + telphone + "otpCode=" + otpCode);
    return CommonReturnType.create(null);
}
```

这里将它抽象出来在baseController中定义CONTENT_TYPE_FORMED

```java
public class BaseController {
    public static final String CONTENT_TYPE_FORMED = "application/x-www-form-urlencoded";
```

##### 遇到的问题

启动项目测试，输入手机号发现报错Access to XMLHttpRequest at 'http://localhost:8080/user/getotp' from origin 'null' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

![15](seckill项目/15.png)

这是一个跨域请求错误，ajax启动于getotp.html，对应的域是本地文件。请求的服务器是localhost域名，虽然请求能够到达服务端并返回，但是ajax请求回调认定这种跨域回调是不安全的，因此会报错，并且走不到ajax的success block里面

**解决方法**：在Spring Boot里有专门解决办法，那就是让reponse时刻返回Access-Control-Allow-Origin这个header为所有的域，也就是*号即可

Spring Boot有专门的注解：**@CrossOrigin**

```java
//用户获取otp短信接口
@RequestMapping(value = "/getotp",method = {RequestMethod.POST},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
@CrossOrigin
public CommonReturnType getOtp(@RequestParam(name="telphone") String telphone) {
    //需要按照一定规则生成OTP验证码
    Random random = new Random();
    int randomInt = random.nextInt(99999);
    randomInt += 10000;
    String otpCode = String.valueOf(randomInt);

    //将OTP验证码同对应用户的手机号关联，使用httpsession的方式绑定他的手机号与OTPCODE
    httpServletRequest.getSession().setAttribute(telphone,otpCode);

    //将OTP验证码通过短信通道发送给用户，这里省略
    System.out.println("telphone=" + telphone + "otpCode=" + otpCode);
    return CommonReturnType.create(null);
}
```

通过这个注解，可以完成Spring Boot对应返回对应Web请求当中加上跨域Allow Header

再次测试，可以发现，返回的Response Header里Access-Control-Allow-Origin=*，代表允许所有的域

![16](seckill项目/16.png)

##### getotp界面美化

添加BootStrap的样式

```html
<html>
<head>
<meta charset="UTF-8">

<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css">
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css">
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css">
<script src = "static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>
</head>

<body class="login">
	<div class="content">
		<h3 class=>获取otp信息</h3>
		
		<div class="form-group">
			<label class="control-label">手机号</label>
			<div>
				<input class="form-control" type="text" placeholder="手机号" name="telphone" id="telphone"/>
			</div>
		</div>
		
		<div class="from-actions">
			<button class="btn blue" id= "getotp" type="submit">
			获取otp短信
			</button>
		</div>
	</div>
</body>


<script>
    jQuery(document).ready(function () {
        $("#getotp").on("click",function () {
            var telphone = $("#telphone").val();
            if (telphone == null || telphone === "") {
                alert("手机号不能为空");
                return false;
            }
            $.ajax({
                type:"POST",
                contentType:"application/x-www-form-urlencoded",
                url:"http://localhost:8080/user/getotp",
                data:{
                    "telphone":telphone,
                },
                success:function (data) {
                    if (data.status === "success") {
                        alert("短信发送成功，请查收");
                    }else {
                        alert("短信发送失败，原因是"+data.data.errMsg);
                    }
                },
                error:function (data) {
                    alert("短信发送失败"+data.responseText);
                }
            })
        });
    });

</script>
</html>
```

##### 用户注册接口

下面实现一个方法用于用户注册

在UserController中实现用户注册接口register

```java
//用户注册接口
public CommonReturnType register(@RequestParam(name="telphone") String telphone,
@RequestParam(name="otpCode") String otpCode, @RequestParam(name="name") String name,
@RequestParam(name="gender") Integer gender,@RequestParam(name="age") Integer age) throws BusinessException {
    //验证手机号和otpCode相符合
    String inSessionOtpCode = (String) this.httpServletRequest.getSession().getAttribute(telphone);
    if(StringUtils.equals(otpCode,inSessionOtpCode)) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"短信验证码不符合");
    }
    //用户注册流程

}
```

接口需要先验证手机号和otpCode相符合后，再执行用户注册流程

StringUtils是Apache-lang包下的一个类，里面的equals方法可以比较两个字符串是否相等

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.7</version>
</dependency>
```

StringUtils.equals源码如下，方法考虑了各种情况

```java
public static boolean equals(CharSequence cs1, CharSequence cs2) {
    if (cs1 == cs2) {
        return true;
    } else if (cs1 != null && cs2 != null) {
        if (cs1.length() != cs2.length()) {
            return false;
        } else {
            return cs1 instanceof String && cs2 instanceof String ? cs1.equals(cs2) : CharSequenceUtils.regionMatches(cs1, false, 0, cs2, 0, cs1.length());
        }
    } else {
        return false;
    }
}
```

为了完成UserController.register的用户的注册流程，需要在UserService接口里增加方法用来注册

```java
void register(UserModel userModel) throws BusinessException;
```

它的实现方法UserServiceImpl的register

```java
@Override
@Transactional
public void register(UserModel userModel) throws BusinessException{
    if(userModel == null) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    if(!StringUtils.isEmpty(userModel.getName()) || userModel.getGender() == null
    || userModel.getAge() == null || StringUtils.isEmpty(userModel.getTelphone())) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    //实现model转dataobject方法
    UserDO userDO = convertFromModel(userModel);
    userDOMapper.insertSelective(userDO);

    UserPasswordDO userPasswordDO = convertPasswordFromModel(userModel);
    userPasswordDOMapper.insertSelective(userPasswordDO);
    return;

}
```

在Controller层构造UserModel传入Service层，Service层将UserModel构造成UserDO和UserPasswordDO，即dataobject插入数据库

加上@Transactional事务标签，防止userDO加进去了，userPasswordDO没加进去

这里为什么用userDOMapper.insertSelective(userDO);而不用insert方法呢，去UserPasswordDOMapper.xml中查看sql语句可以看到，insertSelective会判断各字段是否是空，如果是空就不插入这个字段，用数据库默认值代替，如果不为空就插入。insert的语句则不管是不是空都直接覆盖进去，也就无法依赖数据库的默认值。

```xml
<insert id="insertSelective" parameterType="com.seckillproject.dataobject.UserPasswordDO">
<!--
  WARNING - @mbg.generated
  This element is automatically generated by MyBatis Generator, do not modify.
  This element was generated on Sun Aug 02 14:43:50 CST 2020.
-->
insert into user_password
<trim prefix="(" suffix=")" suffixOverrides=",">
  <if test="id != null">
    id,
  </if>
  <if test="encrptPassword != null">
    encrpt_password,
  </if>
  <if test="userId != null">
    user_id,
  </if>
</trim>
<trim prefix="values (" suffix=")" suffixOverrides=",">
  <if test="id != null">
    #{id,jdbcType=INTEGER},
  </if>
  <if test="encrptPassword != null">
    #{encrptPassword,jdbcType=VARCHAR},
  </if>
  <if test="userId != null">
    #{userId,jdbcType=INTEGER},
  </if>
</trim>
</insert>
```

数据库尽量使用not null，设定默认值的好处：1.可以避免出现null，使得代码更少的处理空指针；2.null只对后端有效，对前端则应该是空的字符串

但也会有必须用null的情况，比如约定telphone是唯一字段，对telphone字段加unique_index唯一索引，但是如果加了这个索引，而用户是第三方登录进来的，他有third_party_id，但是没有telphone因为他不是手机号登录的，除非我们强绑定， 让他绑定手机号才能第三方登录。他没有手机号就默认使用""空字符串，而唯一索引需要每个不为null的都不相同，所以这里只能指定telphone使用null，使第三方登录，手机号为null的不受唯一索引约束

两个convert代码如下

```java
private UserPasswordDO convertPasswordFromModel(UserModel userModel) {
    if(userModel == null) {
        return null;
    }

    UserPasswordDO userPasswordDO = new UserPasswordDO();
    userPasswordDO.setEncrptPassword(userModel.getEncrptPassword());
    userPasswordDO.setUserId(userModel.getId());
    return userPasswordDO;
}
private UserDO convertFromModel(UserModel userModel) {
    if(userModel == null) {
        return null;
    }

    UserDO userDO = new UserDO();
    BeanUtils.copyProperties(userModel,userDO);
    return userDO;
}
```

最后完善UserController的register

```java
//用户注册接口
@RequestMapping(value = "/register",method = {RequestMethod.POST},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
public CommonReturnType register(@RequestParam(name="telphone") String telphone,
@RequestParam(name="otpCode") String otpCode, @RequestParam(name="name") String name,
@RequestParam(name="gender") Integer gender,@RequestParam(name="age") Integer age,
@RequestParam(name="password") String password) throws BusinessException {
    //验证手机号和otpCode相符合
    String inSessionOtpCode = (String) this.httpServletRequest.getSession().getAttribute(telphone);
    if(StringUtils.equals(otpCode,inSessionOtpCode)) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"短信验证码不符合");
    }
    //用户注册流程
    UserModel userModel = new UserModel();
    userModel.setName(name);
    userModel.setGender(gender);
    userModel.setAge(age);
    userModel.setTelphone(telphone);
    userModel.setEncrptPassword(MD5Encoder.encode(password.getBytes()));
    userModel.setRegisterMode("byphone");

    userService.register(userModel);
    return CommonReturnType.create(null);
}
```

加上了RequestMapping、ResponseBody注解

使用MD5Encoder类对password加密

调用UserService的register

##### register.html

首先在getotp.html中加入页面跳转到register.html

```javascript
success:function (data) {
    if (data.status === "success") {
        alert("短信发送成功，请查收");
        window.location.href = "register.html";
    }else {
        alert("短信发送失败，原因是"+data.data.errMsg);
    }
},
error:function (data) {
    alert("短信发送失败"+data.responseText);
}
```

新建register.html

```html
<html>
<head>
<meta charset="utf-8">
<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css"/>
<script src="static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>

</head>

<body class="login">
  <div class="content">
    <h3 class="form-title">用户注册</h3>
    <div class="form-group">
      <label class="control-label">手机号</label>
      <div>
        <input class="form-control" type="text" placeholder="手机号" name="telphone" id="telphone"/>
      </div>
    </div>
     <div class="form-group">
      <label class="control-label">验证码</label>
      <div>
        <input class="form-control" type="text" placeholder="验证码" name="otpCode" id="otpCode"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">用户昵称</label>
      <div>
        <input class="form-control" type="text" placeholder="用户昵称" name="name" id="name"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">性别</label>
      <div>
        <input class="form-control" type="text" placeholder="性别" name="gender" id="gender"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">年龄</label>
      <div>
        <input class="form-control" type="text" placeholder="年龄" name="age" id="age"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">密码</label>
      <div>
        <input class="form-control" type="password" placeholder="密码" name="password" id="password"/>
      </div>
    </div>
    <div class="form-actions">
      <button class="btn blue"type="submit" id="register">//这里和下面的click事件对应
        提交注册
      </button>
    </div>
  </div>
</body>

<script>
  jQuery(document).ready(function() {
    $("#register").on("click", function() {//register对象上面的id
      var telphone = $("#telphone").val();
      var password = $("#password").val();
      var age = $("#age").val();
      var gender = $("#gender").val();
      var name = $("#name").val();
      var otpCode = $("#otpCode").val();
      if (telphone == null || telphone == "") {
        alert("手机号不能为空!");
        return false;
      }
      if (password == null || password == "") {
        alert("密码不能为空!");
        return false;
      }
      if (age == null || age == "") {
        alert("年龄不能为空!");
        return false;
      }
      if (gender == null || gender == "") {
        alert("性别不能为空!");
        return false;
      }
      if (name == null || name == "") {
        alert("昵称不能为空!");
        return false;
      }
      if (otpCode == null || otpCode == "") {
        alert("验证码不能为空!");
        return false;
      }
      $.ajax({
        type: "POST",
        contentType: "application/x-www-form-urlencoded",
        url:"http://localhost:8080/user/register",
        data:{
          "telphone":telphone,
          "password":password,
          "age":age,
          "gender":gender,
          "otpCode":otpCode,
          "name":name
        },
        success:function(data) {
          if (data.status == "success") {
            alert("注册成功");
          } else {
            alert("注册失败，原因为" + data.data.errMsg);
          }
        },
        error:function(data) {
          alert("注册失败，原因为" + data.responseText);
        }
      });
      return false;
    });
  });
</script>

</html>
```

运行项目，测试在otp界面输入手机号

![17](seckill项目/17.png)

输入手机号后跳转到register.html界面输入信息后提交

![18](seckill项目/18.png)

##### 遇到的问题1

上面点提交后注册失败，启动debug模式打上断点，可以发现inSessionOtpCode是null

![19](seckill项目/19.png)

问题：@CrossOrigin的跨域操作是无法做到session共享

解决办法：指定CrossOrigin的范围

去看@CrossOrigin的源码

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CrossOrigin {
    /** @deprecated */
    @Deprecated
    String[] DEFAULT_ORIGINS = new String[]{"*"};
    
    String[] allowedHeaders() default {};
    String allowCredentials() default "";
    
```

如果不指定，默认是DEFAULT_ORIGINS，也就是ajax请求可以从任何(*)域发过来，这是解决了ajax的授信问题

但还要指定allowedHeaders、allowCredentials

allowedHeaders：允许跨域传输所有的header参数，将用于使用token放入header域，做session共享的跨域亲求

allowCredentials=true：需配合前端使用xhrFields授信后使得跨域session共享

如果前端不指定xhrFields，那么后端的crossOrigin已经授信了，但是前端请求在ajax请求没有发送xhrFields也无法实现跨域Session共享

xhrFields的意思是允许跨域的授信请求，使其session成为跨域可授信

getotp和register页面都加入

```javascript
xhrFields: {withCredentials: true},
```

##### 遇到的问题2

加上了以后inSessionOtp不为null了，但是还是注册失败，显示密码不能为空

问题：其实是Java自带的MD5的加密方式结果只支持16位

解决方法：自己实现一个Encode

```java
public String EncodeByMd5(String str) throws NoSuchAlgorithmException, UnsupportedEncodingException {
    //确定计算方法
    MessageDigest md5 = MessageDigest.getInstance("MD5");
    BASE64Encoder base64Encoder = new BASE64Encoder();
    //加密字符串
    String newstr = base64Encoder.encode(md5.digest(str.getBytes("utf-8")));
    return newstr;
}
```

register方法使用这个encode

```java
userModel.setEncrptPassword(this.EncodeByMd5(password));
```

经过测试发现注册成功，成功写入了数据库

![20](seckill项目/20.png)

##### 遇到的问题3

虽然注册成功了，却发现数据库的password表的user_id没有写入

![21](seckill项目/21.png)

原因是UserDOMapper.xml中的insertSelective没有指定主键属性

```xml
<insert id="insertSelective" parameterType="com.seckillproject.dataobject.UserDO" keyProperty="id" useGeneratedKeys="true">
```

于是在insertSelective中指定主键是id，并且是自增id

这样controller层的register没有setId，但是调用insertSelective，它能直到这是主键而且是自增的，就可以自动赋值

在service中新增userModel.setId(userDO.getId());这样在执行insertSelective后userDO是有id的值的，只需获取后放到model中以便再赋值给userPasswordDO

```java
@Override
@Transactional
public void register(UserModel userModel) throws BusinessException{
    if(userModel == null) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    if(StringUtils.isEmpty(userModel.getName()) || userModel.getGender() == null
    || userModel.getAge() == null || StringUtils.isEmpty(userModel.getTelphone())) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    //实现model转dataobject方法
    UserDO userDO = convertFromModel(userModel);
    userDOMapper.insertSelective(userDO);

    userModel.setId(userDO.getId());
    UserPasswordDO userPasswordDO = convertPasswordFromModel(userModel);
    userPasswordDOMapper.insertSelective(userPasswordDO);
    return;
}
```

再次经过测试就可以看到数据库的user_id已经有值了

![22](seckill项目/22.png)

##### ajax-error

我们构造通用CommonReturnType每次都会进入到success:function (data) 中，那么 error:function (data)是什么情况呢

当出现了网络问题连接不到服务器时，会进入到error:function (data)

```javascript
        $.ajax({
            type:"POST",
            contentType:"application/x-www-form-urlencoded",
            url:"http://localhost:8080/user/getotp",
            data:{
                "telphone":telphone,
            },
            xhrFields: {withCredentials: true},
            success:function (data) {
                if (data.status === "success") {
                    alert("短信发送成功，请查收");
                    window.location.href = "register.html";
                }else {
                    alert("短信发送失败，原因是"+data.data.errMsg);
                }
            },
            error:function (data) {
                alert("短信发送失败"+data.responseText);
            }
        })
    });
});
```

##### 总结

至此已经完成了用户注册的功能，但是还存在一个问题，就是telphone不是唯一的，相同的手机号可以不停的注册成功

解决方法：给telphone字段加入唯一索引

![23](seckill项目/23.png)

加入后再次测试，发现报错

Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.

为了将这个打印在控制台的错误能良好的返回给用户，就要修改后端代码

其实这个错误是报在了UserService的register的userDOMapper.insertSelective(userDO);这一行

使用try catch获取DuplicateKeyException重复的唯一索引的异常

```java
@Override
@Transactional
public void register(UserModel userModel) throws BusinessException{
    if(userModel == null) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    if(StringUtils.isEmpty(userModel.getName()) || userModel.getGender() == null
    || userModel.getAge() == null || StringUtils.isEmpty(userModel.getTelphone())) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    //实现model转dataobject方法
    UserDO userDO = convertFromModel(userModel);
    try{
        userDOMapper.insertSelective(userDO);
    } catch (DuplicateKeyException exception) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"手机号已重复注册");
    }

    userModel.setId(userDO.getId());
    UserPasswordDO userPasswordDO = convertPasswordFromModel(userModel);
    userPasswordDOMapper.insertSelective(userPasswordDO);
    return;
}
```

再次测试

![24](seckill项目/24.png)

#### 用户手机登录

在UserController中添加用户登录接口

```java
//用户登录接口
@RequestMapping(value = "/login",method = {RequestMethod.POST},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
public CommonReturnType login(@RequestParam(name = "telphone") String telphone,
                              @RequestParam(name = "password") String password) throws BusinessException {
    //入参校验
    if(StringUtils.isEmpty(telphone) || StringUtils.isEmpty(password)) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    //用户登录服务，校验用户登录是否合法

}
```

入参校验后要判断用户是否合法，会去调用UserService的服务

在UserService加入接口

```java
void validateLogin(String telphone,String password);
```

UserServiceImpl对其实现

```java
@Override
public void validateLogin(String telphone, String password) {
    //通过用户的手机获取用户信息

    //比对用户信息内加密的密码是否和传入的密码相匹配
}
```

这里需要通过telphone查找用户信息，去UserDOMapper.xml查发现没有对应的sql语句，所以添加

```xml
<select id="selectByTelphone" resultMap="BaseResultMap">
<!--
  WARNING - @mbg.generated
  This element is automatically generated by MyBatis Generator, do not modify.
  This element was generated on Sun Aug 02 14:43:50 CST 2020.
-->
select
<include refid="Base_Column_List" />
from user_info
where telphone = #{telphone,jdbcType=VARCHAR}
</select>
```

之后在UserDOMapper中增加映射

```java
UserDO selectByTelphone(String telphone);
```

完善错误类型

```java
USER_LOGIN_FAIL(20002,"用户手机号或密码错误")
```

完善Service层实现validateLogin，这里修改为返回UserModel

```java
@Override
public UserModel validateLogin(String telphone, String encrptPassword) throws BusinessException {
    //通过用户的手机获取用户信息
    UserDO userDO = userDOMapper.selectByTelphone(telphone);
    if(userDO == null) {
        throw new BusinessException(EmBusinessError.USER_LOGIN_FAIL);
    }

    UserPasswordDO userPasswordDO = userPasswordDOMapper.selectByUserId(userDO.getId());
    UserModel userModel = convertFromDataObject(userDO,userPasswordDO);

    //比对用户信息内加密的密码是否和传入的密码相匹配
    if(!StringUtils.equals(encrptPassword,userModel.getEncrptPassword())) {
        throw new BusinessException(EmBusinessError.USER_LOGIN_FAIL);
    }
    return userModel;
}
```

完善登录接口

```java
//用户登录接口
@RequestMapping(value = "/login",method = {RequestMethod.POST},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
public CommonReturnType login(@RequestParam(name = "telphone") String telphone,
                              @RequestParam(name = "password") String password) throws BusinessException, UnsupportedEncodingException, NoSuchAlgorithmException {
    //入参校验
    if(StringUtils.isEmpty(telphone) || StringUtils.isEmpty(password)) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
    }
    //用户登录服务，校验用户登录是否合法
    UserModel userModel = userService.validateLogin(telphone, this.EncodeByMd5(password));

    //将登录凭证加入到用户登录成功的session内
    this.httpServletRequest.getSession().setAttribute("IS_LOGIN",true);
    this.httpServletRequest.getSession().setAttribute("LOGIN_USER",userModel);

    return CommonReturnType.create(null);
}
```

新建login.html

```html
<html>
<head>
<meta charset="utf-8">
<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css"/>
<script src="static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>

</head>

<body class="login">
  <div class="content">
    <h3 class="form-title">用户登录</h3>
    <div class="form-group">
      <label class="control-label">手机号</label>
      <div>
        <input class="form-control" type="text" placeholder="手机号" name="telphone" id="telphone"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">密码</label>
      <div>
        <input class="form-control" type="password" placeholder="密码" name="password" id="password"/>
      </div>
    </div>
    <div class="form-actions">
      <button class="btn green"type="submit" id="login">
        登录
      </button>
      <button class="btn blue"type="submit" id="register">
        注册
      </button>
    </div>
  </div>
</body>

<script>
  jQuery(document).ready(function() {
    $("#register").on("click", function() {
        window.location.href = "getotp.html";
    });

    $("#login").on("click", function() {
      var telphone = $("#telphone").val();
      var password = $("#password").val();
      if (telphone == null || telphone == "") {
        alert("手机号不能为空!");
        return false;
      }
      if (password == null || password == "") {
        alert("密码不能为空!");
        return false;
      }
      $.ajax({
        type: "POST",
        contentType: "application/x-www-form-urlencoded",
        url:"http://localhost:8080/user/login",
        data:{
          "telphone":telphone,
          "password":password,
        },
        xhrFields: {withCredentials: true},
        success:function(data) {
          if (data.status == "success") {
            alert("登录成功!");
          } else {
            alert("登录失败，原因为" + data.data.errMsg);
          }
        },
        error:function(data) {
          alert("登录失败，原因为" + data.responseText);
        }
      });
      return false;
    });
  });
</script>

</html>
```

经过测试，注册用户可以正常登录

![25](seckill项目/25.png)

![26](seckill项目/26.png)

#### 优化校验规则

UserController的login接口在检验用户是否合法之前会先对参数进行一个初步校验，一般就是判空和范围

这里是手动一个个判空，可以用javax的validator接口和annotion的方式进行校验

在pom文件导入依赖

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.3.6.Final</version>
</dependency>
```

创建validator层和校验结果类

```java
package com.seckillproject.validator;

import org.apache.commons.lang3.StringUtils;

import java.util.Map;

public class ValidationResult {
    //校验结果是否有错
    private boolean hasErrors;

    //存放错误信息的map
    private Map<String,String> errorMsgMap;


    public boolean isHasErrors() {
        return hasErrors;
    }

    public void setHasErrors(boolean hasErrors) {
        this.hasErrors = hasErrors;
    }

    public Map<String, String> getErrorMsgMap() {
        return errorMsgMap;
    }

    public void setErrorMsgMap(Map<String, String> errorMsgMap) {
        this.errorMsgMap = errorMsgMap;
    }
    //实现通用的格式化字符串信息获取错误信息的msg方法
    public String getErrMsg() {
        return StringUtils.join(errorMsgMap.values().toArray(),",");
    }
}
```

实现通用的格式化字符串信息获取错误信息的msg方法是因为可能不止一个参数存在Error，用join把他们连起来

定义Validator实现类

```java
package com.seckillproject.validator;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

import javax.validation.Validation;
import javax.validation.Validator;


@Component
public class ValidatorImpl implements InitializingBean {
    private Validator validator;
    //实现校验方法并返回校验结果
    public ValidationResult validate(Object bean) {
        ValidationResult result = new ValidationResult();
        Set<ConstraintViolation<Object>> constraintViolationSet = validator.validate(bean);

        if(constraintViolationSet.size() > 0) {
            result.setHasErrors(true);
            //遍历对应的set,将set每一个元素的Violation的getMessage存放了所违背的信息
            constraintViolationSet.forEach(constraintViolation->{
                String errMsg = constraintViolation.getMessage();
                String propertyName = constraintViolation.getPropertyPath().toString();
                result.getErrorMsgMap().put(propertyName,errMsg);
            });
        }
        return result;
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        //将hibernate validator通过工厂的初始化方式使其实例化
        this.validator = Validation.buildDefaultValidatorFactory().getValidator();
    }
}

```

我们要包装出来Validator这个类，这个Validator是用Javax接口定义实现的一个工具

afterPropertiesSet方法作用：当Spring Bean初始化完成后，会回调对应ValidatorImpl的afterPropertiesSet方式

用了afterPropertiesSet的方式，通过DefaultValidatorFactory可以build一个实例化了一个实现了Javax接口的Validator校验器

定义好后怎么使用这个ValidatorImpl的validate方法呢

比如在UserServiceImpl中的register有对UserModel的校验，首先在UserModel类添加注释

```java
public class UserModel {
    private Integer id;
    @NotBlank(message = "用户名不能为空")
    private String name;
    @NotNull(message = "性别不能不填写")
    private Byte gender;
    @NotNull(message = "年龄不能不填写")
    @Min(value = 0,message = "年龄必须大于0")
    @Max(value = 150,message = "年龄必须小于150")
    private Integer age;
    @NotBlank(message = "手机号不能为空")
    private String telphone;
    private String registerMode;
    private String thirdPartyId;
    @NotBlank(message = "密码不能为空")
    private String encrptPassword;
```

@NotBlank：不能为空字符串，不能为Null

@NotNull：不能为Null

@Min：约定最小值

@Max：约定最大值

随后在UserServiceImpl中引入ValidatorImpl

```java
@Autowired
private ValidatorImpl validator;
```

将以前写的校验注释掉改成下面的形式，这里用了result.getErrMsg()，将多个参数的违背信息传到BusinessException的errMsg中

```java
// if(StringUtils.isEmpty(userModel.getName()) || userModel.getGender() == null
// || userModel.getAge() == null || StringUtils.isEmpty(userModel.getTelphone())) {
//     throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
// }
ValidationResult result = validator.validate(userModel);
if(result.isHasErrors()) {
    throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,result.getErrMsg());
}
```

##### 遇到的问题

下面进行测试，遇到了一个问题No validator could be found for constraint 'javax.validation.constraints.NotBlank' validating type 'java.lang.String'. Check configuration for 'encrptPassword'

```java
javax.validation.UnexpectedTypeException: HV000030: No validator could be found for constraint 'javax.validation.constraints.NotBlank' validating type 'java.lang.String'. Check configuration for 'encrptPassword' at org.hibernate.validator.internal.engine.constraintvalidation.ConstraintTree.throwExceptionForNullValidator(ConstraintTree.java:229) ~[hibernate-validator-5.2.4.Final.jar:5.2.4.Final]
//底下一堆
```

![28](seckill项目/28.png)

将hibernate-validator改成6.0以上版本解决

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.16.Final</version>
</dependency>
```

再次测试

![27](seckill项目/27.png)

## 商品模型管理

### 商品创建

设计数据库之前应该先设计领域模型，不然就像之前一样，把用户的密码字段也放在了用户主表中

首先创建ItemModle

```java
package com.seckillproject.service.model;

import java.math.BigDecimal;

public class ItemModel {
    private Integer id;

    //商品名
    private String title;
    //商品价格
    private BigDecimal price;

    //商品库存
    private Integer stock;

    //商品描述
    private String description;

    //商品销量
    private Integer sales;

    //商品描述图片URL
    private String imgURL;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        this.stock = stock;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Integer getSales() {
        return sales;
    }

    public void setSales(Integer sales) {
        this.sales = sales;
    }

    public String getImgURL() {
        return imgURL;
    }

    public void setImgURL(String imgURL) {
        this.imgURL = imgURL;
    }
}
```

#### 数据库层

然后思考数据库是否和模型一模一样，其实不一样

stock库存字段确实和商品是一对一的关系，但是考虑到商品库存和交易流水相关，每次对商品表的操作就是对库存表的操作，因此最好stock拆到另外的表中

sales销量字段也是这个问题，销量对应商品模型当中是交易行为发生之后产生的一个累加，在这个项目里暂且用于展示，放在这个表中，当用户发生交易行为时，通过异步的方式给销量字段加1，而不会影响下单主链路，因此暂时放在这里

item数据表内容如下

![29](seckill项目/32.png)

接着去创建库存表

![30](seckill项目/30.png)

#### MyBatis自动生成插件

接下来用mybatis-generator去自动生成

这里要将pom文件里mybatis自动生成器的自动覆盖置为false，不然新生成的就把之前的user覆盖掉了

在mybatis-generator.xml里新增生成对应表和类名

```xml
<table tableName="item" domainObjectName="ItemDO" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
<table tableName="item_stock" domainObjectName="ItemStockDO" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
```

可以看到对应的文件已经生成了

![31](seckill项目/31.png)

在ItemDOMapper.xml中，找到insert和insertSelective方法，依然要为它们指定主键并且自增的方式

![33](seckill项目/33.png)

同样的道理，去ItemStockDOMapper.xml，找到insert和insertSelective方法，为它们指定主键并且自增的方式

**需要注意ItemModel里price是BigDecimal类型的，而数据库里是Double类型的，ItemDO是Double类型的，为什么要用BigDecimal？**

**因为Double类型Json传到前端会有精度问题，可能1.9会变成1.99999**

#### Service层

现在完成了一套item模型的设计，再去创建ItemSerivce

```java
package com.seckillproject.service;

import com.seckillproject.service.model.ItemModel;

import java.util.List;

public interface ItemService {
    //创建商品
    ItemModel createItem(ItemModel itemModel);
    //商品列表浏览
    List<ItemModel> listItem();
    //商品详情浏览
    ItemModel getItemModle(Integer id);
}
```

在ItemServiceImpl中实现

```java
package com.seckillproject.service.impl;

import com.seckillproject.dao.ItemDOMapper;
import com.seckillproject.dao.ItemStockDOMapper;
import com.seckillproject.dataobject.ItemDO;
import com.seckillproject.dataobject.ItemStockDO;
import com.seckillproject.error.BusinessException;
import com.seckillproject.error.EmBusinessError;
import com.seckillproject.service.ItemService;
import com.seckillproject.service.model.ItemModel;
import com.seckillproject.validator.ValidationResult;
import com.seckillproject.validator.ValidatorImpl;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
public class ItemServiceImpl implements ItemService {

    @Autowired
    private ValidatorImpl validator;
    @Autowired
    private ItemDOMapper itemDOMapper;
    @Autowired
    private ItemStockDOMapper itemStockDOMapper;
    private ItemStockDO convertItemStockDOFromItemModel(ItemModel itemModel) {
        if(itemModel == null) {
            return null;
        }
        ItemStockDO itemStockDO = new ItemStockDO();
        //这里不会copy类型不一样的price字段
        itemStockDO.setItemId(itemModel.getId());
        itemStockDO.setStock(itemModel.getStock());
        return itemStockDO;
    }
    private ItemDO convertItemDOFromItemModel(ItemModel itemModel) {
        if(itemModel == null) {
            return null;
        }
        ItemDO itemDO = new ItemDO();
        //这里不会copy类型不一样的price字段
        BeanUtils.copyProperties(itemModel,itemDO);
        itemDO.setPrice(itemModel.getPrice().doubleValue());
        return itemDO;
    }
    @Override
    @Transactional
    public ItemModel createItem(ItemModel itemModel) throws BusinessException {
        //校验入参
        ValidationResult result = validator.validate(itemModel);
        if(result.isHasErrors()) {
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,result.getErrMsg());
        }

        //转化ItemModel到dataobject
        ItemDO itemDO = this.convertItemDOFromItemModel(itemModel);

        //写入数据库
        itemDOMapper.insertSelective(itemDO);
        itemModel.setId(itemDO.getId());

        ItemStockDO itemStockDO = this.convertItemStockDOFromItemModel(itemModel);
        itemStockDOMapper.insertSelective(itemStockDO);
        //返回创建完成的对象
        return this.getItemById(itemModel.getId());
    }

    @Override
    public List<ItemModel> listItem() {
        return null;
    }

    @Override
    public ItemModel getItemById(Integer id) {
        ItemDO itemDO = itemDOMapper.selectByPrimaryKey(id);
        if(itemDO == null) {
            return null;
        }
        //操作获得库存数量 
    }
}
```

接着要去getItemById中获取库存数量，需要在ItemStockDOMapper.xml中增加通过item_id查stock的SQL语句

```xml
<select id="selectByItemId" parameterType="java.lang.Integer" resultMap="BaseResultMap">
<!--
  WARNING - @mbg.generated
  This element is automatically generated by MyBatis Generator, do not modify.
  This element was generated on Sun Aug 09 11:17:17 CST 2020.
-->
select
<include refid="Base_Column_List" />
from item_stock
where item_id = #{itemId,jdbcType=INTEGER}
</select>
```

在ItemStockDOMapper中增加映射语句

```java
ItemStockDO selectByItemId(Integer itemId);
```

完善getItemById方法

```java
@Override
public ItemModel getItemById(Integer id) {
    ItemDO itemDO = itemDOMapper.selectByPrimaryKey(id);
    if(itemDO == null) {
        return null;
    }
    //操作获得库存数量
    ItemStockDO itemStockDO = itemStockDOMapper.selectByItemId(itemDO.getId());

    //将dataobject转化成model领域模型
    ItemModel itemModel = convertModleFromDataObject(itemDO,itemStockDO);
    return itemModel;
}
private ItemModel convertModleFromDataObject(ItemDO itemDo,ItemStockDO itemStockDO) {
    ItemModel itemModel = new ItemModel();
    BeanUtils.copyProperties(itemDo,itemModel);

    itemModel.setPrice(new BigDecimal(itemDo.getPrice()));
    itemModel.setStock(itemStockDO.getStock());
    return itemModel;
}
```

#### Controller层

接下来是Controller层，创建ItemController和用于前端展示的ItemVO

将ItemModel的字段复制过来，然后看是否都可以展示，结果是都可以展示给前端

将UserController的下面代码复制过去，因为item同样要处理跨域的请求

```java
@RequestMapping("/item")
@CrossOrigin(allowCredentials = "true",allowedHeaders = "*")
```

然后写一个创建商品方法，参数没有sales，因为销量和创建商品无关

```java
package com.seckillproject.controller;

import com.seckillproject.controller.viewobject.ItemVO;
import com.seckillproject.error.BusinessException;
import com.seckillproject.response.CommonReturnType;
import com.seckillproject.service.ItemService;
import com.seckillproject.service.model.ItemModel;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.math.BigDecimal;

@Controller
@RequestMapping("/item")
@CrossOrigin(allowCredentials = "true",allowedHeaders = "*")
public class ItemController extends BaseController{
    @Autowired
    ItemService itemService;
    //创建商品的controller
    public CommonReturnType createItem(@RequestParam(name = "title") String title,
                                       @RequestParam(name = "description") String description,
                                       @RequestParam(name = "price") BigDecimal price,
                                       @RequestParam(name = "stock") Integer stock,
                                       @RequestParam(name = "imgUrl") String imgUrl) throws BusinessException {
        //封装service请求来创建商品
        ItemModel itemModel = new ItemModel();
        itemModel.setTitle(title);
        itemModel.setPrice(price);
        itemModel.setStock(stock);
        itemModel.setDescription(description);
        itemModel.setImgURL(imgUrl);

        ItemModel itemModelForReturn = itemService.createItem(itemModel);
        ItemVO itemVO = convertVOFromModel(itemModelForReturn);
        return CommonReturnType.create(itemVO);

    }
    private ItemVO convertVOFromModel(ItemModel itemModel) {
        if(itemModel == null) {
            return null;
        }
        ItemVO itemVO = new ItemVO();
        BeanUtils.copyProperties(itemModel,itemVO);
        return itemVO;
    }
}
```

#### createitem.html

仿照以前的html创建createitem.html

```html
<html>
<head>
<meta charset="utf-8">
<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css"/>
<script src="static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>

</head>

<body class="login">
  <div class="content">
    <h3 class="form-title">创建商品</h3>
    <div class="form-group">
      <label class="control-label">商品名</label>
      <div>
        <input class="form-control" type="text" placeholder="商品名" name="title" id="title"/>
      </div>
    </div>
     <div class="form-group">
      <label class="control-label">商品描述</label>
      <div>
        <input class="form-control" type="text" placeholder="商品描述" name="description" id="description"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品价格</label>
      <div>
        <input class="form-control" type="text" placeholder="商品价格" name="price" id="price"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品库存</label>
      <div>
        <input class="form-control" type="text" placeholder="商品库存" name="stock" id="stock"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品图片</label>
      <div>
        <input class="form-control" type="text" placeholder="商品图片" name="imgUrl" id="imgUrl"/>
      </div>
    </div>
    <div class="form-actions">
      <button class="btn blue"type="submit" id="create">
        提交创建
      </button>
    </div>
  </div>
</body>

<script>
  jQuery(document).ready(function() {
    $("#create").on("click", function() {
      var title = $("#title").val();
      var description = $("#description").val();
      var price = $("#price").val();
      var stock = $("#stock").val();
      var imgUrl = $("#imgUrl").val();
      if (title == null || title == "") {
        alert("商品标题不能为空!");
        return false;
      }
      if (description == null || description == "") {
        alert("商品描述不能为空!");
        return false;
      }
      if (price == null || price == "") {
        alert("商品价格不能为空!");
        return false;
      }
      if (stock == null || stock == "") {
        alert("商品库存不能为空!");
        return false;
      }
      if (imgUrl == null || imgUrl == "") {
        alert("商品图片不能为空!");
        return false;
      }
      $.ajax({
        type: "POST",
        contentType: "application/x-www-form-urlencoded",
        url:"http://localhost:8080/item/create",
        data:{
          "title":title,
          "description":description,
          "price":price,
          "stock":stock,
          "imgUrl":imgUrl,
        },
        xhrFields: {withCredentials: true},
        success:function(data) {
          if (data.status == "success") {
            alert("创建成功!");
          } else {
            alert("创建失败，原因为" + data.data.errMsg);
          }
        },
        error:function(data) {
          alert("创建失败，原因为" + data.responseText);
        }
      });
      return false;
    });
  });
</script>

</html>
```

##### 遇到的问题

之后进行测试

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class com.seckillproject.controller.viewobject.ItemVO and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: com.seckillproject.response.CommonReturnType["data"])
	at com.fasterxml.jackson.databind.exc.InvalidDefinitionException.from(InvalidDefinitionException.java:77)
```

原因是将ItemVO序列化返回给前端的时候缺少了一些东西

解决方法是ItemVO加上get set方法

再次测试

![34](seckill项目/34.png)

### 商品详情页

#### Controller层

在ItemController种创建商品详情页浏览接口

```java
//商品详情页浏览
@RequestMapping(value = "/get",method = {RequestMethod.GET},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
public CommonReturnType getItem(@RequestParam(name = "id") Integer id) {
    ItemModel itemModel = itemService.getItemById(id);

    ItemVO itemVO = convertVOFromModel(itemModel);
    return CommonReturnType.create(itemVO);
}
```

#### getitem.html

```html
<html>
<head>
<meta charset="utf-8">
<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css"/>
<script src="static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>

</head>

<body class="login">
  <div class="content">
    <h3 class="form-title">商品详情</h3>
    </div>
    <div class="form-group">
      <label class="control-label">商品名</label>
      <div>
        <label class="control-label" id="title"/>
      </div>
    </div>
     <div class="form-group">
      <label class="control-label">商品描述</label>
      <div>
        <label class="control-label" id="description"/>
      </div>
    </div>
    <div id="normalPriceContainer" class="form-group">
      <label class="control-label">商品价格</label>
      <div>
        <label class="control-label" id="price"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品库存</label>
      <div>
        <label class="control-label" id="stock"/>
      </div>
    </div>
    <div class="form-group">
      <div>
        <img style="width:200px;heigth:auto" id="imgUrl">
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品销量</label>
      <div>
        <label class="control-label" id="sales"/>
      </div>
  </div>
</body>

<script>
  function getParam(paramName) {
	    paramValue = "", isFound = !1;         
	    if (this.location.search.indexOf("?") == 0 && this.location.search.indexOf("=") > 1) {               
	       arrSource = unescape(this.location.search).substring(1, this.location.search.length).split("&"), i = 0;               
	        while (i < arrSource.length && !isFound) arrSource[i].indexOf("=") > 0 && arrSource[i].split("=")[0].toLowerCase() ==  paramName.toLowerCase() && (paramValue = arrSource[i].split("=")[1], isFound = !0), i++  }
	        	
	        return paramValue == "" && (paramValue = null), paramValue 
	}

  var g_itemVO = {}

    //获取商品详情
      $.ajax({
        type: "GET",
        url:"http://localhost:8080/item/get",
        data:{
          "id":getParam("id"),
        },
        xhrFields: {withCredentials: true},
        success:function(data) {
          if (data.status == "success") {
            g_itemVO = data.data;
            reloadDom();
            setInterval(reloadDom, 1000);
          } else {
            alert("获取信息失败，原因为" + data.data.errMsg);
          }
        },
        error:function(data) {
          alert("获取信息失败，原因为" + data.responseText);
        }
      });
      return false;
    });
</script>

</html>
```

### 商品列表页

#### Dao层

先在ItemDOMapper.xml中添加SQL语句

```xml
<select id="listItem" resultMap="BaseResultMap">
<!--
  WARNING - @mbg.generated
  This element is automatically generated by MyBatis Generator, do not modify.
  This element was generated on Sun Aug 09 11:17:17 CST 2020.
-->
select
<include refid="Base_Column_List" />
from item order by sales DESC
</select>
```

这里返回所有的商品列表，并且按销量降序

在ItemDOMapper中加入

```java
List<ItemDO> listItem();
```

#### Service层

接着去ItemSericeImpl中实现它

```java
@Override
public List<ItemModel> listItem() {
    List<ItemDO> itemDOList = itemDOMapper.listItem();
    List<ItemModel> itemModelList = itemDOList.stream().map(itemDO -> {
        ItemStockDO itemStockDO = itemStockDOMapper.selectByItemId(itemDO.getId());
        ItemModel itemModel = this.convertModleFromDataObject(itemDO, itemStockDO);
        return itemModel;
    }).collect(Collectors.toList());
    return itemModelList;
}
```

这里使用Java8的stream，将List里的每个ItemDO都map成ItemModel，最后使用collect方法转出List对象

#### Controller层

接着去ItemController，同样的方法转出itemVOList

```java
//商品列表页面浏览
@RequestMapping(value = "/list",method = {RequestMethod.GET},consumes = {CONTENT_TYPE_FORMED})
@ResponseBody
public CommonReturnType listItem() {
    List<ItemModel> itemModelList = itemService.listItem();

    //使用stream api将list内的itemModel转化为ItemVO
    List<ItemVO> itemVOList = itemModelList.stream().map(itemModel -> {
        ItemVO itemVO = this.convertVOFromModel(itemModel);
        return itemVO;
    }).collect(Collectors.toList());
    return CommonReturnType.create(itemVOList);
}
```

这里将consumes = {CONTENT_TYPE_FORMED}去掉，然后测试

![35](seckill项目/35.png)

#### listitem.html

```html
<html>
<head>
<meta charset="utf-8">
<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css"/>
<script src="static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>

</head>

<body>
  <div class="content">
    <h3 class="form-title">商品列表浏览</h3>
    <div class="table-responsive">
      <table class="table">
        <thead>
          <tr>
            <th>商品名</th>
            <th>商品图片</th>
            <th>商品描述</th>
            <th>商品价格</th>
            <th>商品库存</th>
            <th>商品销量</th>
          </tr>
        </thead>
        <tbody id="container">

        </tbody>
      </table>
    </div>
  </div>
</body>

<script>
  //定义全局商品数组信息
  var g_itemList = [];

  jQuery(document).ready(function() {
      $.ajax({
        type: "GET",
        url:"http://localhost:8080/item/list",
        xhrFields: {withCredentials: true},
        success:function(data) {
          if (data.status == "success") {
            g_itemList = data.data;
            reloadDom();
          } else {
            alert("获取商品信息失败，原因为" + data.data.errMsg);
          }
        },
        error:function(data) {
          alert("获取商品信息失败，原因为" + data.responseText);
        }
      });
      return false;
    });

  function reloadDom() {
    for (var i = 0; i < g_itemList.length; i ++) {
      var itemVO = g_itemList[i];
      var dom = "<tr data-id = '"+ itemVO.id +"' id='itemDetail" + itemVO.id + "'><td>" + itemVO.title + "</td><td><img style='width:100px;height:auto;' src='" + itemVO.imgUrl + "'/></td><td>" + 
                itemVO.description + "</td><td>" + itemVO.price + "</td><td>" + itemVO.stock + "</td><td>" + itemVO.sales + "</td></tr>";
      $("#container").append($(dom));
      $("#itemDetail" + itemVO.id).on("click", function(e) {
        window.location.href = "getitem.html?id=" + $(this).data("id");
      });
    }
  }
</script>

</html>
```

## 交易模型管理

### 交易模型创建

#### Service层

##### OrderModel

在Service的Model中创建OrderModel

```java
package com.seckillproject.service.model;

import java.math.BigDecimal;

//用户下单的交易模型
public class OrderModel {

    //2020081200001812
    private String id;//id为String类型而不是Integer,因为会有明显的字符串拼接,前面是年月日后面是什么什么
    //购买的用户id
    private Integer userId;

    //购买的商品id
    private Integer itemId;//商品的单价可能会变化，这里记录当时购买的时候的单价是多少

    //购买商品的单价
    private BigDecimal itemPrice;

    //购买的数量
    private Integer amount;

    //购买的金额
    private BigDecimal orderPrice;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public Integer getItemId() {
        return itemId;
    }

    public void setItemId(Integer itemId) {
        this.itemId = itemId;
    }

    public Integer getAmount() {
        return amount;
    }

    public void setAmount(Integer amount) {
        this.amount = amount;
    }

    public BigDecimal getOrderPrice() {
        return orderPrice;
    }

    public void setOrderPrice(BigDecimal orderPrice) {
        this.orderPrice = orderPrice;
    }

    public BigDecimal getItemPrice() {
        return itemPrice;
    }

    public void setItemPrice(BigDecimal itemPrice) {
        this.itemPrice = itemPrice;
    }
}
```

这里仅考虑的是每次用户只买一件商品的情况

#### MyBatis自动生成插件

在mybatis-generator.xml中添加

```xml
<table tableName="order_info" domainObjectName="OrderDO" enableCountByExample="false"
enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false"
selectByExampleQueryId="false" ></table>
```

#### Service层

##### createOrder

创建OrderService

```java
package com.seckillproject.service.model;

public interface OrderService {
    OrderModel createOrder(Integer userId,Integer itemId,Integer amount);   
}
```

接着去Impl中实现它

```java
package com.seckillproject.service.impl;

import com.seckillproject.dao.OrderDOMapper;
import com.seckillproject.dataobject.OrderDO;
import com.seckillproject.error.BusinessException;
import com.seckillproject.error.EmBusinessError;
import com.seckillproject.service.ItemService;
import com.seckillproject.service.UserService;
import com.seckillproject.service.model.ItemModel;
import com.seckillproject.service.model.OrderModel;
import com.seckillproject.service.model.OrderService;
import com.seckillproject.service.model.UserModel;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;

@Service
public class OrderServiceImpl implements OrderService {
    @Autowired
    private ItemService itemService;

    @Autowired
    private UserService userService;

    @Autowired
    private OrderDOMapper orderDOMapper;
    @Override
    @Transactional
    public OrderModel createOrder(Integer userId, Integer itemId, Integer amount) throws BusinessException {
        //1.校验下单状态：下单商品是否存在、用户是否合法、购买数量是否正确
        ItemModel itemModel = itemService.getItemById(itemId);
        if(itemModel == null) {
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"商品信息不存在");
        }
        UserModel userModel = userService.getUserById(userId);
        if(userModel == null) {
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"用户信息不存在");
        }
        if(amount <= 0 || amount > 99) {
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"数量信息不正确");
        }
        //2.落单减库存
        boolean result = itemService.decreaseStock(itemId, amount);
        if(!result) {
            throw new BusinessException(EmBusinessError.STOCK_NOT_ENOUGH);
        }
        //3.订单入库
        OrderModel orderModel = new OrderModel();
        orderModel.setUserId(userId);
        orderModel.setItemId(itemId);
        orderModel.setAmount(amount);
        orderModel.setItemPrice(itemModel.getPrice());
        orderModel.setOrderPrice(itemModel.getPrice().multiply(new BigDecimal(amount)));
        //生成交易流水号也就是订单号

        OrderDO orderDO = convertFromOrderModel(orderModel);
        orderDOMapper.insertSelective(orderDO);
        //4.返回前端

        return null;
    }
    private String generateOrderNo() {
        //订单号有十六位
        //前八位为时间信息,年月日,这个可以用于以后用SQL语句查询某年某月某日的信息

        //中间六位为自增序列,用于在某年某月某日一天的订单量不超过六位,保证不重复

        //最后两位为分库分表位,00-99,假如我们用userID%100作为分库分表位,即将订单按userId分到100个库里的100张表里,分散数据库压力,所以最后两位作为分库分表的路由信息
    }
    private OrderDO convertFromOrderModel(OrderModel orderModel) {
        if(orderModel == null) {
            return null;
        }
        OrderDO orderDO = new OrderDO();
        BeanUtils.copyProperties(orderModel,orderDO);
        return orderDO;
    }
}

```

**落单减库存，落单的时候对库存加锁，可能会出现有人恶意刷订单，但是不付钱，等订单失效到期**

**支付减库存，落单的时候检查是否有库存，支付的时候才对库存加锁，但可能会出现超卖的情况**

##### decreaseStock

这里采用落单减库存

这里就体现了item表和item_stock表的设计原理了，item表大部分情况下用于查询，库存字段在高压情况下做降解，也就是将item拆成item展示服务和库存服务

为了保证冻结状态原子性要对数据表记录加锁

实现库存扣减需要在ItemService中添加

```java
//库存扣减
boolean decreaseStock(Integer userId,Integer amount) throws BusinessException;
```

接下来在Impl中实现decreaseStock

```java
@Override
@Transactional
public boolean decreaseStock(Integer itemId, Integer amount) throws BusinessException {
    int affectedRow = itemStockDOMapper.decreaseStock(itemId, amount);
    if(affectedRow > 0) {
        //更新库存成功
        return true;
    }else {
        //更新库存失败
        return false;
    }
}
```

#### Dao层

##### decreaseStock

要改ItemStockDOMapper的SQL语句

```xml
</update>
<update id="decreaseStock">
update item
set stock = stock - #{amount}
where item_id = #{item_id} and stock >= #{amount}
</update>
```

上面这条SQL语句是原子执行的

在ItemStockDOMapper中添加

```java
int decreaseStock(@Param("itemId") Integer itemId,@Param("amount") Integer amount);
```

为什么使用Param看[这里](https://blog.csdn.net/sinat_33010325/article/details/84261662?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

这里让decreaseStock返回一个int，也就是它受影响的条目数，通过条目数为0或者1来判断它是否执行成功

##### generateOrderNo

而对于generateOrderNo的第二布，获取自增序列

由于现在使用的是MySQL，不像Oracle可以获得自增序列，于是在数据库建一个通用的表

![36](seckill项目/36.png)

这张表创建的意义是初始化一些sequence，并且sequence初始是0，每次都加一个步长

![37](seckill项目/37.png)

使用Mybatis-generator自动生成后，要修改一些地方

![38](seckill项目/38.png)

在SequenceDOMapper.xml中添加SQL语句

```xml
<select id="getSequenceByName" parameterType="java.lang.String" resultMap="BaseResultMap">
select
<include refid="Base_Column_List" />
from sequence_info
where name = #{name,jdbcType=VARCHAR} for update
</select>
```

这里使用select的for update方法，可以对数据库加锁

SequenceDOMapper中添加

```java
SequenceDO getSequenceByName(String name);
```

最后完善generateOrderNo

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
private String generateOrderNo() {
    //订单号有十六位
    StringBuilder stringBuilder = new StringBuilder();
    //前八位为时间信息,年月日,这个可以用于以后用SQL语句查询某年某月某日的信息
    LocalDateTime now = LocalDateTime.now();
    String nowDate = now.format(DateTimeFormatter.ISO_DATE).replace("-", "");
    stringBuilder.append(nowDate);
    //中间六位为自增序列,用于在某年某月某日一天的订单量不超过六位,保证不重复
    //获取当前sequence
    int sequence = 0;
    SequenceDO sequenceDO = sequenceDOMapper.getSequenceByName("order_info");
    sequence = sequenceDO.getCurrentValue();
    sequenceDO.setCurrentValue(sequenceDO.getCurrentValue() + sequenceDO.getStep());
    sequenceDOMapper.updateByPrimaryKeySelective(sequenceDO);
    String sequenceStr = String.valueOf(sequence);
    for(int i = 0; i < 6-sequenceStr.length(); i++) {
        stringBuilder.append("0");
    }
    stringBuilder.append(sequenceStr);
    //最后两位为分库分表位,00-99,假如我们用userID%100作为分库分表位,即将订单按userId分到100个库里的100张表里,分散数据库压力,所以最后两位作为分库分表的路由信息
    stringBuilder.append("00");
    return stringBuilder.toString();
}
```

##### 遇到的问题

存在的问题：没有设置上限，很可能超过6位

解决办法：在sequence_info表里添加一列max_value字段设置最大值以及init_value初始值，如果current_value+step超过了max_value，就从初始值开始循环使用

存在的问题：因为generateOrderNo方法是写在createOrder方法中的，createOrder方法又加了事务标签，当事务回滚的时候，generateOrderNo也会回滚，因此下一次会createOrder会拿到上一次回滚的流水号，这是不允许的，因为要保证订单号的全局唯一性，就算那一次createOrder失败了，也要保证不同

解决办法：在generateOrderNo上添加@Transactional(propagation = Propagation.REQUIRES_NEW)

只需要让generateOrderNo执行完成后不需要将事务handle在createOrder，而是直接提交掉，这样就能保证执行完这段代码，外部事务不论成功与否，里面的事务都会提交掉，对应的订单号都会被使用掉

#### Controller层

创建OrderController，userId先空着

```java
package com.seckillproject.controller;

import com.seckillproject.error.BusinessException;
import com.seckillproject.error.EmBusinessError;
import com.seckillproject.response.CommonReturnType;
import com.seckillproject.service.model.OrderService;
import com.seckillproject.service.model.UserModel;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;

@Controller("order")
@RequestMapping("/order")
@CrossOrigin(allowCredentials = "true",allowedHeaders = "*")
public class OrderController extends BaseController{
    @Autowired
    private HttpServletRequest httpServletRequest;
    @Autowired
    public OrderService orderService;
    //封装下单请求
    @RequestMapping(value = "/createorder",method = {RequestMethod.POST},consumes = {CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType createOrder(@RequestParam(name = "itemId") Integer itemId,
                                        @RequestParam(name = "amount") Integer amount) throws BusinessException {

        Boolean isLogin = (Boolean)httpServletRequest.getSession().getAttribute("IS_LOGIN");
        if(isLogin == null || !isLogin.booleanValue()) {
            throw new BusinessException(EmBusinessError.USER_NOT_LOGIN,"用户还未登录，不能下单");
        }
        //获取用户的登录信息
        UserModel userModel = (UserModel) httpServletRequest.getSession().getAttribute("LOGIN_USER");

        orderService.createOrder(userModel.getId(),itemId,amount);
        return CommonReturnType.create(null);
    }
}

```

这里新增了一个错误码USER_NOT_LOGIN(20003,"用户还未登录")

和前端做了一个约定，前端判断错误码如果是20003就跳转到login.html

测试如下

![39](seckill项目/39.png)

登录后下单可以看到数据库已经有了

![40](seckill项目/40.png)

但是item_price和order_price都没进去，原因在于OrderDO的item_price是Double类型，Order_Model是BigDecimal类型的，所以需要转换

```java
private OrderDO convertFromOrderModel(OrderModel orderModel) {
    if(orderModel == null) {
        return null;
    }
    OrderDO orderDO = new OrderDO();
    BeanUtils.copyProperties(orderModel,orderDO);
    orderDO.setItemPrice(orderModel.getItemPrice().doubleValue());
    orderDO.setOrderPrice(orderModel.getOrderPrice().doubleValue());
    return orderDO;
}
```

这里还遗漏了一件事情，就是商品卖出去以后销量会增加

在ItemService里增加

```java
void increaseSales(Integer itemId,Integer amount) throws BusinessException;
```

在ItemDOMapper.xml中增加SQL

```xml
</update>
<update id="increaseSales">

update item
set sales = sales + #{amount}
where id = #{id,jdbcType=INTEGER}
</update>
```

在ItemDOMapper里增加

```java
int increaseSales(@Param("id") Integer id,@Param("amount") Integer amount);
```

最后完成increaseSales

```java
//落单成功就增加销量
@Override
@Transactional
public void increaseSales(Integer itemId, Integer amount) throws BusinessException {
    itemDOMapper.increaseSales(itemId,amount);
}
```

在OrderServiceImpl的createOrder中增加

```java
//增加商品的销量
itemService.increaseSales(itemId,amount);
```

测试，下单前库存97，销量67

![41](seckill项目/41.png)

下单后变成了96和68

![42](seckill项目\42.png)

## 秒杀模型管理

### 活动模型创建

#### 设计数据库

![43](seckill项目/43.png)

MySQL datetime类型默认是年-月-日 时:分:秒的格式

#### Model

新建PromoModel

```java
package com.seckillproject.service.model;

import org.joda.time.DateTime;

import java.math.BigDecimal;

public class PromoModel {
    private Integer id;

    //秒杀活动名称
    private String promoName;

    //秒杀活动的开始时间
    private DateTime startDate;

    //秒杀活动的适用商品
    private Integer itemId;
    //秒杀活动的商品价格
    private BigDecimal promoItemPrice;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getPromoName() {
        return promoName;
    }

    public void setPromoName(String promoName) {
        this.promoName = promoName;
    }

    public DateTime getStartDate() {
        return startDate;
    }

    public void setStartDate(DateTime startDate) {
        this.startDate = startDate;
    }

    public Integer getItemId() {
        return itemId;
    }

    public void setItemId(Integer itemId) {
        this.itemId = itemId;
    }

    public BigDecimal getPromoItemPrice() {
        return promoItemPrice;
    }

    public void setPromoItemPrice(BigDecimal promoItemPrice) {
        this.promoItemPrice = promoItemPrice;
    }
}
```

这里DateTime是Joda的DateTime

### MyBatis自动生成器

mybatis-generator.xml中添加

```xml
<table tableName="promo" domainObjectName="PromoDO" enableCountByExample="false"
enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false"
selectByExampleQueryId="false" ></table>
```

### 活动模型商品模型结合

给秒杀数据库和秒杀模型添加字段endDate作为结束时间

为了判断活动进行和结束状态给PromoModel加一个字段status，通过用结束时间和开始时间在PromoModel中聚合status字段，可以让外层直接调用判断，而不用再自己判断

#### PromoService层

```java
package com.seckillproject.service.impl;

import com.seckillproject.dao.PromoDOMapper;
import com.seckillproject.dataobject.PromoDO;
import com.seckillproject.service.PromoService;
import com.seckillproject.service.model.PromoModel;
import org.joda.time.DateTime;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;

@Service
public class PromoServiceImpl implements PromoService {
    @Autowired
    private PromoDOMapper promoDOMapper;
    @Override
    public PromoModel getPromoByItemId(Integer itemId) {
        //获取对应商品的秒杀活动信息

        PromoDO promoDO = promoDOMapper.selectByItemId(itemId);
        //转化为领域模型
        PromoModel promoModel = convertFromDataObject(promoDO);
        if(promoModel == null) {
            return null;
        }
        //判断当前时间是否秒杀活动开始或正在进行
        if(promoModel.getStartDate().isAfterNow()) {
            promoModel.setStatus(1);
        } else if(promoModel.getEndDate().isBeforeNow()) {
            promoModel.setStatus(3);
        } else {
            promoModel.setStatus(2);
        }
        return promoModel;
    }
    private PromoModel convertFromDataObject(PromoDO promoDO) {
        if(promoDO == null) {
            return null;
        }
        PromoModel promoModel = new PromoModel();
        BeanUtils.copyProperties(promoDO,promoModel);
        promoModel.setPromoItemPrice(new BigDecimal(promoDO.getPromoItemPrice()));//double->bigdecimal
        promoModel.setStartDate(new DateTime(promoDO.getStartDate()));//util.Date->joda DateTime
        promoModel.setEndDate(new DateTime(promoDO.getEndDate()));//util.Date->joda DateTime
        return promoModel;
    }
}
```

#### ItemModel

商品应该有自己的一套秒杀活动的属性

在ItemModel中增加

```java
//使用聚合模型,如果promoModel不为空，则表示拥有未结束的秒杀活动(还未开始+进行中)
private PromoModel promoModel;
```

#### ItemServiceImpl

增加获取活动商品信息

```java
@Override
public ItemModel getItemById(Integer id) {
    ItemDO itemDO = itemDOMapper.selectByPrimaryKey(id);
    if(itemDO == null) {
        return null;
    }
    //操作获得库存数量
    ItemStockDO itemStockDO = itemStockDOMapper.selectByItemId(itemDO.getId());

    //将dataobject转化成model领域模型
    ItemModel itemModel = convertModleFromDataObject(itemDO,itemStockDO);

    //获取活动商品信息
    PromoModel promoModel = promoService.getPromoByItemId(itemModel.getId());
    if(promoModel != null && promoModel.getStatus().intValue() != 3) {
        itemModel.setPromoModel(promoModel);
    }
    return itemModel;
}
```

#### ItemVO

新增字段

```java
//商品是否在秒杀活动中，0没有秒杀活动，1秒杀活动待开始，2秒杀活动进行中
private Integer promoStatus;

//秒杀活动价格
private BigDecimal promoPrice;

//秒杀活动ID
private Integer promoId;

//秒杀活动开始时间
private DateTime startDate;
```

#### ItemController

Model转ItemVO的时候给上面的新字段赋值

```java
private ItemVO convertVOFromModel(ItemModel itemModel) {
    if(itemModel == null) {
        return null;
    }
    ItemVO itemVO = new ItemVO();
    BeanUtils.copyProperties(itemModel,itemVO);
    if(itemModel.getPromoModel() != null) {
        itemVO.setPromoStatus(itemModel.getPromoModel().getStatus());
        itemVO.setPromoId(itemModel.getId());
        itemVO.setStartDate(itemModel.getPromoModel().getStartDate());
        itemVO.setPromoPrice(itemModel.getPromoModel().getPromoItemPrice());
    } else {
        itemVO.setPromoStatus(0);
    }
    return itemVO;
}
```

#### getItem.html

```java
<html>
<head>
<meta charset="utf-8">
<link href="static/assets/global/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/global/css/components.css" rel="stylesheet" type="text/css"/>
<link href="static/assets/admin/pages/css/login.css" rel="stylesheet" type="text/css"/>
<script src="static/assets/global/plugins/jquery-1.11.0.min.js" type="text/javascript"></script>

</head>

<body class="login">
  <div class="content">
    <h3 class="form-title">商品详情</h3>
     <div id="promoStartDateContainer" class="form-group">
      <label style="color:red" class="control-label">秒杀开始时间</label>
      <div>
        <label style="color:red" class="control-label" id="promoStartDate"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品名</label>
      <div>
        <label class="control-label" id="title"/>
      </div>
    </div>
     <div class="form-group">
      <label class="control-label">商品描述</label>
      <div>
        <label class="control-label" id="description"/>
      </div>
    </div>
    <div id="normalPriceContainer" class="form-group">
      <label class="control-label">商品价格</label>
      <div>
        <label class="control-label" id="price"/>
      </div>
    </div>
    <div id="promoPriceContainer" class="form-group">
      <label style="color:red" class="control-label">秒杀价格</label>
      <div>
        <label style="color:red" class="control-label" id="promoPrice"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品库存</label>
      <div>
        <label class="control-label" id="stock"/>
      </div>
    </div>
    <div class="form-group">
      <div>
        <img style="width:200px;heigth:auto" id="imgUrl">
      </div>
    </div>
    <div class="form-group">
      <label class="control-label">商品销量</label>
      <div>
        <label class="control-label" id="sales"/>
      </div>
    </div>
    <div class="form-group">
      <button type="submit" class="btn blue" id = "createorder">
        下单
      </button>
    </div>
  </div>
</body>

<script>
  function getParam(paramName) {
	    paramValue = "", isFound = !1;         
	    if (this.location.search.indexOf("?") == 0 && this.location.search.indexOf("=") > 1) {               
	       arrSource = unescape(this.location.search).substring(1, this.location.search.length).split("&"), i = 0;               
	        while (i < arrSource.length && !isFound) arrSource[i].indexOf("=") > 0 && arrSource[i].split("=")[0].toLowerCase() ==  paramName.toLowerCase() && (paramValue = arrSource[i].split("=")[1], isFound = !0), i++  }
	        	
	        return paramValue == "" && (paramValue = null), paramValue 
	}

  var g_itemVO = {}

  jQuery(document).ready(function() {
   			$("#createorder").on("click",function(){

				$.ajax({
					type:"POST",
					contentType:"application/x-www-form-urlencoded",
					url:"http://localhost:8080/order/createorder",
					xhrFields:{withCredentials:true},
					data:{
						"itemId":g_itemVO.id,
						"amount":1,
						"promoId":g_itemVO.promoId
					},
					success:function(data){
						if(data.status =="success"){
							alert("下单成功！");
							window.location.reload();
						}else{
							alert("下单失败！原因为"+data.data.errMsg);
							if(data.data.errCode=="20003")
								window.location.href="login.html";
						}

					},
					error:function(data){
						alert("下单失败！原因为"+data.responseText);
					}
				});

			});

    //获取商品详情
      $.ajax({
        type: "GET",
        url:"http://localhost:8080/item/get",
        data:{
          "id":getParam("id"),
        },
        xhrFields: {withCredentials: true},
        success:function(data) {
          if (data.status == "success") {
            g_itemVO = data.data;
            reloadDom();
            setInterval(reloadDom, 1000);
          } else {
            alert("获取信息失败，原因为" + data.data.errMsg);
          }
        },
        error:function(data) {
          alert("获取信息失败，原因为" + data.responseText);
        }
      });
      return false;
    });

    function reloadDom() {
      $("#title").text(g_itemVO.title);
      $("#description").text(g_itemVO.description);
      $("#stock").text(g_itemVO.stock);
      $("#price").text(g_itemVO.price);
      $("#imgUrl").attr("src",g_itemVO.imgUrl);
      $("#sales").text(g_itemVO.sales);
      if (g_itemVO.promoStatus == 1) {
        //秒杀活动还未开始
        var startTime = g_itemVO.startDate.replace(new RegExp("-", "gm"), "/");
        startTime = (new Date(startTime)).getTime();
        var nowTime = Date.parse(new Date());
        var delta = (startTime - nowTime) / 1000;
        if (delta <= 0) {
          //活动已开始
          g_itemVO.promoStatus = 2;
          reloadDom();
        }
        $("#promoStartDate").text("秒杀活动将于：" + g_itemVO.startDate + "开始！倒计时：" + delta + "秒！");
        $("#promoPrice").text(g_itemVO.promoPrice);
        $("#createorder").attr("disabled", true);
      } else if (g_itemVO.promoStatus == 2) {
        //秒杀正再进行中
        $("#promoStartDate").text("秒杀活动正在进行中！");
        $("#promoPrice").text(g_itemVO.promoPrice);
        $("#createorder").attr("disabled", false);
        $("#normalPriceContainer").hide();
      }
    }
</script>

</html>
```

测试情况如下

![44](seckill项目/44.png)



![45](seckill项目/45.png)

#### OrderModel

```java
//若非空则以秒杀商品方式下单
private Integer promoId;
```

新增promoId，另外数据库的order表也要新增一个字段

![46](seckill项目/46.png)

OrderDO新增

```java
private Integer promoId;

public Integer getPromoId() {
    return promoId;
}

public void setPromoId(Integer promoId) {
    this.promoId = promoId;
}
```

OrderDOMapper.xml新增字段

#### OrderService

接口增加了promoId

```java
package com.seckillproject.service;

import com.seckillproject.error.BusinessException;
import com.seckillproject.service.model.OrderModel;

public interface OrderService {
    //1通过前端传过来promoId,然后下单接口内校验对应id是否属于对应商品且活动已开始
    //2直接在下单接口内判断对应的商品是否存在秒杀活动，若存在进行中的则以秒杀价格下单
    //倾向于第一种，第一个原因因为一个商品可能会存在不同APP的有不同的秒杀活动，所以让前端传promoId
    //第二个原因因为如果在订单接口里还要判断秒杀活动领域模型，相当于任何平销商品也要去判断，性能低
    OrderModel createOrder(Integer userId, Integer itemId, Integer promoId, Integer amount) throws BusinessException;
}
```

orderserviceimpl修改createOrder

```java
@Override
@Transactional
public OrderModel createOrder(Integer userId, Integer itemId, Integer promoId, Integer amount) throws BusinessException {
    //1.校验下单状态：下单商品是否存在、用户是否合法、购买数量是否正确
    ItemModel itemModel = itemService.getItemById(itemId);
    if(itemModel == null) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"商品信息不存在");
    }
    UserModel userModel = userService.getUserById(userId);
    if(userModel == null) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"用户信息不存在");
    }
    if(amount <= 0 || amount > 99) {
        throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"数量信息不正确");
    }
    //校验活动信息
    if(promoId != null) {
        //校验对应活动是否存在这个适用商品
        if(promoId.intValue() != itemModel.getPromoModel().getId()) {
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR, "活动信息不正确");
        }
        //校验活动是否在进行中
        else if(itemModel.getPromoModel().getStatus().intValue() != 2){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"活动还未开始");
        }
    }
    //2.落单减库存
    boolean result = itemService.decreaseStock(itemId, amount);
    if(!result) {
        throw new BusinessException(EmBusinessError.STOCK_NOT_ENOUGH);
    }
    //3.订单入库
    OrderModel orderModel = new OrderModel();
    orderModel.setUserId(userId);
    orderModel.setItemId(itemId);
    orderModel.setPromoId(promoId);
    orderModel.setAmount(amount);
    if(promoId != null) {
        orderModel.setItemPrice(itemModel.getPromoModel().getPromoItemPrice());
    } else {
        orderModel.setItemPrice(itemModel.getPrice());
    }
    orderModel.setOrderPrice(orderModel.getItemPrice().multiply(new BigDecimal(amount)));
    //生成交易流水号也就是订单号
    orderModel.setId(generateOrderNo());
    OrderDO orderDO = convertFromOrderModel(orderModel);
    orderDOMapper.insertSelective(orderDO);

    //增加商品的销量
    itemService.increaseSales(itemId,amount);
    //4.返回前端
    return orderModel;
}
```

经过下单测试后，发现order_info表里有了一条订单记录，活动id=1，商品价格也为活动价格

![47](seckill项目/47.png)