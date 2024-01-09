---
title: GraphQL
categories: 模块
tags: [模块]
---

## 简介

- Restful存在的弊端
  - 想要部分数据，但是返回的有多余的数据，资源浪费
  - 一次请求不能满足需求
- 按需索取资源、一次查询多个数据、无需划分版本

## 数据类型

- Int：有符号32位

- Float

- String

- Boolean

- ID：唯一标识符

- 自定义类型

- 枚举

  ```yaml
  enum Episode {
  	A
  	B
  	C
  }
  ```

- 接口

  ```yaml
  interface Character	{ #定义接口
  	id: ID! #!表示非空
  	name: String
  	age: Int
  }
  type Human implements Character{
  	id: ID! #!表示非空
  	name: String
  	age: Int
  	#其他的属性
  } 
  ```