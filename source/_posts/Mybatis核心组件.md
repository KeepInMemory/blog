---
title: Mybatis核心组件
date: 2020-08-22 19:12:18
tags:
---

SqlSessionFactoryBuilder：根据配置文件生成的Configuration对象生成SqlSessionFactory

SqlSessionFactory：用于创建SqlSession的工厂类

SqlSession：MyBatis核心组件，用于向数据库执行SQL

Mapper接口：就是DAO接口

Mapper映射器：用于编写SQL，并将SQL和实体类映射，采用XML和注解均可实现

![1](Mybatis核心组件/1.png)