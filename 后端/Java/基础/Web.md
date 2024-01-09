---
title: Web
categories: Java基础
tags: [基础]
---

# IO模型

|      | BIO              | NIO                       | AIO              | IO多路复用     |
| ---- | ---------------- | ------------------------- | ---------------- | -------------- |
| 类型 | 同步阻塞         | 同步非阻塞                | 异步非阻塞       | 异步阻塞       |
| 组成 | 字节流/字符流    | Buffer、Channel、Selector |                  |                |
| 场景 | 连接数少，短链接 | 连接数多，流量小          | 连接数多，流量大 | 连接数多，流量 |

## BIO

### 分类

字节流

- FileInputStream
- FileOutputStream 

字符流

- FileReader
  - 可以读一个字符或一个字符数组

- FileWriter
  - 可以写一个字符或一个字符数组或字符数组的一部分

缓冲流

- 缓冲流自带8kb的缓存区，可以提高原始字节流读写数据的性能

  - 字节缓存流：
    - BufferedInputStream / BufferedOutputStream


  - 字符缓冲流：
    - BufferedReader / BufferedWrite
    - 可按行读取


转换流

- InputStreamReader
  - 可以解决字符流读取不同编码乱码的问题
  - 可以指定编码把原始字节流转换成字符流，解决乱码
- OutputStreamWriter
  - 按指定编码把字节输出流转换成字符输出流

序列化

- 对象输入流，反序列化
  - ObjectInputStream
- 对象输出流，序列化
  - ObjectOutputStream

打印流

- 字节输出流
  - PrintStream 
- 字符输出流
  - PrintWriter 

数据流

- 数据输入流
  - DataInputStream
- 数据输出流
  - DataOutputStream

### 弊端

- 一个Socket会创建一个线程，线程间的竞争和上下文切换开销很大
- 每个线程还会占用栈空间和CPU资源
- Socket等待时浪费系统资源
- 无法支持高并发场景，可用线程池优化，但并不能真正的解决问题

## NIO

### 简介

![image-20220310000437904](../../../blog/source/images/image-20220310000437904.png)

- Buffer缓冲区
  - 是一个内存块，底层是一个数组，负责数据的读取和写入
  - 固定容量，不能修改，读写需要指定操作数据的大小，不能超过容量
  - 可以标记一个位置，然后通过reset方法跳到标记的位置
  - 内部clear方法不会清除数据，它只会把position的位置设为0
  - 可以分配直接内存给Buffer
- Channel管道
  - 是一个接口
  - 一个Channel对应一个Buffer
  - 负责传输，可以非阻塞的读写，也支持异步读写，双向传输
- Selector选择器
  - 一个Selector对应多个Channel

## AIO

## IO多路复用

- 传统的select和poll只会通知用户进程有Socket就绪，但不确定具体是哪个Socket，需要遍历
- IO多路复用epoll则会通知用户进程有Socket就绪并写入用户空间

### Select

- 使用一个bitmap用来表示哪几个文件描述符是被监听的，大小是1024位，
  - 每次与内核交互都需要传递这个bitmap，开销大且不好修改
  - 位图不能复用，每次使用前需要将位图初始化
- 执行select函数是会将用户态的bitmap复制到内核态中，交给内核来判断事件，是一个阻塞函数，当事件有变动时会将描述符对应的事件置位，然后返回
  - 开销比较大，而且当有事件就绪时不能明确知道是哪个事件，每次都需要遍历

### poll

- 封装了一个对象pollfd，可复用
- 执行poll函数是会将用户态的pollfd复制到内核态中，交给内核来判断事件，当事件有变动时会将pollfd中的revents置位为1，然后返回
  - 每次都要将就绪的事件置位后拷贝到内核空间，开销还是比较大的
  - 当有事件就绪时不能明确知道是哪个事件，每次都需要遍历

### epoll

- epoll使用一个文件描述符管理多个描述符，将用户关心的事件放到内核中的一个事件表，这样在用户空间和内核空间只需要复制一次且只需要保存一个fd的引用即可，开销小
- 在内核态中，用户态通过一个fd就可以找到内核态中epoll
- 有一个监听列表、就绪队列和等待队列
  - 监听列表是红黑树结构的
  - 等待队列保存的是调用epoll_wait的进程
- epoll对文件操作符的操作有两种触发模式
  - LT（水平触发）：
    - 事件就绪后，用户可以选择处理或不处理，如果用户不处理，那么下次调用epoll_wait方法是还会将未处理的事件返回，即不会删除就绪列表中未处理的事件
  - ET（边缘触发）：
    - 事件就绪后，用户必须处理，因为就绪列表在返回后已经清空了

## 网络编程

### Socket

- 客户端套接字，是两台机器间通信的端点
- 使用构造方法和服务器创建连接

### ServerSocket

- 服务端套接字
- 使用构造方法侦听并接收连接服务端套接字的连接	

# JavaWeb

## Tomcat

### 部署方式

1. 把目录拷贝到tomcat目录下的webapps
   - 访问的路径就是项目的名称
   - 可以把项目压缩成war包，拉入webapps就会自动创建项目

2.  在tomcat/conf/server.xml文件中配置项目 
   - `<context path="虚拟路径" doBase="实际地址" /> `
   - 不安全，一般不使用

3. 在tomcat/conf/Catalina/localhost下新建一个xml文件 
   - `<context path="虚拟路径" doBase="实际地址" /> `
   -  热部署

## Servlet

### 执行原理

​    1.当服务器接受到客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资		源路径
​    2.查找web.xml文件，是否有对应的<url-pattern>
​    3.找到对应的<servlet-class>全类名
​    4.tomcat会将字节码文件加载进内存，并为其创建对象
​    5.调用其方法。

### 生命周期

​    创建时：执行init方法，只执行一次
​        默认情况下，第一次被访问时，servlet被创建，并执行init
​        配置servlet的创建时机：（在xml中配置）
​            <load-on-`startup`>num</load-on-startup>
​                num大于等于0在服务器创建时启动
​                num小于0在第一次访问时启动
​        注意：
​            init只执行一次，说明Servlet在内存中只有一个对象，即是单例的
​            多个用户同时访问时，可能存在线程安全问题
​            尽量不要在Servlet中定义成员变量，即使定义了，也不要修改值。
​    提供服务：执行service方法，执行多次
​        每次访问Servlet时会执行
​    被销毁：执行destroy方法，执行一次
​        服务正常关闭才会调用，会在销毁之前调用，一般用于释放资源

### 体系结构

​    HttpServlet extends GenericServlet extends Servlet
​        GenericServlet:
​            将Servlet的方法做了默认实现，之间Service()做了抽象
​        HttpServlet:
​            对http协议的一种封装，简化操作
​                重写doGet和doPost.实际上也是Service方法,只是区别了提交方式

### 缺点

​    每次创建servlet都需要写xml配置信息
​    解决：
​        在servlet3.0中支持注解配置，不需要配置xml了
​            @WebServlet(urlPatterns="资源路径")
​            可以定义多个资源路径 {"/a","/b"....}