---
title: Redis
categories: 数据库
tags: [中间件]
---

## 简介

- Redis是基于AP思想的，是最终一致性
- 数据全部存储在内存中，操作效率高，速度快
- 单线程处理，避免了不必要的上下文切换，不用考虑线程安全问题
- IO多路复用技术，可以处理高并发的连接请求
- 事件派发机制

##  数据类型

### String

| 功能                                    | 命令                                                |
| --------------------------------------- | --------------------------------------------------- |
| 添加/修改数据                           | set key value<br />mset key1 value1 key2 value2 ... |
| 获取数据                                | get key<br />mget key1 key2...                      |
| 删除数据                                | del key                                             |
| 获取字符串长度                          | strlen key                                          |
| 追加消息                                | append key value                                    |
| 增加                                    | incr key<br />inceby / incrbyfloat   key 数值       |
| 减少                                    | decr key<br />decrby key 数值                       |
| 设置有效期                              | setex key seconds 值<br />psetex key millseconds 值 |
| 设置一个值，当redis中没有时才能设置成功 | setnx key 值                                        |

- 数值超过上限会报错
- 0 表示失败，1表示成功
- nil 表示为null
- 数据最大存512MB

### Hash

| 功能                                        | 命令                                                |
| ------------------------------------------- | --------------------------------------------------- |
| 添加 / 修改数据                             | hset key field value                                |
| 获取数据                                    | hget key field<br />hgetall key                     |
| 删除数据                                    | hdel key field                                      |
| 添加 / 修改多个数据                         | hmset key field1 value1...                          |
| 获取多个数据                                | hmget key field1 field2...                          |
| 获取key中字段的数量                         | hlen key                                            |
| 获取key中是否存在指定的字段                 | hexists key field                                   |
| 获取key中所有的字段名或字段值               | hkeys key<br />hvals key                            |
| 设置指定字段的数值数据增加指定范围的值      | hincrby key field 值<br />hincrbyfloat key field 值 |
| 获取一个key中所有的field-value              | hgetall key                                         |
| 添加一个key中的field，前提是这个field不存在 | hsetnx key field 值                                 |

- 一个key对应多个field - value
- 如果filed少，会优化为类数组
- 如果field多，会优化为HashMap
- value只能存字符串
- 一个key可以存2^32^ - 1 个键值对
- hash可以存储少量对象

### List

| 功能                             | 命令                 |
| -------------------------------- | -------------------- |
| 左侧插入元素                     | LPUSH key 值         |
| 移除左侧第一个元素               | LPOP key             |
| 右侧插入元素                     | RPUSH key 值         |
| 移除右侧第一个元素               | RPOP key             |
| 返回一段范围内的所有元素         | LRANGE key start end |
| 阻塞取元素，没元素会等待一段时间 | BLPOP / BRPOP key    |

- 可以看成双向链表

### Set

| 操作               | 命令             |
| ------------------ | ---------------- |
| 添加元素           | Sadd key 值      |
| 移除元素           | Sred key 值      |
| 返回元素个数       | Scard key        |
| 判断元素是否存在   | Sismember key 值 |
| 获取所有元素       | Smembers         |
| 求key1和key2的交集 | Sinter key1 key2 |
| 求key1和key2的差集 | Sdiff key1 key2  |
| 求key1和key2的并集 | Sunion key1 key2 |

- 类似hashSet

### SortedSet

| 操作                       | 命令                      |
| -------------------------- | ------------------------- |
| 添加                       | Zadd key score 成员       |
| 删除                       | Zrem key 成员             |
| 获取分值                   | Zscore key 成员           |
| 获取排名                   | Zrank key 成员            |
| 获取元素个数               | Zcard key                 |
| 统计范围内个数             | Zcount key min max        |
| 指定元素自增               | Zincrby key 自增的值 成员 |
| 排序后，取指定排名范围的值 | Zrange key min max        |
| 排序后，取指定分数范围的值 | ZrangeByScore key min max |
| 差集 / 交集/ 并集          | Zdiff / Zinter / Zuinon   |

- 排序默认是升序的，Zrev为降序
- 底层数据结构是跳表

### Stream消息队列

#### 消费者

- 添加信息
  - `Xadd key [nomkstream] [maxlen | minId [=|~] threshold [limit count]] *|ID field value...` 
    - [nomkstream]：队列不存在则创建
    - [maxlen | minId [=|~] threshold [limit count]]：设置队列的最大消息数量
    - *|ID：消息唯一id，\*代表自动生成
    - field value：要发送的消息
- 读取消息
  - `Xread [Count count] [block milliseconds] streams key... Id...`
    - [Count count]：每次读取的数量
    - [block milliseconds]：没有消息时的阻塞时长
    - streams key：队列名
    - Id：起始id，0代表从第一个消息开始，$代表从最新的消息开始

#### 消费者组

- 创建消费者组
  - `Xgroup create key groupName Id [mkstream]`
    - key：队列名
    - groupName：消费者组名
    - Id：$代表队列中最后一个消息，0代表第一个消息
    - mkstream：队列不存在则创建
- 读取消息
  - `XreadGroup Group group consumer [Count count] [block milliseconds] [NoAck] streams key... Id...`
    - group：消费者组名
    - consumer ：消费者名
    - [Count count]：每次读取的数量
    - block milliseconds：没有消息时的阻塞时长
    - NoAck：无需手动Ack，自动确认
    - key：队列名
    - Id：获取信息的起始ID
      - “ >”：从下一个未消费的信息开始
      - 下标：从pending-list下标位置开始读取
- 查看pending-list
  - `  Xpending key group id范围 count`

### GEO

| 作用                                                        | 命令                |
| ----------------------------------------------------------- | ------------------- |
| 添加地理空间信息                                            | GEOadd 经度 纬度 值 |
| 计算两点之间距离                                            | GEOdist 点1 点2     |
| 指定点的坐标转换为hash字符串                                | GEOhash             |
| 返回点的坐标                                                | GEOpos              |
| 指定圆心，半径，找到园内点，按照离圆心的距离排序（6.2废弃） | GEOradius 圆心 半径 |
| 指定范围内找点，可以是圆型、矩形，会排序（6.2新增）         | GEOsearch           |
| 与GEOsearch功能一样，但可以把结果存入一个key                | GEOsearchStore      |

- 底层是SortedSet

### BitMap

| 作用                    | 命令        |
| ----------------------- | ----------- |
| 指定位置插入0或1        | setBIT      |
| 获取指定位置值          | getBIT      |
| 统计1的数量             | BITcount    |
| 查、改、增 指定位置的值 | BITfield    |
| 获取数组                | BITfield_ro |
| 结果做位运算            | BITop       |
| 找第一个1 或0 的位置    | BITpos      |

### HyperLogLog

- 用于确定非常大的集合的基数，不需要存储其所有值
- 基于String实现的，当个HLL内存小于16kb
- 测量结果有小于0.81%的误差

## 线程模型

### 简介 

- 内部使用文件事件处理器（file event handler），是单线程的
- 使用的IO多路复用机制同时监听多个socket，将产生事件的socket压入内存队列
- 事件分派器根据socket上的事件类型来选择对应的事件处理器
- 在Redis6中引入了多线程机制，主要用于处理回复事件、命令转换

### 文件事件处理器（file event handler）

![Redis-single-thread-model](https://doocs.github.io/advanced-java/docs/high-concurrency/images/redis-single-thread-model.png)

1. 首先，Redis 服务端进程初始化的时候，会将 server socket 的 `AE_READABLE` 事件与连接应答处理器关联。

2. 客户端 socket01 向 Redis 进程的 server socket 请求建立连接，此时 server socket 会产生一个 `AE_READABLE` 事件，IO 多路复用程序监听到 server socket 产生的事件后，将该 socket 压入队列中。文件事件分派器从队列中获取 socket，交给**连接应答处理器**。连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 `AE_READABLE` 事件与命令请求处理器关联。

3. 假设此时客户端发送了一个 `set key value` 请求，此时 Redis 中的 socket01 会产生 `AE_READABLE` 事件，IO 多路复用程序将 socket01 压入队列，此时事件分派器从队列中获取到 socket01 产生的 `AE_READABLE` 事件，由于前面 socket01 的 `AE_READABLE` 事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。命令请求处理器读取 socket01 	的 `key value` 并在自己内存中完成 `key value` 的设置。操作完成后，它会将 socket01 的 `AE_WRITABLE` 事件与命令回复处理器关联。

4. 如果此时客户端准备好接收返回结果了，那么 Redis 中的 socket01 会产生一个 `AE_WRITABLE` 事件，同样压入队列中，事件分派器找到相关联的命令回复处理器，由命令回复处理器对 socket01 输入本次操作的一个结果，比如 `ok` ，之后解除 socket01 的 `AE_WRITABLE` 事件与命令回复处理器的关联。这样便完成了一次通信。

## 客户端

### SpringDataRedis序列化的方式

- ​	自定义redisTemplate，修改序列化器

  ```java
  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
      //创建Template
      RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
      //设置连接工厂
      redisTemplate.setConnectionFactory(redisConnectionFactory);
      //设置序列化工具
      GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
      //key和HashKey采用String序列化
      redisTemplate.setKeySerializer(ReidsSerializer.string());
      redisTemplate.setHashKeySerializer(ReidsSerializer.string());
      //value和HashValue采用Json序列化
      redisTemplate.setValueSerializer(jsonRedisSerializer);
      redisTemplate.setHashValueSerializer(jsonRedisSerializer);
      return redisTemplate;
  }
  ```

- 使用StringRedisTemplate，手动序列化Json

## 预热、雪崩、穿透、击穿

### 缓存预热

- 主从之间数据吞吐量较大、数据同步操作频度高、请求数量较高，而且redis中没有数据，会导致redis启动时快速宕机，所以需要进行redis缓存预热
- 日常需要对数据的访问进行记录，统计热点数据
- 利用LRU数据删除策略，构建数据留存队列
- redis启动时加载热点数据
  - 使用脚本固定触发数据预热

### 缓存穿透

- 缓存穿透是指客户端请求的数据在缓存和数据库中的不存在，这些请求每次都会打到数据库

- 解决方案

  - 使用缓存空对象，设置过期时间，可能存在短期不一致性

  - 使用布隆过滤器解决，在客户和缓存之间添加，缓存预热时，预热布隆过滤器

  - 增强id的复杂度

  - 加强用户权限校验

  - 做好热点参数的限流


### 缓存雪崩

- 同一时段大量的缓存key同时失效或者redis宕机，导致大量的请求直接到达数据库，带来巨大压力

- 解决方案：

  - 给不同的缓存key的过期时间（TTL）添加随机值
    - 不同业务过期时间不同
  
    - 对超级热点数据使用永久key
  
  - 使用Redis集群，主从+哨兵
  - 给缓存业务添加降级限流策略
  - 给业务添加多级缓存
  - 优化数据库中严重耗时的业务
  - redis监控服务器性能指标
  - 把缓存key到期删除策略改成按命中次数删除策略
  - redis开启持久化，宕机后快速恢复

### 缓存击穿

- 一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求会打到数据库

- 解决方案
  - 如果缓存的数据基本不更新，可以设置为不过期
  
  - 互斥锁
    - 适合缓存更新较快的场景
    - 没有额外内存消耗，保证一致性，实现简单，但是性能受影响，可能出现死锁
    - 利用Redis的 SETNX 实现，只有第一个执行SETNX的成功	
    - 或者使用redisson实现
  
  - 逻辑过期
    - 即热点数据缓存不设置过期时间，而是在其对象中设置一个时间属性，在这上面设置过期时间
      - 性能较好，但不保证一致性，有额外内存消耗，实现复杂
  


## 消息队列

|              |        List        |       Pubsub       |                   Stream                   |
| ------------ | :----------------: | :----------------: | :----------------------------------------: |
| 持久化       |         √          |         ×          |                     √                      |
| 阻塞读取     |         √          |         √          |                     √                      |
| 消息堆积处理 | 可利用多消费者处理 | 受限于消费者缓冲区 | 受限于队列长度，可使用消费者组加快消费速度 |
| 消息确认     |         ×          |         ×          |                     √                      |
| 消息回溯     |         ×          |         ×          |                     √                      |

## 内存淘汰策略

- noeviction：不清除缓存，内存满了不允许写入新的数据（默认）
- allkeys-lru：对所有key使用LRU算法进行删除 （推荐）
- volat-lru：对所有设置了过期时间的key使用LRU删除
- allkeys-random：对所有key随机删除
- volatile-random：对所有设置了过期时间的key随即删除
- volatile-ttl：删除马上就要 过期的key
- allkeys-lfu：对所有key使用LFU算法进行删除（根据数据访问频率）
- volatile-lfu：对所有设置了过期时间的key使用LFU删除

## 数据过期策略

1. 惰性删除
   - 过期后不删除，使用到key时会检查他是否过期，过期删除
2. 定期删除
   - 每隔一段时间，对一定量的key进行检查，过期就删除
   - 分为slow模式和fast模式
     - slow：定时任务
     - fast：间隔2ms，耗时不超过1ms

## 应用场景

- 短信验证
  - 手机号作为key，验证码为value
  - 登录成功获取token，token为key，user对象为值
  - 设置有效期

- 数据缓存
  - 分布式事务保证原子性
- 全局ID
  - incrby方法，是原子性操作
  - 符号位(1) + 时间戳(31) + 序列号(32)
  - 雪花算法
    - 符号位 + 时间戳 + 机器码 + 序列号
- 分布式锁
  - setnx方法，超时释放，分布式id冲突
  - 释放锁时需要判断value值是否一致，保证原子性
  - Redission实现
- 消息队列
  - XGroup，消息分流、表示、确认、阻塞读取、可回溯
  - 使用Lua脚本添加信息，保证原子性
  - 消息发送后进入pending-list，XACK后移除
- 点赞
  - SortedSet
- 关注
  - Set，查看共同关注
- 推送
  - 有推，拉，推拉结合
  - 为用户建立收件箱，实现分页查询，SortedSet实现
- 附近商铺
  - GeoLocation，底层是SortedSet
- 签到
  - BitMap，String实现

## 事务

- 将多个命令打包，然后一次性、有序的执行
- 多个命令会被列入到事务队列中，然后按先进先出FIFO的顺序执行
- 事务不会被中断
- 使用场景

## 缓存一致性

- 做法
  - 读的时候先读缓存，缓存没有就读数据库，然后存入缓存
  - 先更新数据库再删除缓存 或者 先删除缓存再更新数据库
    - 删除缓存时因为如果每次更新数据库都更新缓存，无效的写操作太多
- 可能出现的问题
  1. 删除缓存时失败了，导致缓存中是旧数据
     - 延时双删，即在一定时间后再删除一次
     - 可以使用消息队列完成删除的动作
  2. 更新数据库时有读请求请求到旧数据
     - 当读取缓存时没取到数据，就将请求存入MQ中，然后等待数据库更新完成存入MQ去删除缓存
     - 可以使用Canal中间件实现无侵入性的异步删除缓存
     - 加读写锁（强一致）
  3. 更新数据库时有读请求请求到旧数据并加入缓存
     - 加读写锁（强一致）

## 压缩列表

- 设置字符压缩阈值
  - 阈值最好不要大于2048
    - 因为压缩列表涉及解码编码
- 压缩指针
  - 三个指针的存储
- 大列表拆分
  - lua脚本实现
- 存数据索引


## 持久化

### RDB

- Redis数据备份文件，也被叫做Redis数据快照
- RDB把内存中的所有数据记录到磁盘中，当Redis故障重启后，从磁盘读取RDB，RDB读取较快
- Redis停机时会保存一次RDB，也可以设置单位时间修改量阈值，超过阈值就保存
- redis.conf文件中可以对RDB进行配置
- RDB执行间隔长，俩次RDB之间写入数据有丢失的风险
- 可以使用主进程执行或者子进程执行
  - save：主进程，会阻塞
    - 操作虚拟内存，通过页表映射到物理内存

  - bgsave：获得子进程，往磁盘写RDB都比较耗时，异步非阻塞


#### 存储的流程

- 由于Linux系统中所有的进程都无法直接操作物理内存，都是通过操作进程自身的虚拟内存，然后通过页表建立虚拟内存和物理内存之间的映射关系实现
- 克隆一个与主进程一直的子进程，共享内存空间，子线程克隆主进程中的页表，速度快
- 读取物理内存数据写入新的RDB文件并存入磁盘
- 用新RDB文件替换旧的
- 如果子进程在写RDB时主进程有写入的操作，此时会将物理内存拷贝一份，新的读写请求都会操作这个拷贝的物理内存，这样可以避免数据不一致

### AOF

- Redis追加文件，redis的每一个写命令都会记录在aof中
- aof写入性能高，使用的是追加写入，没有磁盘寻址开销，文件不易破损
- 默认是关闭的，需要在配置文件中开启，也可以配置相关配置
  - Always：同步刷盘，可靠性高，不丢数据，性能影响大
  - everysec：每秒刷盘，性能适中，最多丢失1s数据
  - no：操作系统控制，性能最好，可靠性较差，可能丢失大量数据
- AOF文件比RDB文件大的多
- aof会记录对一个key的多次写操作，但其实只有最后一次写有意义，可以通过bgrewriteaof命令，让aof执行重写，用最少的命令达到同样的效果
- 可用于紧急故障回滚
- 也可以设置阈值触发重写aof

### AOF和RDB混合持久化

- 同时使用两种持久化功能需要耗费大量系统资源，系统的硬件必须能够支撑运行这两种功能所需的资源消耗，否则会给系统性能带来影响
- Redis服务器在启动时，会优先使用AOF文件进行数据恢复，只有在没有检测到AOF文件时，才会考虑寻找并使用RDB文件进行数据恢复
- 当Redis服务器正在后台生成新的RDB文件时，如果有用户向服务器发送BGREWRITEAOF命令，或者配置选项中设置的AOF重写条件被满足了，那么服务器 将把 AOF重写操作 推延到 RDB文件创建完毕之后再执行，以此来避免两种持久化操作同时执行并争抢系统资源
- 同样，当服务器正在执行BGREWRITEAOF命令时，用户发送或者被触发的BGSAVE命令也会推延到BGREWRITEAOF命令执行完毕之后再执行
- 开启混合持久化，4.0之后才可以使用
  - `aof-use-rdb-preamble yes`
- 开启混合持久化之后，生成的aof文件由两部分组成
  - RDB+AOF

## 集群

### 分片集群

#### 简介

- 解决海量数据存储和高并发读写的问题
- 集群中有多个master节点，每个master保存不一样的数据
- 每个节点都会开启两个端口，一个用于数据交互，另一个用于节点间通信
  - 通信用的gossip二进制协议，占用更少的网络带宽和处理时间。

- 每个master可以有多个slave节点
- master之间心跳检测
- 客户端可以访问集群中的任意节点，最终都会被正确转发到正确的节点

#### 哈希槽

- redis分片集群引入了哈希槽的概念，一共有16384（2^14^ 大小的数组）个哈希槽，redis节点不会超过1000个
- 每个key通过CRC16校验后对16384取模来决定分配到哪一个哈希槽中，集群中的每一个节点负责一部分哈希槽
- 相当于在请求和redis之间添加了一个反向代理（槽），由它来选择存储的redis节点

### 哨兵集群

#### 简介

哨兵（sentinel）是 Redis 集群架构中非常重要的一个组件，主要有以下功能：

- 集群监控：负责监控 Redis master 和 slave 进程是否正常工作。
  - 基于心跳检测，每一秒ping一次，超过规定的时间没有响应，则视为**主观下线**
  - 超过一半（可设置）的哨兵认为该节点主观下线，则该节点变为客观下线

- 消息通知：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
- 故障转移：如果 master node 挂掉了，会自动转移到 slave node 上。
- 配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址。

哨兵用于实现 Redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作。

- 故障转移时，判断一个 master node 是否宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题。
- 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的

#### 哨兵选主规则

1. 判断主从断开时间长短，超过指定值则排除
2. 判断节点的salve-priority值，越小优先级越高，如果一样则判断offset，越大优先级越高
3. 最后判断节点运行id大小，越小优先级越高

#### 哨兵间的自动发现机制

- 是通过redis的 pub/sub 实现的，每个哨兵都会往 `__sentinel__:hello` 这个 channel 里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在
- 每个哨兵还会跟其他哨兵交换对 `master` 的监控配置，互相进行监控配置的同步

### 主从同步 / 主从集群

#### 简介

- 能够实现读写分离，但无法实现集群高可用
- 分为主节点master和从节点salve/replica
  - 主节点接收写操作
  - 从节点接收读操作
- 每一个主节点都有一个replid，这个id是数据集的标记，从节点会继承主节点的replid，从节点同步时会带上这个replid，如果和主节点的不一致则说明是第一次同步
- 主节点有一个偏移量offset，这个偏移量会根据replicated_baklog中的数据增多而逐渐增大，从节点完成同步时也会记录当前同步的主节点的偏移量，如果。从节点偏移量小于主节点，则会进行数据同步更新

#### 全量同步

1. **从节点**向主节点建立连接，通过执行**replicaof**命令
2. 请求数据同步，会携带replid、offset
3. **主节点**判断是否是第一次同步，通过判断replid是否一致，不一致则是第一次
   - 第一次
     - 是第一次，将主节点完整的版本信息发送给**从节点**，即replid、offset
     - **主节点**执行持久化生成RDB，发送给**从节点**
     - **从节点**清空本地数据，加载RDB文件
     - **主节点**会记录生成RDB文件期间接收到的所有命令，封装为replicated_baklog日志文件发送给**从节点**
     - **从节点**执行接收到的日志文件
   - 不是第一次
     - **主节点**计算从节点的偏移量和主节点偏移量的差值数据封装为replicated_baklog日志文件发送给**从节点**
     - **从节点**执行接收到的日志文件

#### 增量同步

- 一般在从节点重启时或者主从数据发送变化时
- 就是全量同步中不是第一次同步的情况

#### 脑裂

- 由于网络分区或者网络故障集群分割，两个分区都选举了master节点，故障恢复后其中一个master会变回salve节点，其间其接收到的写请求都会在同步后丢失
- 解决方法
  - 设置min-replicas-to-write 1：表示主节点最少需要有一个从节点
  - 设置min-replicas-max-lag 5：表示数据复制和同步的延迟不能超过5s

## 分布式锁

### setnx

- `set lock value NX EX 1`
- 必须设置失效时间兜底，否则会导致死锁的问题
  - 预估业务执行时间
  - 给锁续期，自己实现需要开一个线程去监控，比较麻烦

### Redission

#### 加锁原理

- 使用lua脚本保证的原子性

#### 可重入原理

- 使用hash结构来记录获取锁的线程id，value记录的是重入的次数
- 内部还是实现的lua脚本

#### 可重试原理

利用信号量和PubSub功能实现

- 使用带参的构造函数获取锁，参数是重试的等待时间和时间单位
- 底层会给一个默认的超时时间30秒（**看门狗时间**）
- 当获取锁失败时会返回一个锁的剩余有效期 (**pttl**) (剩余的超时时间)
- 用等待时间减去获取锁失败这个过程消耗的时间
  - 小于等于0，返回false
  - 大于0，进入等待通知状态（**订阅状态**），当有线程释放锁时会通知其他等待的线程
- 当等待通知的时间超过了等待时间时，取消订阅状态，返回false
- 当在等待的期间获得了通知
  - 用等待时间减去等待通知这个过程消耗的时间
    - 小于等于0，返回false
    - 大于0，进入一个循环，尝试获取锁，成功返回
      - 失败进入一个等待**信号量**的状态，ttl时间到或者等待时间到时尝试获取锁
        - 判断等待时间，超时false，不超时进入下一轮循环
- 代码中经常涉及到对剩余时间的计算，比较严谨

#### 不超时原理

- 使用带参的构造函数获取锁，参数是重试的等待时间、超时时间和时间单位 
- 不设置超时时间才能实现锁不超时（**看门狗机制**）

- 当获取锁成功时会调用**scheduleExpirationRenewal()**方法
  - 方法中会创建一个**ExpirationEntry**对象，用一个**ConcurrentHashMAp**存储，key是当前锁的名称，value是**ExpirationEntry**对象，一个锁对应一个entry，重入不覆盖
- 执行**renewExpiration**方法刷新超时时间
  - 方法中会创建一个**Timeout**定时线程，只要当前线程持有锁，每隔**看门狗时间**
  
    /3 秒就会调用**renewExpirationAsync**异步刷新一次超时时间
- 会把线程id作为key，定时刷新的线程作为value存入一个map中
- 当锁释放时，执行**cancelExpirationRenewal**
  - 根据锁id取出map中的定时刷新线程然后取消掉
  - 然后把entry删除
- 避免阻塞导致锁超时释放

#### 主从一致（**连锁**）

- Redission不分主从关系，加锁时会向所有redis集群中一半以上的节点加锁，这些节点可以有从节点，即红锁
  - 实现复杂，性能影响大

- 通过**redissionClient.getMultiLock(锁对象1,锁对象2...)**创建连锁
  - 内部会把这些锁放在一个集合中
    - 所有操作都需要集合中每一个锁都成功才能成功

- 在获取每一个锁时如果超时会释放所有以获得的锁
- 如果设置了超时时间，在获取所有锁之后会一次性给所有锁刷新超时时间
- 没设置超时时间会使用**看门狗机制**

## 多级缓存

### JVM进程缓存

#### Caffeine

### Nginx本地缓存

#### Lua

### OpenResty

## 最佳实践

### 批处理
