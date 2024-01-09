---
title: SQL
categories: 数据库
tags: [数据库]
---

## 简介

## 概念

### 外键

### 编码

utf8和utf8mb4的区别

- utf8mb4是utf8的超集,专门用来兼容四字节的
- 兼容emojo表情和不常用的中文汉字，utf8不兼容

## 操作符

| 操作符      | 作用                         | 备注 |
| ----------- | ---------------------------- | ---- |
| IN / NOT IN | 等于/不等于 列表中的任意一个 |      |
| ANY / SOME  | 和子查询返回的某一个值比较   |      |
| ALL         | 和子查询返回的所有值比较     |      |
| \|\|        | 拼接字段                     |      |

## 函数

| 函数名                                        | 作用                     | 参数                             | 备注                       |
| --------------------------------------------- | ------------------------ | -------------------------------- | -------------------------- |
| count                                         | 统计记录数               | */字段/常量                      |                            |
| sum                                           | 求和                     |                                  |                            |
| avg                                           | 平均值                   |                                  |                            |
| max                                           | 最大值                   |                                  |                            |
| min                                           | 最小值                   |                                  |                            |
| cast                                          |                          | 值 as 类型                       |                            |
| with as 名                                    | 临时表                   | 子查询                           | 8.0之前要用temporary table |
| DATE_FORMAT                                   |                          | 时间，格式                       |                            |
| CONCAT                                        | 连接两个字符             | A，B                             |                            |
| IFNULL                                        | 判断是否为空             | 可能为null的值，如果为空返回的值 | 只传一个参数相当于if       |
| LENGTH                                        | 获取长度                 | 字符串                           |                            |
| UPPER                                         | 大写                     |                                  |                            |
| LOWER                                         | 小写                     |                                  |                            |
| substr                                        | 截取字符串               | 字符串，开始索引，长度           |                            |
| instr                                         | 返回B在A第一次出现的索引 | strA,strB                        |                            |
| trim                                          | 去除前后字符             | 去除的字符，str                  | 只传str则去除空格          |
| lpad                                          | 左填充字符               | str,结果长度,填充字符            |                            |
| rpad                                          | 右填充字符               | str,结果长度,填充字符            |                            |
| replace                                       | str中的A换成B            | str,strA,strB                    |                            |
| round                                         | 四舍五入                 | 数值，保留位数                   |                            |
| ceil                                          | 向上取整                 | 数值                             |                            |
| floor                                         | 向下取整                 | 数值                             |                            |
| truncate                                      | 截断                     | 数值，保留位数                   |                            |
| mod                                           | 取余                     | 数值，除数                       |                            |
| rand                                          | 获取随机数               |                                  |                            |
| now                                           | 当前系统日期和时间       |                                  |                            |
| curdate                                       | 当前系统日期             |                                  |                            |
| curtime                                       | 当前系统时间             |                                  |                            |
| YEAR/MONTH/DAY                                | 获取指定的部分           |                                  |                            |
| STR_TO_DATE                                   | 日期转日期               | 时间，格式                       |                            |
| DATE_FORMAT                                   | 日期转字符               | 时间，格式                       |                            |
| DATEDIFF                                      | 俩个时间的之间的天数     |                                  |                            |
| if                                            | 判断                     | 判断,true返回的值,false返回的值  |                            |
| listagg() within group(order by 字段) as 别名 | 行转列                   | 字段，分隔符                     | Oracle                     |

## 流程控制

If

- IF(判断,true返回的值,false返回的值)

- if 条件1 then 语句1;

  elseif 条件2 then 语句2;

  【else 语句n;】

  end if;

case

- case(要判断的值)

  when 常量1 then 要显示的值或语句;

​	   when 常量2 then 要显示的值或语句;

​	   【ELSE 以上都不符合要显示的值或语句;】

​	   END

- case

  when 判断1 then 要显示的值或语句;

  when 判断2 then 要显示的值或语句;

  【ELSE 以上都不符合要显示的值或语句;】

  END

Having

- 对分组筛选后的数据进行操作，用在GROUP BY 后，效果同where
- GROUP BY 和 HAVING 后都可接别名
- 筛选条件能写在where后优先现在where后，效率高
- 多个字段分组用逗号隔开即可

## 约束

| 约束        | 作用   | 备注                     |
| ----------- | ------ | ------------------------ |
| not null    | 非空   | 字段不能为空             |
| default     | 默认值 | 字段有默认值             |
| primary key | 主键   | 字段具有唯一性，非空     |
| unique      | 唯一   | 字段具有唯一性，可以为空 |
| check       | 检查   | mysql中不支持            |
| foreign key | 外键   | 不建议使用               |

## 索引

### 索引结构

- 不同的存储引擎实现的索引结构不同
  - **B+Tree索引**：大部分引擎都支持
  - **Hash索引**：**Memory引擎支持**
    - 索引速度快，效率高，但不支持范围索引
    - 无法利用索引排序
  - R-tree(空间索引)：**MyISAM引擎支持**
    - 一个特殊索引类型，主要用于地理空间类型
  - full-text(全文索引)：**elasticSearch**，**InnoDB**(5.6之后)、**MyISAM**支持

### 索引分类

|      | 主键索引   | 唯一索引 | 常规索引 | 全文索引                 |
| ---- | ---------- | -------- | -------- | ------------------------ |
|      | 只能有一个 | 可以多个 | 可以多个 | 可以多个                 |
|      | 不能null   | 可null   | 可null   | 可null                   |
|      | 值唯一     | 值唯一   | 值不唯一 | 值唯一                   |
|      |            |          |          | 只能在字符串、文本上建立 |


### InnoDB中的索引

- **聚集索引**
  - 必须有，而且只有一个，一般是主键索引，没有就选唯一索引，再没有InnoDB会自动生成一个rowid作为隐藏的聚集索引
  - 索引的叶子结点保存了行数据
- **二级索引**
  - 可以有多个
  - 索引的叶子结点存放的是对应的字段值

### 索引语法

- 创建索引
  - `create index 索引名 on 表名(字段名)` 
  - `alter table 表名 add index 索引名(字段)` 
- 查看索引
  - `show index from 索引名`
- 删除索引
  - `drop index 字段名 on 表名`
  - `alter table 表名 drop index 索引名` 
- 查看表的索引情况
  - `show index from 表名` 
  - `show create table 表名` 
- 修改索引
  - 只能通过删除索引再添加索引

## DQL(数据查询语言--Data Query Language)

### 基础查询

`select 查询列表 from 表名`

- 起别名
  - 使用as
  - 使用空格
  - 字符串用" "

- 去重
  - DISTINCT
- +号的作用
  - 运算符
    - 俩个操作数都为数值型，则进行加法运算
    - 其中一方为字符型
      - 字符为数字则进行计算
        - "100" + 50 -> 150
      - 为字符则字符数值为0
        - "John" + 100 -> 100
    - 其中一方为null则都为null
      - null + 50 -> null

### 条件查询

`select 查询列表 from 表名 where 筛选条件 `

- 筛选条件
  - 条件表达式
    - \> \< \= \!= \>= \<=   \<>不等于
  - 逻辑表达式
    - \&& \|| \!
    - and or not 
  - 模糊查询
    - like
      - %任意多个字符，包含0个
      - _任意当个字符
      - \转义字符   也可以使用ESCAPE 转义
        - ESCAPE '$'  ==    \
      - 无法显示null值
    - between and 
      - 包含临界值
    - in(列表1，列表2)
      - 判断值是否属于in列表中
      - 不能使用通配符
    - is null
      - <=>   安全等于,可以搭配null

### 排序查询

`select 查询列表 from 表名 where 筛选条件 order by 排序列表 [asc默认升序/desc降序] `

#### 注意

- order by 后可以用别名
- order by 要放在语句的最后
- 支持多个字段，函数，表达式，别名

### 连接查询

| 连接方式            | 结果                                 | 备注                         |
| ------------------- | ------------------------------------ | ---------------------------- |
| 表1，表1            | 笛卡尔积                             | 自连接，处理自身层级关系数据 |
| 表1，表2            | 表1表2交集                           | SQL92                        |
| 表1 inner join 表2  | 表1表2交集                           | SQL99                        |
| 表1 Join 表2        |                                      |                              |
| 表1 left Join 表2   | 表1为主表拼接表2                     |                              |
| 表1 right Join 表2  | 表2为主表拼接表1                     |                              |
| 表1 full Join 表2   | 交集 + 表1有表2没有的+表2有表1没有的 |                              |
| 表1 cross Join 表2  | 笛卡尔积                             |                              |
| 子查询              |                                      | 查询中嵌套查询               |
| 查询 union 查询     | 合并为一个结果集                     |                              |
| 查询 union all 查询 | 合并为一个结果集                     |                              |

### 子查询

- 出现在其他语句内部的select语句称为子查询或内查询

- 内部有子查询的语句为主查询

#### 分类

##### 按位置

- SELECT后
  - 结果集只能为标量子查询（一行一列）
- FROM后
  - 将子查询的结果充当一张表，必须起别名
- WHERE或HAVING后
  - 标量子查询（单行子查询）
    - 子查询返回的值是一行一列的
  - 列子查询（多行子查询）
    - 子查询返回的值是一列多行的
    - IN / ANY / SOME / ALL
  - 行子查询（一行多列）
    - 子查询返回的值是一行多列的
    - (A,B) = (select语句)
- EXISTS后(相关子查询)
  - EXISTS（完整的查询语句）
    - 有结果返回1，没结果返回0

##### 按结果集行列数

- 标量子查询（一行一列）
- 列子查询（一列多行）
- 行子查询（一行多列）
- 表子查询（多行多列）

#### 特点

- 子查询的执行优先于主查询

### 分页查询

`SQL 语句 limit offset,size `

- offset：起始索引，索引从0开始
- size：要显示的个数

#### 注意

- limit最后执行
- 索引从0开始可以省略
- page 显示的页数  size每一页显示的条目
  - limit (page-1)*size,size  

### 联合查询

`语句A union 语句B union...`

- 将多条查询语句的结果合并成一个表

#### 注意

- 要求多条查询语句查询的字段个数要一致 
- 要求多条查询语句查询的每一个字段的类型和顺序最好一致
- 默认情况会自动去重，不想去重可以使用union ALL

## DML(数据操纵语言--Data Manipulation Language)

`insert into 表名(字段名...) values(值...);`

- 插入的值的类型要与字段的类型一致或兼容
- 可以为空的字段可以不写也可以写null
- 可以省略字段名，但所有字段都得有值
- 支持多行插入，用逗号分隔
- 支持子查询

`insert into 表名 set 字段名=值...`

- 不支持插入多行
- 不支持子查询

`update 表名 set 字段名=新值,字段名=新值... where 筛选条件`

- 单表修改

`update 表1 别名 inner/left/right join 表2 别名 on 连接条件 set 字段名=值。。。 where 筛选条件`

- 多表修改

`delete from 表名 where 筛选条件`

`delete 要删除的表的别名 from 表1 别名 inner/left/right join 表2 别名 on 连接条件 where 筛选条件`

- 删除后自增长列从删除前断点开始
- 有返回值（能显示删除了几条）
- 删除后能回滚

`truncate table 表名`

- 只能删除整张表，不能使用where
- 删除后自增长列从1开始
- 没返回值（不能显示删除了几条）
- 删除后不能回滚

## DDL(数据定义语言--Data Define Language)

### 库的管理

`create database (if no exists) 库名`

- 创建库

`alter database 库名 character set 编码格式`

- 修改库

`drop database (if is exists) 库名`

- 删除库

### 表的管理

`desc 表名`

- 查看表结构

`create table 表名(`

​					`字段名 字段类型[(长度)] [约束],`

​					`字段名 字段类型[(长度)] [约束],`

​					`...) `

- 创建表

`alter table 表名 change column 旧字段名 新字段名 新字段类型`

- 修改字段名

`alter table 表名 modify column 字段名 新字段类型`

- 修改字段的类型

`alter table 表名 add column 新字段名 新字段类型`

- 添加字段

`alter table 表名 drop column 字段名`

- 删除字段

`alter table 表名 rename to 新表名`

- 修改表名

`alter table 表名 modify column 字段名 字段类型 新约束`

- 添加列级约束

`alter table 表名 add [constraint 约束名] 约束类型(字段名) [外键的引用]  `

- 添加表级约束

`drop table (if is exists) 表名`

- 删除表

`create table 复制表 like 原始表`

- 复制表的结构

`create table 复制表 select 字段1，字段2 from 原始表 where 0`

- 复制表的部分结构

`create table 复制表 select * from 原始表`

- 复制表的结构和所有数据

`create table 复制表 select 字段1，字段2 from 原始表 where 筛选条件`

- 复制表的结构和部分数据

