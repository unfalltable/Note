---
title: MyBatis源码
categories: 源码
tags: [源码]
---

# 架构

![image-20230317103336946](../pic/image-20230317103336946.png)

- 接口层

  - 提供操作数据的Api接口，完成交互

  - 提供两种调用方式
    - 基于statementId，需要传递xml中查询的id和参数
      - 基于mapper接口，生成代理对象调用方法，只需要传参数即可，本质上还是基于statementId的

- 数据处理层
  - 核心层，主要完成对数据的处理工作
    - 参数映射（ParameterHandler）、sql解析（sqlSource）、sql执行（Executor）、结果处理和映射封装（ResultSetHandler）

- 框架支撑层
  - 为基础服务提供支持，负责通用服务支持
    - sql配置方式：xml，注解
    - 事务管理、连接池管理、缓存管理

- 引导层
  - 提供两种配置方式，获取mybatis启动需要的配置信息
    - 基于xml
    - 基于JavaAPI

# 组件及其调用关系

![image-20230317115152145](../pic/image-20230317115152145.png)

# Mybatis

## 执行流程

1. 读取配置文件信息，加载sql映射文件
   - 通过类加载器对配置文件进行加载，加载成了字节输入流，存到内存中，这一步中配置文件还没有被解析
   - 内部使用了ClassLoaderWrapper，它是 ClassLoader 的装饰器，封装了多个 ClassLoader，方便统一使用
2. 构建SqlSessionFactory
   - 解析xml配置文件 将inputStream转换成Document对象，创建Configuration
   - 使用构建者模式构建复杂的Configuration对象中的成员变量（配置信息和Mapper信息）
   - 创建SqlSessionFactory对象
   - XPathParser是一个基于Java XPath的解析器，将传入的inputStream 转换成一个Document对象
   - 创建Configuration对象，完成类型别名注册
3. 创建对应的SqlSession
4. 通过Executor执行器操作数据库，维护缓存
5. 读取MapperStatement对象

## 二级缓存

- 提升数据的检索效率
- 一级缓存为本地缓存（SqlSession）
  - SqlSession中有一个Excutor，Excutor中有一个localcache
  - 当用户发起查询时，mybatis会去localcache中查
  - 分布式环境下可能会出现一个脏读的问题

- 二级缓存为跨SqlSession的缓存
  - 即多个用户查询时，只要有一个SqlSession查询到数据后，就会加入二级缓存，其他SqlSession就可以直接从二级缓存中取数据
  - 在Excutor上做了一个装饰器（CachingExcutor），查询一级缓存前会先查这个
  - 二级缓存实现了多个SqlSession之间的共享，是一个全局的缓存，缓存粒度在namespace级别，可以通过Cache接口实现不同缓存实现类组合
  - 原理就是在每次查询时生成一个CacheKey对象，然后去二级缓存中找这个CacheKey对象是否有对应的数据
    - 没有就查数据库，然后加入二级缓存
    - 有的话直接返回
  - 创建CacheKey对象的方法中已经重写了equals和hashcode方法，所以其比较的是hash值而不是地址值
  - CacheKey由6部分构成的，为statementId、分页参数（偏移、条数）、sql、参数的值、环境变量，它们各自生成的hash组成的一个集合

# MybatisPlus