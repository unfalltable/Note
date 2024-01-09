---
title: JUC并发编程
categories: Java基础
tags: [基础]
---

## 简介

### 为什么需要多线程

- 解决**可见性**问题：因为CPU和内存间存在缓存，存在主从不一致的情况
- 解决**有序性**问题：因为编译程序执行指令时为了提高缓存使用效率，会调整指令的执行次序
- 解决**原子性**问题：因为有多线程、多进程、分数复用CPU，导致存在资源共享的问题

### 线程安全问题

- 多个线程访问同一个共享变量就会出现线程安全问题
- private 和 final可以一定程度上提供线程安全
- 线程安全的方法组合起来就不能保证线程安全了
- 在多线程的情况下满足JMM规范就可以避免线程安全问题
- 线程安全程度
  - 不可变
    - final、枚举、Number部分子类（Long、Double、BigInteger、BigDecimal）、使用`Collections.unmodifiableXXX()` 包装的集合
    - 不可变类：DateFormatter、String
  - 绝对线程安全
    - 调用者都不需要任何额外的同步措施
  - 相对线程安全
    - 有一定的安全，但有意外的情况，例如顺序的连续调用
    - 通过一些同步方法来保证安全
  - 线程兼容
    - 对象本身不安全，需要通过一些同步方法来保证安全
  - 线程对立
    - 即使使用了同步方法也无法保证线程安全

### 临界区

- 多线程读写同一个共享资源，这个共享资源就称为临界区

### 竟态条件

- 多个线程同时在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竟态条件

### 创建线程的方式

- 继承Thread
  - 代价比较大，不建议
  - 内部实现了Runnable接口
- 实现Runnable接口
  - 本质上并不是创建线程，实际执行还是需要通过线程调用
- 实现Callable接口
  - 本质上并不是创建线程，实际执行还是需要通过线程调用
  - 可以有返回值（FutureTask），需要实现call方法
  - 内部是FutureTask实现了RunnableFutrue接口，这个接口继承了Runnable和Future接口
  - 内部实现也是执行的run方法，在方法中创建了Callable对象并执行call方法，将结果值返回
- 线程池创建
  - 合理利用线程资源，减少频繁创建销毁的开销
  - 内部创建的worker线程也是实现了Runnable接口

### 守护线程和非守护线程

- 守护线程就像是守护其他线程一样，其他的完成了他也就结束了
- 非守护线程都运行结束了，即使守护线程还没结束也会直接结束
- main主线程是一个非守护线程
- setDaemon(true) 将线程设置为守护线程
  - 垃圾回收器（gc）就是一个守护线程
  - Tomcat中的 Acceptor 和 Poller 线程也是守护线程

### Java的线程和操作系统的线程有区别吗？

- java线程分为守护线程和用户线程
- 在window上线程是一样的，在Unix上采用Pthread实现
- jdk1.2之前是jvm自己有一套线程管理机制，即和操作系统上的不一样
- jdk1.2之后采用了操作系统原生的内核级线程，将线程的调度交给了操作系统内核
- 总的来说，java的线程就是操作系统的线程，Java线程与操作系统线程一对一映射，依赖于操作系统的具体实现

## 方法

sleep()

- Thread提供的方法，不会释放锁
- 使当前线程进入阻塞状态，可通过interrupt()打断
- 睡眠结束后线程未必会立刻得到执行，等cpu轮转

wait / notify

- object的方法，必须与synchronized一起使用，会释放锁
  - 在多个线程之间通信需要保证线程间对共享变量的修改是可见的
  - synchronized就可以保证可见性

- 实现多线程之间的通信
- 调用 wait() 方法让当前线程进入waitSet中等待
- 通过 notify() / notifyAll() 唤醒等待中的线程
- 防止虚假唤醒

==yield==

- 调度执行其它同优先级的线程
  - 如果有，当前线程挂起等待相应的线程执行结束
  - 如果没有，则继续运行当前线程


interrupt

- interrupt()设置打断标记为true
  - 睡眠中的线程被打断不会将标记设置为true
- isInterrupted() 判断线程是否被打断 不会清除标记
- interrupted() 判断线程是否被打断 会清除标记 即设置为false

==Join==

- 在线程中调用另一个线程的 join() 方法，会将当前线程挂起，直到目标线程结束

Park

- 让当前线程停下来
- 如果打断标记为false则停止线程 ，如果打断标记为true则不会停止线程
  - 即只能打断一次


优先级设置

- setPriority() 为线程设置新的优先级。默认是5
- getPriority() 返回线程的当前优先级

Fork / join

- 体现的是一种分治思想，把递归交给线程做
- 默认会创建与cpu核心数数量相同的线程池

- 需要继承RecursiveTask\<V>（有返回结果）或RecursiveAction（无返回结果）
  - 覆盖重写compute()方法，实现任务拆分
- 创建ForkJoinPoll线程池，执行RecursiveTask实现类对象

## 生命周期 / 运行状态

cpu层面

- 创建：create
- 就绪：Runnable 
- 运行：run
- 等待：wait
- 终结：Terminated

java层面，在Thread类中有一个枚举类，规定了java线程的六种生命周期

- 新建 - New
- 运行 & 就绪 -Runnable 
- 阻塞 - Blocker（Waiting、Time_Wating）
- 销毁 - Terminated

## 关键字

### Synchronized

- 是Java提供的同步关键字，可以修饰在方法和类上，可以以代码块形式使用
- 可以通过锁的对象来控制锁的作用范围
  - 锁静态对象或类对象，那么就是一个全局锁
  - 锁普通实例对象，那就是取决于对象的生命周期

#### 锁升级

轻量级锁

- 锁对象虽然是多线程访问的，但是线程间不存在竞争，此时Synchronized会优化为一个轻量级锁

偏向锁

- 在轻量级锁的情况下，线程每次访问锁对象时都需要进行CAS操作，增加了开销
- java6时引入了偏向锁来优化，在cas前会判断线程id和锁对象的对象头中的线程id是否一致，一致直接通过
- 偏向锁默认是开启的，但是默认也是延迟的，即看不出实际的效果
  - 可以设置 `-XX:BiasedLockingStartupDelay=0` 来禁用延迟
  - `-XX:-UseBiasedLocking` 关闭偏向锁

- 偏向锁关闭的情况
  - 锁升级
  - 对象调用了hashcode
  - 调用wait/notify

- 当撤销偏向锁的次数超过了阈值，整个类的所有对象都会变为不可偏向

重量级锁（Monitor锁 / 管程锁）

- 结构
  - WaitSet：
  - EntryList：阻塞队列
  - Owner：持有的线程
- 加锁流程
  1. 当线程获取重量级锁时，Owner指向当前线程
  2. 其他线程获取锁时会会先进行自旋获取锁，失败一定次数后进入EntryList等待
     - jdk 1.6 后自旋是自适应的，会根据之前线程的平均自旋次数自动调整自旋的次数
  3. 线程释放锁后，唤醒EntryList中等待的线程来竞争锁，是非公平的

#### 锁优化

锁膨胀

- 循环的粒度太小导致频繁的加锁释放锁，JIT即时编译器会将锁的粒度变大来优化

自旋优化

- 重量级锁竞争时会先通过自旋尝试获取锁，没获取到再进入阻塞队列
- 有一定的次数，也有自适应的自旋

锁粗化

- 多个细粒度的加锁流程会被优化为一个大粒度的锁，减少频繁的加锁释放锁

锁消除

- JIT即时编译器判断锁对象不会被共享时会消除锁

### Volatile

- 可以保证在多线程环境下共享变量的可见性
  - 对于增加了 volatile 关键字修饰的共享变量，JVM会自动增加一个#Lock汇编指令，这个指令会根据CPU 型号自动添加**总线锁**或**缓存锁**
- 通过增加内存屏障防止多个指令之间的重排序
  - 因为CPU引入了 StoreBuffer机制，而这一种优化机制会导致 CPU 的乱序执行，Volatile通过内存屏障来避免CPU重排序
  - 跳过编译器的指令重排序

## JUC工具包

### 原子变量

- AtomicBoolean、AtomicInteger、AtomicLong
  - 底层使用的Unsafe提供的Cas实现线程安全的计算
- AtomicReference、AtomicMarkableReference、AtomicStampedReference、AtomicIntegerArray
- ...FieldUpdater

### AQS

- 全称是AbstractQueuedSynchronizer，是一个多线程同步器和阻塞式锁的框架，很多JUC组件的底层都是由它实现的，如Lock、CountDownLatch、Semaphore等
- 本质上来说，AQS提供两种锁的机制，分别是排他锁和共享锁，用state属性来表示资源的状态
- 提供了一个volatile修饰的int类型的state
  - 锁记录线程是否持有
  - Semaphore记录有多少资源
  - CountDownLatch中作为计数器

- 提供了基于FIFO的等待队列（双向链表），类似于Monitor的EntryList
- 提供条件变量ConditionObject来实现等待队列，也是一个双向链表，
  - 唤醒机制，调用了await的线程会在这个队列中等待唤醒
  - 支持多个条件变量。类似于Monitor的WaitSet


### ReentrantLock

- 锁的粒度由lock和unlock之间包裹的代码范围决定，作用域则取决于lock实例的生命周期
- 基于AQS，内部有一个sync抽象静态内部类，它有两个实现类
- 支持公平锁和非公平锁，默认非公平
  - NonfairSync 非公平同步器
  - FailSync 公平同步器
    1. 自旋获取锁，会判断在自己之前是否有线程
    2. 加入队列执行 addWaiter方法和 acquireQueued方法

- 可重入、可打断、支持锁超时、多条件变量，支持非竞争获取锁

- 锁的竞争是使用 CAS 机制来实现的

### ReentrantReadWriteLock

- 基于AQS实现的，将state拆成了两部分
  - 写锁占state的低16位，读锁是state高16位
  - 使用了ThreadLocal解决多线程读时无法判断线程的重入次数
    - ThreadLocal记录了线程重入次数
  
- 读读不加锁，类似数据库意向共享锁
- 内部分别使用读锁保护数据的read方法，写锁保护write方法
- 读锁不支持条件变量
- 持有读锁时去获取写锁，会导致获取写锁永久等待
- 读写锁用的是同一个Sycn同步器，所以等待队列，state也是同一个
- 有写锁饥饿问题
  - 当有大量读操作持有锁时会导致写锁等待
  - 新的读操作需要排到写操作之后执行才能避免写锁饥饿
    - 即每个读锁获取锁资源前先判断是否有等待的写锁


### ThreadLocal

- 是一种线程隔离机制，它提供了多线程环境下对于共享变量访问的安全性
- ThreadLocal本身不存储数据，像是一个工具类，去操作Thread中的ThreadLocalMap
- ThreadLocalMap是ThreadLocal的一个内部类
  - ThreadLocalMap是基于Entry数组实现的
  - key是ThreadLocal对象本身，value就是要存的数据

- 真正存数据的地方是Thread中的成员遍历ThreadLocalMap
- Thread的key是弱引用，在GC时会被清除，因为存在内存泄露的风险
  - 如果想用强引用类型的可以使用ThreadLocal的子类InheritableThreadLocal
  - get时遇到key为null值，会将其value置为null
  - set时遇到key为null值，会替换key-value，还会删除改位置一定范围内的key为null的键值，范围和map中的元素个数有关
- 推荐使用后用remove删除元素，因为value是会出现内存泄漏的问题的
- 推荐设置ThreadLocal对象为static final 的，这样Gc就不会删除了，避免了内存泄漏
- 每个ThreadLocal会对每一个线程创建一个ThreadLocalMap，第一次使用才会创建
  - key是ThreadLocal实例，value是数据
  - 初始容量是16，扩容2倍，负载因子是2/3, 大于等于阈值时扩容
- 开放寻址法解决hash冲突
  - 插入位置有数据时，接着遍历找没数据的位置后插入
- 存在ABA问题，可以通过版本号解决


### ConcurrentHashMap

- 本质上还是HashMap，只是做了并发安全的处理
- 扩容时采用了多线程并发扩容的机制，也是通过创建新的数组然后数据qian'yi

- 提供的方法是保证线程安全的，但是多个方法同时使用不是线程安全的

- 结构

  - 1.7：segment段+HashEntry数组+链表

  - 1.8：数组+链表 / 红黑树

### LongAdder

- 原子累加器，用一个原子整型模拟锁
- 用了@Contended解决了伪共享的问题

### Condition

- 相当于wait/notify

### StampedLock

- jdk8加入的，用于优化读性能，使用读写锁时配合使用
- 支持乐观读，读取完毕后做一次戳校验，通过则表示这期间没有写操作，数据可以安全使用，如果没通过需要重新读取数据
- 不支持条件变量、不可重入

### Semaphore

- 信号量，用于限制访问共享资源线程的上限，限流

### CountdownLatch

- 用来进行线程同步协作，内部维护一个计数器，线程完成计算减一

### CyclicBarrier

- 用来进行线程同步协作，内部维护一个计数器，等待线程满足某个计数，即可调用wait进行等待，再等待满足某个计数再继续执行
- 计数要和线程数一致

### CompletableFuture

- 1.8 引入的一个基于事件驱动的异步回调类
- 使用异步线程去执行一个任务的时候，在任务结束以后触发一个后续的动作
- 提供了 5 种不同的方式
  1. thenCombine：把两个任务组合在一起，当两个任务都执行结束后触发事件回调
  2. thenCompose：把两个任务组合在一起，这两个任务串行执行，也就是第一个任务执行完以后自动触发执行第二个任务
  3. thenAccept：

## 线程池

### 简介

- 是一种资源复用的思想，目的是减小线程频繁创建和销毁带来的开销，因为线程创建会涉及到cpu的上下文切换，内存分配等工作
- 仅适合短链接的场景，使用时推荐手动创建ThreadPool对象而不是shi'yong
- 核心参数
  - corePoolSize：核心线程数
  - maximumPoolSize：最大线程数
  - keepAliveTime：救急线程的空闲时间
  - unit：时间单位
  - workQueue：阻塞队列
    - 默认是LinkedBlockingQueue
  - threadFactory：线程工厂
    - 设置线程的信息，起名，设置守护线程、优先级等等
    - 当任务没有核心线程去执行时会加入阻塞队列，队列满了会创建救急线程
  - handler：拒绝策略
    - AbortPolicy：抛出异常，默认
    - CallerRunsPolicy：主线程帮忙处理任务
    - DiscardOldestPolicy：抛弃等待队列头节点任务，即最早的任务
    - DiscardPolicy：抛弃当前任务
- 处理机制
  - 当有新的任务进来时
    - 如果核心线程数没满，即使有空闲的核心线程也会创建新的核心线程来处理任务
    - 如果核心线程满了，就把任务加入等待队列
    - 如果队列满了，创建救急线程处理队列中的任务
    - 如果最大线程数和等待队列都满了，则根据拒绝策略处理任务

### 种类

- newFixedThreadPool
  - 固定的线程池
  - 都是工作线程
- newSingleThreadExecutor
  - 单例线程池，只有一个工作线程
  - 顺序消费
- newCachedThreadPool
  - 没有工作线程，任务进来后丢进阻塞队列，然后创建救急线程去执行
  - 频繁的创建和销毁线程，性能不好
- newScheduleThreadPool
  - 定时任务的线程池
  - 阻塞队列是DelayWorkQueue，底层是堆
- newWorkStealingPool
  - 基于ForkJoinPool实现的
  - 每个线程都有一个阻塞队列，当线程对应的阻塞队列为空时，会去访问其他线程的阻塞队列并取出任务执行
  - 会将任务进行拆分，然后并行处理，并将所有返回的结果合并

### 属性 / 状态

- crl
  - 原子整形
  - 32位，高三位是线程池的状态，低29位是线程池的数量
  - 状态
    - 111 = -1：Running，正常处理任务，刚构建出来时的状态
    - 000 = 0：Shutdown，不接收新任务，会处理完所有的任务
      - shutdownNow方法会直接进入stop状态
    - 001 = 1：stop，不接收新任务，所有的任务中断
      - 本质上是执行interrupt方法
    - 010 = 2：Tidying，过度阶段
    - 011 = 3：Terminated，关闭线程池
      - 可以实现该方法，实现一些关闭前的操作
