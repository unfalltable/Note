---
title: JUC源码
categories: 源码
tags: [源码]
---

## AtomicInteger

```java
//自增自减都是通过这个方法实现，
unsafe.getAndAddInt(this, valueOffset, 1 / -1);
```

```java
//复杂运算
public final int updateAndGet(IntUnaryOperator updateFunction) {
    int prev, next;
    do {
        prev = get();
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSet(prev, next));
    return next;
}
```

- IntUnaryOperator是一个函数式接口，使用lambda传值

## ConcurrentHashMap

### 成员变量

- sizeCtl
  - 下一次扩容的阈值
  - 默认为0，初始化时为-1
  - 扩容时为 - (1 + 扩容线程数)
- Node
  - 节点
- ForwardingNode
  - 继承Node
  - 扩容时作为旧数组的头节点
- ReservationNode
  - 继承Node
  - 在compute以及computeIfAbsent时，用来占位
  - 计算完成后替换为普通Node
- TreeBin
  - 继承Node
  - 作为树的头节点
  - 存储root和first
  - 加了读写锁
- TreeNode
  - 继承Node
  - 作为TreeBin的节点
  - 存储parent，left，right
- Node[] table
  - 哈希表
- Node[] nextTable
  - 扩容时的新哈希表

### 成员方法

- tabAt
  - 获取Node[]中第 i 个Node
  - 参数：table，下标
- casTabAt
  - cas修改Node[] 中Node的值
  - 参数：数组table、下标、旧值、新值
- setTabAt
  - 直接修改Node[] 中 Node的值
  - 参数：数组table、下标、新值

### 构造方法 / 初始化

- 1.7
  - 初始化时就会创建segment数组和HashEntry数组的0号元素
    - segment数组默认大小16
    - 初始化后segment数组的长度就不能改变
    - 并发度是ConcurrentHashmap的长度（segment数组的大小）
    - 这个数组的0号元素为其他HashEntry数组的创建提供原型
    - 每个片段下的数组大小 = 容量 / 并发度
      - 最小值为2，数组的元素数量超过阈值（0.75）就会扩容
  - 指定容量时大于等于接近等级的容量x0.75时，容量会升一级创建
- 1.8
  - 懒惰初始化，第一次put数据时才会创建数组结构，默认初始容量是16
  - 无参构造创建时，数组初始容量是16
  - 初始化时传了容量大小时，判断是否超过容量×负载因子
    - 大于则创建下一个阶段的大小

### 索引计算

- 1.7
  - segment数组索引
    - 并发度为16（2^4^）时，看二次hash值的高四位
    - 并发度为32（2^5^）时，看二次hash值的高五位
  - 数组索引
    - 数组的大小为2（2^1^）时，看二次hash的低一位
    - 数组的大小为4（2^2^）时，看二次hash的低二位

### put方法

- 1.7
  - 头插法
  - hash找segement，内部数组执行put操作
  - 使用cas初始化槽
  - 通过tryLock()获取锁，获取不到就重试，再获取不到就lock住，直到获取锁
- 1.8
  - 尾插法，锁链表头或红黑树根节点

### get方法

- 1.7
  - 计算hash找segement，再hash找数组，再遍历链表
  - 没加锁，需要考虑并发问题
    - get时有put操作
      - 源码中使用了CAS保证初始化segement时也能get
      - 源码中使用了UNSAFE.putOrderedObject保证put后的数据也能被get到
      - 源码中使用了volatile 保证扩容时数据也能被get到
    - get时有remove操作
      - 源码中使用了UNSAFE.putOrderedObject保证remove后的数据不会被get到
      - 源码中使用了volatile 保证remove时数据不会被get到
- 1.8
  - 计算hash，找数组对应位置
    - 如果hash值为0，说明正在扩容或者是红黑树

### 初始化table方法

- 1.8
  - size计算实际发生在put，remove改变集合元素的操作之中
    - 没有竞争发生，向baseCount累加计数
    - 有竞争发生，新建counterCells，向其中的一个cell累加计数
      - counterCells初始有两个cell
      - 如果计数竞争比较激烈，会创建新的cell来累加计数


### 链表转红黑树

- 1.8
  - 

### 扩容

- 1.7
  - segement不能扩容，扩容的是HashEntry数组
  - 元素个数 > 容量 × 负载因子 时扩容，大小变为2倍，先扩容，再插值
- 1.8 
  - 元素个数 >= 四分之三时扩容，与负载因子无关
  - 以链表为单位，从后往前迁移数据，处理完当前位置会在这个位置打上标记（forwardingNode）
  - 可以多个线程帮忙扩容，一个线程扩容16个位置，从后往前做数据迁移

## ReentrantLock

- 锁的竞争是用的CAS机制来实现的
- 没有竞争到锁时是使用AQS来存储竞争锁的线程
- 锁的重入是内部变量记录了当前持锁线程的id，重入时判断id是否相同

## ThreadLcoal

## LongAdder

### 成员变量

- Cell[] cells
  - 累加单元数组，懒惰初始化，24字节
- base
  - 基础值，如果没有竞争，则用cas累加这个域
- cellsBusy
  - 在cells创建或扩容时，置位1，表示加锁
- Cell
  - 内部维护一个value值
  - 用cas进行累加
  - 一个缓存行(64字节)可以存下2个Cell，但是会有问题
    - 当有两个线程同时修改同一个缓存行中的cell时，一方的修改会导致另一方失败
    - @sun.misc.Contended可以解决这个问题，它通过在对象的前后各加上128字节的padding，从而让cpu将对象预读至不同的缓存行，所以不会造成失效

## 线程池

### 任务执行流程

1. 判断任务是否为null，不为空正常往下走，为空抛异常
2. 检查是否能创建工作线程，通过比对stl低29位和线程池参数
   - 还能创建核心线程，创建并执行任务
   - 不能创建核心线程
     - 有可能创建失败，同时又多个任务进来，然后同时创建新的线程，核心线程数够了的话就会有可能创建失败，也有可能核心线程数已经满了
3. 如果不能创建核心线程，判断线程池状态，判断是否能放进阻塞队列中
   - 线程池正常并成功放进队列中
     - 判断线程池状态，判断是否能去除等待队列中的任务
       - 线程池正常并能取出队列中的任务，查看工作线程是否有空闲，空闲直接执行任务，否则创建救急线程
       - 线程池异常或不能取出队列中的任务，走拒绝策略
   - 线程池异常或不能放进队列中，尝试添加救急线程处理，不能添加就走拒绝策略

### 添加线程流程

- 核心方法是addWorker方法，方法主要做了两步

  1. 检验线程池状态以及工作线程个数

     1. 取ctl高三位判断线程池状态，不是running状态则失败

     2. 通过传入的参数判断是添加核心线程还是救急线程

     3. 取ctl的低29位，

        - 判断线程数是否超出最大线程数

        - 判断创建核心线程还是救急线程

        - 使用cas方式对ctl低29位进行+1操作

  2. 构建线程并且启动

     - new一个工作线程，获取他的线程对象
     - 工作线程是放在一个hashset中的，添加的时候使用了lock锁保证线程安全
     - 通过线程对象启动任务

## 
