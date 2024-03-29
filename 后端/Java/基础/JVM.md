---
title: JVM虚拟机
categories: Java基础
tags: [基础]
---

## 简介

## JVM内存模型

### 堆

- 存放new出来的对象、字符串常量池、静态变量
  - 字符串常量池是在1.7的时候移入堆中，之前都是在方法区中
- 堆内存由年轻代和老年代组成

### 栈

- 主要存放的是线程，每个线程开辟一块内存空间，称为线程栈
  - 最多占用1m内存
- 线程栈中由多个栈帧组成，栈帧是调用的每个方法
  - 栈帧包括：，局部变量，
    - 操作数栈：方法运行期间数据的操作空间 
    - 动态链接：符号引用转变为直接引用
    - 方法出口：记录方法结束时下一句执行的代码的地址

### 本地方法栈

- 存放本地方法（native method）
  - 本地方法是底层方法，主要是c和c++语言编写的

### 方法区

- 存放类的信息，静态方法信息、类加载器，全局变量，常量
  - 类信息就是常量池就是一张表，里面有类名，方法名，参数类型，字面量等信息
  - 当类被加载时，常量池的信息会放入运行时常量池，里面的符号地址变为真实地址
- 1.7之前称为永久代，1.8之后称为元空间并开始使用物理（本地）内存，不受JVM管理了
- 类加载器将类信息加载进元空间，当类加载器加载的所有类不再被引用时，gc会将类加载器、加载的类、元空间的类信息释放
- 在1.7之前永久代空间不足会触发堆的垃圾回收

### 程序计数器

- 存放下一条要运行的代码的地址
- 便于线程切换
- 通过寄存器实现，是线程私有的，不会出现内存溢出

### 字符串常量池

- StringTable，1.7之后在堆中，是不能扩容的，底层是一个hash表

- 编译期优化

```java
String s = "ab"    //是加到字符串常量池中的
String s1 = "a" ,String s2 = "b"   
    //s1+s2     
    //相当于 new StringBuilder().append("a").append("b").toString()
    //new出来的是在堆中的
    
"a" + "b"    //如果字符串常量池中有则引用，没有则创建
"a" + "b"    //javac在编译期间已经确定结果一定为”ab"
s1 + s2      //javac在编译期间已经不能确定结果一定为”ab"，因为s1和s2是变量
```

- 常量池中的字符串仅是符号，第一次用到时才变为对象

- 利用串池的机制。来避免创建重复的字符串对象

- 字符串拼接的原理时StringBuilder

- 字符串常量拼接的原理时编译期优化

- 可以使用intern方法，将字符串对象放入串池

  - 1.8时，串池有则不放入，没有则放入，会把串池中的对象返回
  - 1.6时，串池有则不放入，没有则把对象赋值一份，放入串池，会把串池中的对象返回

- 设置StringTable桶的个数减少哈希冲突

  - -XX:StringTableSize=(1009 - ....)   最少是1009个

- 将字符串对象入池优化

### 总结

- 栈、本地方法栈、程序计数器是线程私有的
- 堆、方法区/元空间是线程共享的

## Java内存模型

- Java 的并发采用的是共享内存模型，Java 线程之间的通信总是隐式进行
- JMM定义的是线程本地内存和主内存的抽象关系，用于屏蔽各种硬件和操作系统的内存访问差异
  - 可见性：当线程修改了共享变量，其他线程能立即知道改变更
  - 原子性：要么都发生，要么都不发生
  - 有序性：按顺序执行

- 本地内存是一个抽象的概念，并不真实存在，是一种规范，它包括了缓存，写缓冲区，寄存器、编译器
  - 写缓冲区：可以保证指令流水线持续运行，它可以避免由于处理器停顿下来等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，可以减少对内存总线的占用
- 从 java 源代码到最终实际执行的指令序列，会分别经历三种重排序
  1. 编译器优化重排序
     - 编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序
  2. 指令级并行的重排序
     - 如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序
  3. 内存系统的重排序
     - 由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行
- happens-before
  - 先行发生原则，判断数据是否存在竞争，线程是否安全
    - 次序规则：一个线程内的前一步操作后的结果对后一步来说一定是可见的
    - 锁定规则：同一个锁的unlock操作一定先于lock操作
    - volatile变量规则：写操作后的数据一定是可以被读到的
    - 传递规则：如果A先行于B，B先行于C，那么A一定先行于C
    - 线程启动规则：Thread对象的start方法先行于线程中的所有操作
    - 线程中断规则：interrupt方法先行于interrupted方法才能检测出线程中断事件
    - 线程终止规则：线程中的所有操作都先行于线程的终止检测 isAlive
    - 对象终结规则：初始化方法的执行一定先行于终止方法finalizefang'fa


## 类加载子系统

### 类定义

- 魔数与class文件的版本
- 常量池
  - 变量的属性、类型和名称
  - 方法的属性、类型和名称
- 访问标志
- 类索引、父类索引、接口索引
- 字段表属性
- 方法表属性
- 属性表属性

### 类加载过程

1. 加载
   - 根据类的全限定名，将类的信息加载进方法区
   - 在堆中生成一个对象指向该类，作为方法区的数据访问入口
   - 如果这个类没有父类，会先加载他的父类
2. 验证
   - 验证类是否符合JVM规范，例如检查class文件的魔数，符号引用验证
3. 准备
   - 为静态变量分配空间，如果静态变量是基本数据类型的话为其设置默认值
4. 解析
   - 将类中的符号引用解析为直接引用
5. 初始化
   - 初始化即调用 <cinit>()V 方法，虚拟机会保证这个类的构造方法的线程安全
   - 为静态变量赋值
   - 初始化是懒惰的，使用时才初始化
     - 类在main方法中
     - 类的静态变量或静态方法被访问
     - 子类初始化时父类会在其之前初始化
     - 子类访问父类的静态变量，只会触发父类初始化
     - new、class.forName
6. 使用
7. 销毁

### 类加载器

- Bootstrap ClassLoader 一级
  - 加载 JAVA_HOME/jre/lib 下的类，无法直接访问
  - C语言实现的，所以打印时为null
- Extension ClassLoader 二级
  - 加载 JAVA_HOME/jre/lib/ext 下的类，显示为null
- Application ClassLoader 三级
  - 加载 classpath下的类，线程上下文加载器
- 自定义类加载器 四级
  - 加载自定义位置的类，通过接口实现，常见于tomcat中
  - 步骤：
    - 继承ClassLoader父类
    - 要遵从双亲委派机制，重写findClass方法
    - 读取类文件的字节码
    - 调用父类的defineClass方法来加载类
    - 使用者调用该自定义类加载器的loadClass方法

### 双亲委派模式

- JDK1.2之后才有该模型
- 避免类的结构混乱，保护核心类不被篡改，安全
- 就是调用类加载器的loadClass方法，所经历的过程
  - 检查该类是否已经加载，有加载则返回
  - 没加载找上级，让上级loadClass，上级有加载则返回
  - 没加载再找上级，如果没上级了，则找BootstrapClassLoader，有则返回
  - 每一层都找不到，调用findClass方法（每个类加载器自己扩展）来加载

### 字节码执行引擎

- 执行方法区中的字节码文件
- 修改程序计数器的数据
- 垃圾收集线程

### 解释器

- 将字节码解释为机器码，下次即使遇到相同的字节码，仍会执行重复的解释
- 将字节码解释为针对所有平台都通用的机器码

### JIT即时编译器

- 将字节码编译为机器码，存入CodeCache中，下次遇到相同的代码，直接执行缓存中的，无需重复解释
- 根据平台类型，生成平台特定的机器码

## 垃圾回收

### 简介

- gc root 对象不可被回收
- GCRoots对象包括：虚拟机运行过程中的核心类对象，虚拟机栈中的局部变量表引用的对象，方法区中类静态属性引用和常量引用对象，本地方法栈中的引用的对象，被加锁的对象，活动线程中引用的对象
- 引用队列
  - ReferenceQueue

### 引用类型

- 强引用
  - 在堆中创建对象并用一个变量引用为强引用
- 软引用
  - SoftReference<>(软引用, 引用队列)
  - 垃圾回收过后内存仍然不足则会回收软引用，前提是没用强引用引用它
  - 当软引用的对象被回收时，软引用会进入引用队列
- 弱引用
  - WeakReference<>(软引用, 引用队列)，用于解决内存泄漏问题
  - 垃圾回收过后会回收软引用，前提是没用强引用引用它
  - 当弱引用的对象被回收时，弱引用会进入引用队列
  - full gc 会清除所有弱引用对象
- 虚引用
  - 也称幽灵引用，一旦创建就被回收了，所以需要队列cun'chu
  - 当虚引用的对象被回收时，虚引用会进入引用队列
  - 引用队列会定期查看引用队列中是否有虚引用
  - 有虚引用的话会调用虚引用的clean方法调用unsafe.freeMemory()方法释放直接内存
- 终结器引用
  - 当对象重写了终结方法finalize()并且没用强引用时，它就会被垃圾回收
  - 对象没有强引用时，JVM会创建终结器引用引用该对象，并将终结器引用加入引用队列
  - finalizeHandler会定期查看引用队列中是否有终结器引用
  - 有终结器引用时会调用引用对象的finalize方法

### 回收算法

- 引用计数法（已过时）
  - 当对象没有被强引用所指向时在gc时就进行清除
  - 有A引用B，B引用A无法清除的问题
- 标记+清除算法
  - 标记是使用**可达性分析算法**（沿着根对象寻找），**三色标记法**标记存活对象
  - 记录起始位置和结束位置，新数据直接覆盖旧数据，速度快
  - 空间碎片多不连续
- 标记+清除+整理算法（老年代）
  - 可以进行整理解决标记清除算法产生碎片的问题，
  - 但移动对象会涉及地址改变，速度慢
- 复制算法（新生代）
  - 将不回收的数据迁移到另一个相同大小的区域，然后交换这两个区域
  - 效率高，不会产生内存碎片，但会占用双倍的的内存空间，浪费空间

### 分代回收

- 新生代
  - 伊甸园 + 幸存区from + 幸存区to
  - 8：1：1
  - 伊甸园满了触发 minor gc
    - minor gc 会暂停所有线程STW
    - 把存活的对象复制到幸存区to，且寿命+1
    - 然后交换幸存区to和幸存区from的位置
    - 当幸存区的对象寿命超过阈值（默认是15），就将其加入老年代
  - 使用的是复制算法
  
- 老年代
  - 使用标记清除+整理算法
  
  - 当老年代内存不足时触发minor gc，如果空间仍不足则触发full gc
  
  - 触发老年代GC的时机
  
    - 老年代可用内存小于新生代全部对象的大小，如果没开启空间担保参数，会直接触发Full GC，所以一般空间担保参数都会打开
  
    - 老年代可用内存小于历次新生代GC后进入老年代的平均对象大小，此时会提前Full GC
  
    - 新生代Minor GC后的存活对象大于Survivor，那么就会进入老年代，此时老年代内存不足full gc
  
    - 就是“-XX:CMSInitiatingOccupancyFaction”参数
  
      如果老年代可用内存大于历次新生代GC后进入老年代的对象平均大小，但是老年代已经使用的内存空间超过了这个参数指定的比例，也会自动触发Full GC。
    
  - 对象直接进入老年代的情况
  
    - 大对象直接进入老年代
      - 需要设置-XX:PretenureSizeThreshold，只在Serial和ParNew收集器下有效
      - 单位是字节
    - 对象动态年龄判断
      - 当Survivor区内的对象总大小超过50%（可以设置），那么这些对象中年龄最大的都会直接进入老年代，对象动态年龄判断一般在minor gc后触发
    - 年龄15岁，cms为6岁的会进入老年代

### 垃圾收集器

#### CMS（Concurrent mark sweep)

- 注重响应时间，但对cpu压力大
- 可以并发标记，并发清除，存在并发漏标,不会STW
  - 使用标记清除算法，有内存碎片
  - **三色标记法**解决并发漏标，会STW
- 有并发失败兜底策略（Failback Full GC）
- 有一些Bug
  - 依然会产生漏标
  - 浮动垃圾上升老年代的Bug

#### G1

- jdk1.9默认
- 注重吞吐量和响应时间
  - 超大堆内存，将堆划分为大小相等的Region
    - 每个区都可以充当（eden、survivor、old、humongous(用于存放大对象)）
  - 单区域是标记+整理算法，区域之间是复制算法
  - 当新生代大小超过阈值时触发新生代回收，使用标记复制法到幸存区（跨区域）
  - 当老年代占堆内存45%是触发**并发标记**，采用**原始快照法**解决漏标问题，处理漏标记录依然需要STW
  - 并发标记结束后。开始**混合收集**，会选择回收价值高的老年代，eden，survivor，可能需要执行多次
  - 并发失败兜底策略（Failback Full GC）

### 可达性分析

- 从GC Roots对象为起点开始向下搜索引用的对象，找到的都标记为非垃圾对象，其余没找到的都标记为垃圾对象
  - GC Roots：线程栈中的本地变量、方法区中的静态变量、本地方法栈的变量等等

- 不可达的对象将暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少**要经历两次标记**过程

- 如果对象在进行可达性分析后发现没有与 GC Roots 相连接的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 **finalize**() 方法。
- 当对象没有覆盖 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”，直接进行第二次标记。
- 如果这个对象被判定为有必要执行 finalize() 方法，那么这个对象将会放置在一个叫做 F-Queue 的队列之中，并在稍后由一个由虚拟机自动建立的、低优先级的 Finalizer 线程去执行它。
- 这里所谓的“执行”是指虚拟机会触发这个方法，但并不承诺会等待它运行结束，因为如果一个对象在 finalize() 方法中执行缓慢，将很可能会一直阻塞 F-Queue 队列，甚至导致整个内存回收系统崩溃。
- 在HotSpot的实现中，是使用一组称为`OopMap`的数据结构来达到直接得知哪些地方存放着对象引用，在类加载完成的时候，HotSpot就把对象内什么偏移量上是什么类型的数据计算出来，在JIT编译过程中，也会在特定的位置记录下栈和寄存器中哪些位置是引用。这样，GC在扫描时就可以直接得知这些信息了。
- OopMap数据结构存储GCRoot对象，但是随着系统的运行会导致OopMap会逐渐变大，所以也并不会存储所有的GCRoot对象，而是在一个所谓的安全点进行记录GCRoot对象。
- 实际上，HotSpot的确没有为每条指令都生成OopMap，只是在“特定的位置”记录了这些信息，这些位置称为安全点（Safepoint）——即程序执行时只有在到达安全点时才能暂停。
- Safepoint的选定既不能太少以致于让GC等待时间太长，也不能过于频繁以致于过分增大运行时的负荷。所以，安全点的选定基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的——因为每条指令执行的时间都非常短暂，程序不太可能因为指令流长度太长这个原因而过长时间运行，“长时间执行”的最明显特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等，所以具有这些功能的指令才会产生Safepoint。
- 对于Sefepoint，另一个需要考虑的问题是如何在GC发生时让所有线程（这里不包括执行JNI调用的线程）都“跑”到最近的安全点上再停顿下来。
- 这里有两种方案可供选择：抢先式中断（Preemptive Suspension）和主动式中断（Voluntary Suspension）
  - 抢先式中断不需要线程的执行代码主动去配合，在GC发生时，首先把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它“跑”到安全点上。现在几乎没有虚拟机实现采用抢先式中断来暂停线程从而响应GC事件。
  - 主动式中断的思想是当GC需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的，另外再加上创建对象需要分配内存的地方。
- 使用Safepoint似乎已经完美地解决了如何进入GC的问题，但实际情况却并不一定。Safepoint机制保证了程序执行时，在不太长的时间内就会遇到可进入GC的Safepoint。但是，程序“不执行”的时候呢？所谓的程序不执行就是没有分配CPU时间，典型的例子就是线程处于Sleep状态或者Blocked状态，这时候线程无法响应JVM的中断请求，“走”到安全的地方去中断挂起，JVM也显然不太可能等待线程重新被分配CPU时间。对于这种情况，就需要**安全区域（Safe Region）**来解决。
- 安全区域是指在一段代码片段之中，引用关系不会发生变化。在这个区域中的任意地方开始GC都是安全的。我们也可以把Safe Region看做是被扩展了的Safepoint。
- 在线程执行到Safe Region中的代码时，首先标识自己已经进入了Safe Region，那样，当在这段时间里JVM要发起GC时，就不用管标识自己为Safe Region状态的线程了。在线程要离开Safe Region时，它要检查系统是否已经完成了根节点枚举（或者是整个GC过程），如果完成了，那线程就继续执行，否则它就必须等待直到收到可以安全离开Safe Region的信号为止。

### 三色标记法

- 没处理的对象为白色，孩子对象未处理的为灰色，自己和孩子对象都处理了为黑色

- 记录标记过程中的变化，主要有两种解决方法

- Incremental Update **CMS解决方案**

  - 只要引用发生改变，被改变引用的对象需要被记录
  - 通过写屏障，黑->白 时 黑变灰
  - 有BUG
    - 依然会产生漏标，当黑变灰后另一个垃圾回收线程将其又变黑，白漏标
      - CMS后续解决方案是再从头到尾扫一边，虽然有优化但效率还是低

  

- Snapshot At The Beginning, SATB **G1解决方案** （**原始快照法**）

  - 记录标记过程的新增对象
  - 记录被删除引用关系的对象
    - 灰->白 断开了，记录白，下次排查，有黑->白不清除，反之清除
    - 但是判断白是否有引用比较困难，但是G1每个分区头部有记录哪些分区引用了当前分区，Rset（记忆集）

- 最后在STW时处理这些记录

## 对象

### 组成

​		对象头（markword）+ 类型指针（classpointer） + 数据（data） + （数组Length） + 对齐区（padding）

- markword：
  - 32位虚拟机中占4字节，64位占8字节
  
  |                             结构                             |   状态   |
  | :----------------------------------------------------------: | :------: |
  | hashcode:                      \| age:    \| biased_lock:  0 \| 01 | 普通状态 |
  | thread:       \| epoch:     \| age:    \| biased_lock:  1 \| 01 | 偏向状态 |
  | ptr_to_lock_record:                                                  \| 00 | 轻量级锁 |
  | ptr_to_heavyweight_monitor:                                \| 10 | 重量级锁 |
  | 垃圾收集器信息                                                \| 11 |  GC状态  |
  
- classpointer：
  - 表示这个对象属于哪个类型，正常情况下4个字节，因为经过压缩
  - 当系统内存超过32g将会膨胀到8个字节

- data：存放对象数据

- 如果对象是数组还有一个Length，4字节

- padding：如果整个对象不能给8整除，补齐，追求效率

### 定位

- 直接指针引用
  - 变量指向堆中对象，对象中的类型指针指向方法区中类型
  - 垃圾回收时变量指向要改变

- 句柄
  - 变量指向一组指针，这组指针一个指向堆中对象，一个指向方法区中类型
  - 好处是对象小，垃圾回收时不用频繁改动变量指向，改动的是指针组中的指向


### 创建

- 使用new关键字
- 使用Class.newInstance
- 使用Constructor.newInstance
- 使用Clone方法
  - 对象必须实现Cloneable接口，重写clone方法，调用父类的clone方法
  - 在Object中这个方法是protected的，重写才能调用
- 使用反序列化
  - 对象需要实现Serializable接口

## 动态绑定

- JVM 的方法调用指令有五个，分别是：

  - invokestatic：调用静态方法；

  - invokespecial：调用实例构造器构造方法、私有方法和父类方法；

  - invokevirtual：调用虚方法；

  - invokeinterface：调用接口方法，运行时确定具体实现；

  - invokedynamic：运行时动态解析所引用的方法，然后再执行，用于支持动态类型语言。

- 其中，invokestatic 和 invokespecial 用于静态绑定，invokevirtual 和 invokeinterface 用于动态绑定。可以看出，动态绑定主要应用于虚方法和接口方法。

- 静态绑定在编译期就已经确定，这是因为静态方法、构造器方法、私有方法和父类方法可以唯一确定。这些方法的符号引用在类加载的解析阶段就会解析成直接引用。因此这些方法也被称为非虚方法，与之相对的便是虚方法。

- 虚方法的方法调用与方法实现的关联（也就是分派）有两种，一种是在编译期确定，被称为静态分派，比如方法的重载；一种是在运行时确定，被称为动态分派，比如方法的覆盖。对象方法基本上都是虚方法。

- 这里需要特别说明的是，final 方法由于不能被覆盖，可以唯一确定，因此 Java 语言规范规定 final 方法属于非虚方法，但仍然使用 invokevirtual 指令调用。

- 静态绑定、动态绑定的概念和虚方法、非虚方法的概念是两个不同的概念。

- 以 invokevirtual 指令为例，在执行时，大致可以分为以下几步：

  1. 先从操作栈中找到对象的实际类型 class
  2. 找到 class 中与被调用方法签名相同的方法，如果有访问权限就返回这个方法的直接引用，如果没有访问权限就报错 java.lang.IllegalAccessError 
  3. 如果第 2 步找不到相符的方法，就去搜索 class 的父类，按照继承关系自下而上依次执行第 2 步的操作；
  4. 如果第 3 步找不到相符的方法，就报错 java.lang.AbstractMethodError 

  可以看到，如果子类覆盖了父类的方法，则在多态调用中，动态绑定过程会首先确定实际类型是子类，从而先搜索到子类中的方法。这个过程便是方法覆盖的本质。

- 商用虚拟机为了保证性能，通常会使用虚方法表和接口方法表，而不是每次都去判断。以虚方法表为例，虚方法表在类加载的解析阶段填充完成，其中存储了所有方法的直接引用。也就是说，动态分派在填充虚方法表的时候就已经完成了。

- 在子类的虚方法表中，如果子类覆盖了父类的某个方法，则这个方法的直接引用指向子类的实现；而子类没有覆盖的那些方法，比如 Object 的方法，直接引用指向父类或 Object 的实现。

## 直接内存

- 分配回收成本较高，但读写性能高，不受JVM内存回收管理

- NIO中的 ByteBuffer对象.allocateDirect(1024) 可以申请直接内存
  - 这个对象创建时是调用的Unsafe对象的allocateMemory()和setMemory() 方法完成直接内存的分配
  - 当这个ByteBuffer对象被回收时，然后会由ReferenHandler线程通过虚引用类型 Cleaner.clean 方法调用unsafe.freeMemory()方法来回收直接内存
- 推荐使用unsafe对象手动的管理直接内存

## 优化

### 逃逸分析

- JVM将执行状态分成了5个层次
  - 0层：解释执行
  - 1层：使用c1即时编译器执行，不带profiling
  - 2层：使用c1即时编译器执行，带基本的profiling
  - 3层：使用c1即时编译器执行，带完全的profiling
  - 4层：使用c2即时编译器执行
- profiling是指在运行过程中收集一些程序执行状态的数据，例如方法的调用次数，循环的回边次数
- 对于不常用的代码，采取解释执行，对于热点代码，采取编译执行

### 方法内联

- 如果被调用的方法是热点方法，且其内部代码不长时，会进行内联
- 所谓的内联就是把调用的方法内部的代码拷贝至调用者的位置上，还会进行常量折叠

### 字段优化

- 尽量使用局部变量，而不使用成员变量和静态成员变量

### 反射优化

- 当通过反射调用方法超过膨胀阈值时，会转换成类调用
