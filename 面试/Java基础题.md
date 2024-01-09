---
title: 基础题
categories: 面试
tags: [面试]
---

## JavaSE

- [ ] 谈谈你对面向对象的理解
  - [ ] 定义
  - [ ] 优缺点
  - [ ] 设计模式七大原则
    - [ ] 合成复用原则
  - [ ] 类与类之间的关系
    - [ ] 里氏替换原则
- [ ] 谈谈你对集合的理解
  - [ ] 底层实现
  - [ ] 区别
  - [ ] Set怎么实现去重
  - [ ] HashTable、HashMap、ConcurrentHashMap的区别
- [ ] == 和 equals 
  - [x] 区别
- [x] 创建对象的方法
- [ ] 异常相关
- [ ] 抽象类和接口
  - [x] 区别
- [ ] Double运算有什么问题
  - [x] 问题
  - [x] 解决
- [ ] 多态的底层原理
  - [ ] 动态绑定
- [ ] 怎么减少hash冲突
  - 拉链法、开放寻址法、多哈希算法、一致性哈希算法
- [ ] Fail-Fast和Fail-Safe
- [x] new String("aa") 创建了多少个对象
- [x] a="123" 和 a = new String("123")的区别
- [ ] final原理
- [ ] String、StringBuffer、StriBuilder
  - [x] 区别
  - [ ] 用途
- [ ] Volatile
  - [x] 作用
  - [ ] 原理
- [ ] 集合

  |          | list | set  | map    | queue |
  | -------- | ---- | ---- | ------ | ----- |
  | 元素重复 | √    | ×    | ×      |       |
  | null值   |      |      |        |       |
  | 速度     |      |      |        |       |
  | 底层     | 数组 | map  | 哈希表 |       |
  | 分类     |      |      |        |       |

  |      | ArrayList | LinkedList | ArrayDeque | PriorityQueue | BlockingQueue | Vector |
  | ---- | --------- | ---------- | ---------- | ------------- | ------------- | ------ |
  |      |           |            |            |               |               |        |
  |      |           |            |            |               |               |        |
  |      |           |            |            |               |               |        |

  |      | HashSet | LinkedHashSet |
  | ---- | ------- | ------------- |
  |      |         |               |
  |      |         |               |
  |      |         |               |

  |          | HashMap         | HashTable            | ConcurrentHashMap         | LinkedHashMap | TreeMap             | WeakHashMap |
  | -------- | --------------- | -------------------- | ------------------------- | ------------- | ------------------- | ----------- |
  | 线程安全 | 不安全          | 安全                 | 安全                      |               |                     |             |
  |          | 无contains()    | 有contains()         | 有，等价为containsValue() |               |                     |             |
  |          | 可null          | 不可null             | 不可null                  |               |                     |             |
  |          | 二次hash        | 一次hash             | 二次hash                  |               |                     |             |
  |          | 继承AbstractMap | 继承Dictionary       | 继承AbstractMap           |               |                     |             |
  |          | 初始16          | 初始11               | 初始16                    |               |                     |             |
  |          | 扩容 X2         | 扩容 X2 + 1          | 扩容 X2                   |               |                     |             |
  | 创建时机 | put时创建       | 构造时创建           | put时创建                 |               |                     |             |
  | 并发度   | 无              | 低（synchronized锁） | 高（分段锁）              |               |                     |             |
  | 场景     | 键值的存储      |                      |                           | LRU           | key的顺序取存储键值 |             |

- [ ] HashMap

  - [ ] 为什么用红黑树不用AVL树
  - [ ] 红黑树何时退化为链表
  - [ ] 树化阈值为什么是8
  - [ ] 索引如何计算
  - [ ] 为什么要进行二次hash
  - [ ] 容量为何是2的n次幂
  - [ ] 扩容后如何确定新位置
  - [ ] 扩容一致为什么是0.75
  - [ ] 多线程下存在什么问题
- [ ] 红黑树

  - [ ] 特点
  - [ ] 方法

- [ ] Unsafe类

  - [ ] 操作内存
  - [ ] cas
  - [ ] 内存屏障

## JVM

- [ ] JMM内存模型
  - [ ] 为什么堆保存对象，栈保存函数参数这些
    - 堆是由程序员手动申请和释放的，栈是由系统自动分配和回收的
    - JVM中堆保存对象是因为对象的大小和生命周期不确定，需要动态分配和管理
    - 栈保存函数参数和局部变量是因为它们的大小和生命周期相对固定，可以由系统快速分配和回收
- [ ] 类加载机制
  - [ ] 字节码文件的内容
  - [ ] 类加载的过程
    - 五个阶段、顺序开始、解析阶段开始时机是不确定的，为了支持动态绑定
- [ ] 垃圾回收
  - [ ] 引用类型
  - [ ] 垃圾回收算法
  - [ ] 分代回收
    - [ ] 触发老年代GC的时机
    
    - [ ] 对象直接进入老年代的情况
  - [ ] 垃圾收集器
    - [ ] CMS
    - [ ] G1
  - [ ] 可达性分析
  - [ ] 三色标记法
  - [ ] 优化手段
- [ ] 对象
  - [ ] 组成
  - [ ] 如何定位
  - [ ] 对象的创建方式
- [ ] 字符串常量池
- [ ] 直接内存
- [ ] 动态绑定和静态绑定
- [ ] 类加载器
  - [ ] 双亲委派模式
- [ ] 触发老年代GC的时机
- [ ] 对象进入老年代的情况
- [ ] new Object() 在内存中占用多少字节
- [ ] 调优
  - [ ] 对数据量大的任务和数据访问量大的任务怎么调优
  
- [ ] JVM问题排查

  - 系统正在运行的
    - jmap看JVM内存情况
    - jstack看线程运行情况，阻塞死锁等
    - jstat看垃圾回收状况
    - jvisualvm看
    - 如果频繁fullgc但是又回收了大量对象的话
      - 可能是因为对象比较大直接进入了老年代
      - 可以调大年轻代的大小，让这些对象在minorgc时就回收掉
  - 系统已经oom
    - 看dump文件，用jvisualvm分析


## JUC

- [ ] 创建线程的方式

  - 四种，底层都是实现的runnable实现

- [ ] 线程的状态

  - 从cpu和java两方面讲

- [x] 守护线程和非守护线程

- [ ] 锁

  - [x] 什么是锁的可重入
  - [ ] 不可重入锁

    - 线程池中的worker工作线程，它继承了AQS实现了锁，这个锁是不可重入的，但是没用到
  - [ ] CAS
    - [x] 原理
    - [ ] 优缺点
      - 避免了线程挂起唤醒导致用户态和内核态之间的切换
      - 自旋时间和次数过长比较占用cpu资源
      - 可以像synchronized中的自旋优化一样解决这个问题
      - 也可以像LongAdder的分段锁
      - ABA问题
        - 虽然可以更新数据，但是不符合原子性
        - 通过添加版本号解决AtomicStampReference

- [ ] volatile

  - [ ] 作用
    - 禁用cpu缓存，保证共享变量的可读性
    - 保证代码的有序性
  - [ ] 原理
    - [ ] 禁用cpu缓存

- [ ] sleep / wait

  |      | wait         | sleep        |
  | ---- | ------------ | ------------ |
  | 来源 | Object的方法 | Thread的方法 |
  | 特点 | 会释放锁     | 不会释放锁   |
  |      |              |              |

- [ ] wait / notify

  - [ ] 为什么要写在 synchronized 代码 块中
    - wail实际上操作的是motor锁中的waitset队列
    - wait的作用是为了实现线程间的通信，那么就需要能接收到其他线程对共享变量的修改或者通知其他线程共享变量被修改了，synchronized可以保证一定的可见性，它会将修改过后的信息刷进内存中

- [ ] Synchronize

  - [ ] Synchronize 和 Lock 的区别

    |          | Synchronize                | Lock                                    |
    | -------- | -------------------------- | --------------------------------------- |
    | 来源     | java关键字                 | juc接口                                 |
    | 场景     | 竞争小，有很多优化         | 竞争大，自旋优化                        |
    | 特点     | 非公平锁，可重入，不可中断 | 公平锁/非公平锁，可重入、可中断，多条件 |
    | 时机     | 自动释放锁                 | 手动申请和释放锁                        |
    | 作用对象 | 修饰方法、修饰类、代码块   | 修饰一块区域                            |
    | 阻塞     | 无法实现非阻塞竞争         | youtryLock方法实现非阻塞竞争            |
    | 性能     | 竞争小时好                 | 竞争大时好些                            |

  - [ ] 升级
  - [ ] 优化
    - [ ] 为什么要优化
  - 重量级锁每次加锁解锁都涉及到用户态和内核态之间的切换，性能影响大

- [ ] LongAdder

- [ ] ReentrantLock

  - [ ] 特性

  - [ ] 原理

  - [ ] ReentrantReadWriteLock

  - [ ] 中断原理

- [ ] StampedLock

- [ ] Semaphore

- [ ] CountdownLatch

- [ ] CyclicBarrier

- [ ] ConcurrentHashMap

  - [ ] 底层实现原理
  - [ ] 1.8优化
    - 

- [ ] ThreadLocal

  - [ ] 是什么
  - [ ] ThreadLocal和Synchronized的区别
    - 思路不同
  - [ ] 索引的计算
    - 累加常量1640531527 取模
  - [x] ThreadLocalMap中的key为什么要设置为弱引用 / 内存泄漏问题
    - key的内存泄露ThreadLocal已经通过弱引用解决了，但是还存在value的内存泄漏
    - 推荐使用remove来释放值的内存，因为一般开发中会将ThreadLocal定义为静态final变量，为强引用，所以gc不会清理key

- [ ] CopyOnWriteArrayList

- [ ] AQS

  - [ ] 理解

  - [ ] 为什么采用双向链表
    - 新线程加入队列时需要判断前驱节点的状态
    - 线程在链表中需要自旋尝试竞争锁，此时需要判断自己是不是头节点

  - [ ] 唤醒队列中的节点时为何从后往前
    - 插入节点时插入队尾，新节点会指向队尾节点，但是队尾节点是指向null的，从前往后遍历的话可能会错过节点
    - 取消节点时的指针更改也是先改前驱节点，再改后继节点，所以从后往前遍历可靠性会高一些

- [ ] 线程池
  - [ ] 种类
  - [x] 参数
    - [ ] 如何设置
      - 很难控制任务类型，cpu密集型，io密集型，混合型
      - 需要通过压测来参考
      - 动态监控和修改线程池
  - [ ] 状态
  - [ ] 任务执行流程
  - [ ] 创建线程流程
  - [ ] 为什么要使用线程池
  - [ ] 线程池用完了为什么一定要shutdown
    - 源码中对线程启动是通过工作线程worker对象的start方法启动的，会在栈中创建一个线程栈
    - 由于工作线程默认是不会销毁的，所以不会被gc回收
    - shutdown会将线程池从running状态改为shutdown，工作线程不会再阻塞读取队列
      - shutdownNow会改为stop状态
    - 还会调用中断方法，将没有在执行任务的方法的中断标记设置为true
  - [ ] 为什么要构建空任务的救急线程
    - 避免阻塞队列中的任务因为线程池中没有工作队列，也没有新任务进来创建新的工作线程导致的饥饿问题
    - 线程池是运行设置核心线程数为0的
    - 工作线程被设置了超时，一个任务进入队列时工作线程刚好超时
    - 所以线程池会在任务进入阻塞队列时判断工作线程是否为0，为0创建空任务的救急线程
    - 在execute方法中当任务进入阻塞队列时，判断当前工作线程是否为0，为0创建一个救急线程
  - [ ] 线程池如何知道线程的任务已经完成
    - 使用isTerminated方法，当任务都完成返回true，需要先调用shutdown方法和阻塞主线程（不推荐）
    - 用submit执行线程，这种方式会有返回值，通过future.get()方法判断线程是否结束
    - 使用getCompletedTaskCount方法判断
    - 使用CountDownLatch判断，需要提前知道任务数且只能用一次
    - 使用CyclicBarrier判断，可重复使用

## Spring相关

- [x] spring的单例bean是线程安全的吗

- [ ] spring的Bean生命周期

- [ ] AOP
  - [ ] 使用的场景
  - [ ] 为什么JDK动态代理必须是接口
    - 因为动态代理是通过Proxy.newProxyInstance()方法来实现的，这个方法需要传入被代理类的接口
    - 之所以要传入接口，因为JDK动态代理的底层实现是通过生成一个代理类$Proxy0，这个代理类会继承Proxy类，同时实现被代理类实现的接口，而JDK代理设计的想法就是为了对代理对象的方法进行增强，本质上就是对其接口进行增强，为了体现设计模式中的依赖倒转原则
  - [ ] JDK和CGLIB的区别

- [ ] 事务
  - [ ] 失效场景和原因
  - [ ] 为什么只能作用在public修饰的方法上
  - [ ] 事务的隔离级别
  - [ ] 事务的传播行为

- [ ] @Autowired

  - [ ] 失效的情况

  - [ ] 和@Resource的区别

    |          | @Autowired                    | @Resource        |
    | -------- | ----------------------------- | ---------------- |
    | 作用     | 成员、方法                    | 成员、方法       |
    | 来源     | spring提供                    | JDK提供          |
    | 注入方式 | byType，byName要结合@Qulifier | 先byName再byType |
    |          |                               |                  |

- [ ] spring如何处理线程并发问题
  - [ ] ThreadLocal
  - [ ] 声明为多例
  - [ ] syn / juc

- [ ] 循环依赖
  - [x] 原因
  - [ ] 解决
    - 提前暴露对象，将对象的创建和初始化分开
  - [ ] 只有一级 / 二级缓存能解决吗
    1. 即半成平对象和成品对象都放到一个map中，需要在value上打标签，每次获取对象时要取出value判断，很麻烦
    2. 前提是整个循环引用过程中没有使用到aop
  - [ ] 为什么使用AOP就需要有三级缓存
    - 因为在同一时刻会同时存在原始对象和代理对象，而属性在赋值调用和对外暴露的时候，没办法确定是原始对象还是代理对象，所以我需要在第一次对外暴露的时候，通过一个lambda表达式。类似于回调机制的东西，来把最终版本对象返回回去
  - [ ] 缓存的放置时间和删除时间
  
- [ ] BeanFactory、ApplicationContext、FactoryBean

  | BeanFactory    | ApplicationContext                   | FactoryBean      |
  | -------------- | ------------------------------------ | ---------------- |
  | 延迟加载Bean   | 一开始就创建好Bean                   | 用于快速创建Bean |
  | spring内部使用 | 内部间接调用Benfactory，扩展了新功能 |                  |
  |                |                                      |                  |

- [ ] BeanFactoryPostProcessor和BeanPostProcessor

  |      | BeanFactoryPostProcessor | BeanPostProcessor |
  | ---- | ------------------------ | ----------------- |
  |      | 对容器增强               | 对Bean增强        |
  |      |                          |                   |
  |      |                          |                   |
  
- [ ] spring中使用到的设计模式

  - 单例模式：单例Bean并不是实现了单例模式
  - 原型模式：bean指定为prototype
  - 工厂模式：BeanFactory
  - 工厂方法模式：FactoryBean
  - 模板模式：onRefresh、JDBC
  - 策略模式：XmlBeanDefinitionReader、PropertiesBeanDefinitionReader
  - 观察者模式：listner、event、multicast
  - 适配器模式：HandlerAdapter、advice
  - 装饰器模式：BeanWrapper
  - 责任链模式：aop会生成拦截器链
  - 代理模式：aop动态代理
  - 建造器模式：Builder结尾的对象（BeanDefinitionBuilder）


- [ ] @Index
  - 编译时生成一个需要加载的bean的文件，提升性能
- [ ] @Import
  - 将配置类加入容器
  - 如果导入的类实现了ImportSelector接口，会调用接口声明的方法，该方法会返回类全路径
  - 如果导入的类实现了ImportBeanDefinitionRegistrar接口，会调用接口声明的方法中提供的注册器来注入bean对象
- [ ] SpringMVC

  - [ ] 对MVC的理解
  - [ ] 和spring的关系
    - 父子容器关系，spring是父容器
    - mvc管理controller，spring管理service和dao
  - [ ] 执行流程
- [ ] springboot

  - [ ] springboot中为什么引入stater就能有对应的功能

    - [ ] SPI机制
    - [ ] springboot自动装配


## 微服务分布式

- [ ] 微服务概念

- [ ] 分布式概念

- [ ] Nacos
  - [ ] Nacos和Eureka的区别
  
  - [ ] nacos配置中心动态刷新的原理
  
  - [ ] 心跳机制机制
    
  - [ ] 服务注册
  
      - [ ] 服务表结构
      
      
    
    ![image-20230801073557674](../../../images/image-20230801073557674.png)
    
    - 底层的数据通信基于protocol buffer序列化协议，跨语言，跨平台
    - 可以使用protoc转换类型
    
  - [ ] 如何支持数十万服务压力
  
    - 集群和负载均衡
  
- [ ] Gateway
  - [ ] 为什么要使用Gateway
    - 接口的访问ip有变动的可能，由网关来实现服务路由
    - 对请求的客户端进行鉴权
    - 负载均衡、反向代理
    - 隐藏真实请求路径
  
- [ ] Feign

  - [ ] Feign和OpenFeign的区别

