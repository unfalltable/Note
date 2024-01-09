---
title: Maven
categories: Java基础
tags: [基础]
---

## 命令 

| 命令        | 参数 | 作用           |
| ----------- | ---- | -------------- |
| mvn clean   |      | 清除target     |
| mvn compile |      | 编译           |
| mvn test    |      | 单元测试       |
| mvn package |      | 打包           |
| mvn install |      | 打包到本地仓库 |
| mvn deploy  |      | 打包到远程仓库 |

## 关键字

**dependencyManagement**

- 只是声明依赖，并不引入，子类需要声明引用

**scope**

- provided 目标环境已存在，不用打包

**properties**

- 版本管理
