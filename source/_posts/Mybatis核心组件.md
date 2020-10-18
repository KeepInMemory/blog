---
title: Mybatis核心组件
date: 2020-08-22 19:12:18
tags:
  - Mybatis
---

SqlSessionFactoryBuilder：根据配置文件生成的Configuration对象生成SqlSessionFactory

SqlSessionFactory：用于创建SqlSession的工厂类

SqlSession：MyBatis核心组件，用于向数据库执行SQL

Mapper接口：就是DAO接口

Mapper映射器：用于编写SQL，并将SQL和实体类映射，采用XML和注解均可实现

![1](Mybatis核心组件/1.png)

1.创建SqlSessionFactoryBuilder对象，调用build(inputstream)方法读取并解析配置文件（Mybatis-config.xml），返回SqlSessionFactory对象。SqlSessionFactory里有Configuration对象

```java
public class DefaultSqlSessionFactory implements SqlSessionFactory {
    private final Configuration configuration;
```

解析配置文件信息的本质就是创建Configuration对象，将解析的xml数据封装到Configuration内部的属性中。Configuration对象的结构和xml配置文件的对象几乎相同。里面有properties（属性），settings（设置）mappers（映射器）等对象。

```java
public class Configuration {
    protected final MapperRegistry mapperRegistry;
    protected final Map<String, MappedStatement> mappedStatements;
   	//...
```

当解析mapper.xml的mappers标签时，会再将对应配置文件中的增删改查标签一 一封装成MappedStatement对象，MappedStatement与Mapper配置文件中的一个SQL对应，主要用途是描述一条SQL语句，它会被存入Configuration的mappedStatements中。mappedStatements 是一个HashMap，存储时key = namespace + id，value = 对应的MappedStatement对象。

当判断解析到接口时，会创建此接口对应的MapperProxyFactory对象，存入mapperRegistry的knownMappers中，key = 接口的Class对象，value = 此接口对应的MapperProxyFactory对象。

2.通过SqlSessionFactory构造SqlSession
SqlSession中的两个最重要的参数，configuration与初始化时的相同，Executor为执行器。Executor也是一个接口，他有三个常用的实现类BatchExecutor（重用语句并执行批量更新），ReuseExecutor（重用预处理语句prepared statements），SimpleExecutor（普通的执行器，默认）。

3.执行SqlSession.selectOne(namespace+id,参数)方法时，会获取相应的MappedStatement，交给executor去执行底层的jdbc代码
执行SqlSession.getMapper(Mapper接口的字节码对象)方法是，会去获取Configuration对象的MapperRegistry的Map<Class<?>, MapperProxyFactory<?>> knownMappers，拿到对应的MapperProxyFactory动态代理工厂，去生成相应的MapperProxy实例返回，使用了JDK动态代理的MapperProxy实现了InvocationHandler接口，重写了invoke方法。getMapper获取到的实际上是MapperProxy，调用了里面的代理方法，代理方法增加了jdbc的代码。

Mapper接口对应的Sql语句的获取是通过将SqlSession一步步传入，sqlSession的Configuration里有，在底层进行获取

Note：

SSM里Mapper.xml可以不写，接口必须要写，接口如果加@Insert@Select就不写mapper.xml，否则就要写

Spring Boot为什么要写@Mapper+接口？因为要生成MapperProxyFactory生成的MapperProxy代理对象，JDK动态代理需要实现接口，调用mapper对象的方法实际上调用的是接口的方法

Mybatis的实现方式有两种，写Mapper接口+xml，写Mapper接口+@Insert，两个都有Mapper接口，要生成代理对象