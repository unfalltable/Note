---
title: Spring全家桶
categories: Java框架
tags: [框架]
---

# Spring5

## IOC

- IOC：控制反转，即将对象的创建个调用交给Spring管理
- DI：依赖注入

## 容器

主要通过两个接口来实现容器的创建

- BeanFactory：
  - spring核心容器，主要是内部使用
  - 使用到对象是才创建，懒加载
  - 提供getBean()方法
- ApplicationContext：
  - 继承了BeanFactory，扩展了更多功能国际化、匹配资源、发布事件、环境信息等
  - 加载配置是就创建所有的对象，体验更好但是启动慢

## Bean

**加载方式**

- XML中 \<Bean id="" class="" />
- @Component、@service、@Controller、@Repository、@Import、@Bean
- @ComponentScan(“路径”)、@Configuration
- @ImportResource(“xml名”)
- @Configuration(proxyBeanMethods =)
- FactoryBean\<T>
- 容器对象.registerBean()
- 容器对象.registerSingleton(“Bean名”, Bean对象)    不会走bean的创建、依赖注入、初始化过程
- 实现 ImportSelector、ImportBeanDefinitionRegister、BeanDefinitionRegistryPostProcessor 接口，实现对应的方法
- 构造注入

**加载控制**

- @Conditional()
- 根据一些条件决定是否加载Bean

**作用域**

- singleton：单例
- prototype：多例
- request
- session
- application

**生命周期**

- 实例化前
- 实例化后
- 初始化前
- 初始化后
- 使用
- 销毁

## 后处理器

### Bean后处理器

- AutowiredAnnotationBeanPostProcessor
  - 解析@Autowired、@Value

- CommonAnnotationBeanPostProcessor
  - 解析@Resource、@PostConstruct、@PostDestroy

- ConfigurationPropertiesBindingPostProcessor
  - 解析@ConfigurationProperties

- AnnotationAwareAspectJAutoProxyCreator
  - 解析AOP相关注解
  - 获取所有切面

    - 通过findEligibleAdvisors(目标类) 找到所有作用于目标类的切面，会将高级切面转化为低级切面

  - 创建代理

    - 通过wrapIfNecessary(目标类对象)，需要判断findEligibleAdvisors()返回的集合是否为空

### BeanFactory后处理器

- ConfigurationClassPostProcessor
  - 解析@ComponentScan、@Bean、@import、@ImportResource
- MapperScannerConfigurer
  - 解析@MapperScan
- internalConfigurationAnnotationProcessor

  - 解析@Configuration、@Bean

## AOP

**是什么**

- AOP是IOC流程中的一个扩展点
- 作用是在不通过修改源代码的基础上，对其进行功能的添加
- 体现了装饰器模式
- 代理对象一般是在初始化之后创建
- 出现循环依赖时，则会在依赖注入之前创建，并暂存二级缓存中

**使用的场景**

- 记录日志、缓存处理、spring内置的事务处理

**实现**

- ajc编译器静态代理
  - 需要添加aspec-maven-plugin
  - 原理是直接将增强方法写进类中，不使用代理，不走IOC
- Agent类静态代理
  - 需要添加JVM参数 `-Javaagent:maven仓库路径/`
  - 原理也是直接将增强方法写进类中，不使用代理，不走IOC
- JDK动态代理
  - 代理对象有实现接口则使用JDK代理，创建的代理和代理对象一个等级
  - 前16次通过反射调用方法，后续都是直接调用
- Cglib动态代理
  - 创建的子类代理对象，所以被代理对象不能是final修饰的
  - 直接调用被代理方法
  - 基于ASM 实现的，动态字节码操作
- AspectJ
  - 属于编译时增强，基于字节码操作
  - 切面多的话AspectJ性能更强


**使用**

1. 创建一个AOP处理类，添加@AspectJ 和 @Component
2. 使用@Pointcut标注一个方法，指定代理的方法
   - execution 表达式匹配方法名
   - @Annotation() 表达式匹配注解名
3. 选择对应的通知类型
   - @Before、@After、@AfterReturning、@Around、@AfterThrowing
4. 切面
   - @Advice
5. 优先级
   - @Order(值)    前置通知值小优先，后置通知值大优先

**注意**

- 代理对象无法增强静态方法
- Jdk代理和代理对象时同级关系，而cglib代理和代理对象时父子关系，所以代理对象不能是final修饰
- Jdk代理前16次都是通过反射的方式调用目标方法
- cglib代理通过代理方法直接调用目标方法

## 事务

**是什么**

- 即要么都成功，要么都失败，不能存在中间状态
- 事务一般加在Service层
- 底层使用AOP，通过TransactionInterceptor实现
- @Transaction
  - propagation传播属性
  - isolation隔离级别
  - rollbackFor
  - noRollbackFor

**事务的传播行为**

- Requirded：A有B就用A的，没有就创建
- Required_New：不管A有没有，B创建新的
- Supports：A有就用A的，没有就不用
- Not_Supports：A有就挂起，B不用事务
- Mandatory：A没有就抛异常
- Never：A有就抛异常
- Nested：A里有，B在A里的事务执行，A里没有就在里面创建

**事务失效的场景**

- 异常捕获处理
  - 即catch到异常并处理时，spring就没发现有异常，所以事务失效
  - 需要抛出才能回滚

- 抛出非运行时异常/检查异常
  - 非运行时异常如io读取异常等等，即编译阶段可以发现的异常
  - 在@Transaction标注回滚异常类型为Exception可以解决
  
- 非public方法
- 非代理对象调用被代理对象的方法
- 在事务方法中直接调用其他事务方法
  - 因为直接调用不是用到代理对象调用，其实是用的this
  - 设置@EnableTransactionManagement(exposeProxy = true)，然后使用AopContext.currentProxy()获取代理对象，然后执行方法

- 多线程下可能出现失效
- mysql没开启事务

## 内置功能

**Aware**

- BeanNameAware：注入Bean的名字
- BeanFactoryAware ：注入BeanFactory容器   
- ApplicationContextAware：
  - 接口
  - 注入ApplicationContext容器

- EmbeddedValueResolverAware：解析 $ { }

**InitializingBean**

- 容器初始化接口

# SpringWebFlux

## 简介

- 异步非阻塞的框架、响应式编程，功能和SpringMVC类似，基于Netty
- SpringWebFlux + Reactor + Netty
- 请求和响应是 ServerReuquest 和 ServerResponse
- 观察者模式 Oberver（Flow取代）
- 主要使用Flux 和 Mono
  - Flux
    - 声明多个数据流
    - 可传入数组（fromArray）、集合（fromItertr）、流（fromStream）

  - Mono
    - 声明0个或1个数据流

- 可以发送信号
  - 错误信号

- 需要订阅后才能发出数据流
  - subscribe()

- 操作符
  - map 映射
  - flatMap 将元素映射成流

# SpringBoot

## 简介

- 基于spring的框架，有内置的tomcat，可以独立运行
- 约定大于配置理念，最大的简化配置
- 核心功能在于依赖管理和自动配置
  - 依赖管理：
    - spring-boot-dependencies 几乎声明了所有开发中常用依赖的版本号，实现自动版本仲裁
  - 自动配置
    - 启动时自动加载所有的自动配置类
    - 如果不想使用默认的配置，可以直接@Bean替换底层的组件

## 配置文件

### 密码加密

- 自定义后置处理器，实现EnvironmentPostProcessor接口

## 监听器



# SpringMVC

## 解决乱码问题

- 配置全局过滤的filter

  ```xml
  <filter>
  	<filter-name>CharacterEncodingFilter</filter-name>
      <filter-class>
         org.springframework.web.filter.CharacterEncodingFilter
      </filter-class>
      <init-param>
      	<param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
      </init-param>
  </filter>
  <filter-mapping>
  	<filter-name>CharacterEncodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

## 拦截器（Interceptor）

- 自定义拦截器

  - 创建拦截器类实现HandlerInterceptor接口

    - preHandle()
      - 处理前被调用
      - 有返回值，返回false后续的方法都不会执行了
    - postHandle
      - 处理之后被调用
      - 在DispatcherServlet进行视图渲染之前调用，所以我们可以对ModelAndView对象进行操作
    - afterCompletion

  - 配置拦截器，xml或者配置类

    ![image-20220113151640290](../../../blog/source/images/image-20220113151640290.png)

# SpringCLoud

## 注册中心

Eureka

- 支持服务注册和拉取、心跳检测
- Eureka采用AP方式
- 服务提供者启动时会向Eureka注册信息，每30秒发送一次心跳
- Eureka为服务消费者提供服务提供者的信息
- 负载均衡策略
  - 默认是轮询，定时任务30s获取服务提供者信息

Nacos

- 支持服务注册和拉取、心跳检测
  - 临时实例采用心跳检测，非临时采用主动检测
  - 临时实例不正常会剔除，非临时不会

- 支持服务列表变更的消息推送模式，服务列表更新更及时
- Nacos集群采用AP模式，集群中存在非临时实例采用CP模式
- 基于Java语言实现的
- 服务分级存储模型
  - 服务 - 集群 - 实例
  - 可以进行地域划分集群
- 负载均衡策略
  - 默认是随机访问
  - 需要在配置文件中配置使用Nacos负载均衡规则
    - 服务调用优先调用本地集群，本地集群不可用时访问其他集群
- 可以在nacos控制面板中设置实例权重
  - 权重设置为0则不处理请求
- 实例可以选择临时实例或者非临时实例
- 本质上也是通过mvc调用接口实现

## 配置中心

Nacos

- 统一配置管理，实现热更新，常配置开关或者格式
- 服务拉取Nacos配置
  - 新建一个boostrapt.yml 或者 properties 文件
    - 配置Nacos地址，配置文件后缀名，模块名

  - 使用@NacosValue可以获取配置文件信息
- 配置自动刷新，优先使用nacos的配置
  - 接口上添加 @RefreshScope 实现自动刷新
  - 也可以创建一个类专门完成属性的加载
    - 类上添加 @ConfigurationProperties(prefix = “”)、@Component
    - 对应Nacos配置中的前缀
    - 创建一个变量接收这个配置的信息
- 环境隔离NameSpace 多环境配置共享
  - Nacos控制面板中可以创建命名空间，在配置文件中配置namespace，内容是命名空间自动生成的id
  - 多种配置的优先级
    - 服务名称-开发环境.yaml  > 服务名称.yaml 中 > 本地配置
  - 命名空间可以按服务来划分，各个服务使用自己的命名空间
  - 可以使用配置多配置文件，将配置信息都放在nacos上


## 负载均衡

### Ribbon

- 请求先到Ribbon，然后Ribbon取注册中心找服务地址
- 使用时往Spring容器中添加IRule对应的子类即可（全局），也可以在配置文件中配置（部分微服务）
- 默认是懒加载的，第一次访问时才会创建LoadBalanceClient，请求时间比较长
  - 可以设置饥饿加载，启动时就会创建，可以指定饥饿加载的服务器


### 规则

- RoundRobinRule：轮询
- AvailabilityFilteringRule：忽略短路和并发数过高（可以指定上限）的服务器
- WeightedResponseTimeRule：服务器响应时间越久，服务器的权重越小
- ZoneAvoidanceRule：分区，再对区内轮询
- BestAvailableRule：忽略短路的服务器，选择并发数低的服务器
- RandomRule：随机
- RetryRule：重试机制

### LoadBalance

## 远程调用

### Dubbo

#### 简介

- 服务治理框架
- 支持多协议

### Feign

#### 简介

- 是一个声明式Http客户端
- 传统的Http远程调用是使用 RestTemplate ，存在一些问题
  - 代码可读性差，编程不统一
  - 参数复杂Url难以维护
- 自定义Fegin的配置
  - Fegin运行自定义的配置来覆盖默认配置
  - 配置：
    - fegin.Longger.Level：修改日志等级
      - NONE：没任何日志
      - BASIC：一次请求的请求时间，结束时间，耗时时间等基本信息
      - HEADERS：基本信息 + 请求头信息
      - FULL：基本信息 + 请求头信息 + 请求体 + 响应体

    - fegin.codec.Decoder：响应结果的解析器，将Json 转换为 Java对象
    - fegin.codec.Encoder：请求参数编码
    - fegin.Contract：支持的注解格式，规定支持的注解，默认是SpringMVC的注解
    - fegin.Retryer：失败重试机制，默认是不重试

- 使用
  - 配置文件
  - 加入Spring容器，创建对应的配置类
    - 全局配置：@EnableFeignClient(defaultConfiguration = 配置类)
    - 局部配置：@FeignClient(value = “作用的服务”, configuration = 配置类)

#### 性能优化

- 底层的客户端，修改为支持连接池的客户端（导包 - 配置）
  - 默认是 URLConnection，不支持连接池
  - Apache HttpClient：支持连接池
  - OKHttp：支持连接池
- 日志级别最好设置为Basic

#### 最佳实践

- 继承
  - 创建一个接口写这个方法声明，再让Client和Controller去继承和实现
  - 缺点
    - 对SpringMVC不起作用，这个接口的参数列表中的映射不会被继承
    - 服务紧耦合
- 抽取
  - 将方法声明，使用到的Pojo，默认配置等抽取为一个模块
  - 缺点
    - 会将所有方法引入，多余了一些

### OpenFeign

## 网关

### Gateway

- 基于WebFlux实现d
- 拦截所有客户端请求，进行身份认证和权限校验
- 进行服务路由，负载均衡
- 请求限流
- 路由断言工厂 （predicates）
  - 配置写的字符串会被 Predicate Factory 读取并处理，转变为路由判断的条件
  - spring 提供了11个断言工厂
    - After、Before、Between、Cookie、Header、Host、Method、Path、Query、RemoteAddr、Weight
- 过滤器 （filters）
  - 当前路由过滤器
  - 默认过滤器
    - Default-filter

  - 全局过滤器
    - GlobalFilter 接口

  - 执行顺序
    - 指定Order来决定执行顺序，值相同时，先执行默认，然后局部，然后全局 

## 安全监控

### Sentinel

- 雪崩问题
  - 超时处理：一定时间没响应就返回错误信息
  - 舱壁模式（线程隔离）：限定每个业务能使用的线程数，避免tomcat资源耗尽
  - 熔断降级：断路器统计业务执行的异常比例。超出阈值则熔断该业务，拦截访问该业务的一切请求
  - 流量控制：限制业务访问的QPS，避免服务流量突增发生故障

### 流量控制

### 雪崩问题

#### 隔离

- 线程池隔离
- 信号量隔离

#### 降级

### 授权规则

- 通过过滤器给请求添加一个特殊标识的头信息，携带这个信息的请求才可以正常访问

### 规则持久化

### 线程隔离

- 线程池
  - 支持主动超时、支持异步调用
  - 线程开销大
  - 支持低扇出
- 信号量（默认实现）
  - 轻量级，开销小
  - 无主动超时和异步调用
  - 高频调用、高扇出

## 熔断降级

- 断路器
  - closed：服务正常调用，统计服务异常比例，达到阈值则熔断，进入open状态
  - open：熔断状态，禁止目标服务调用，熔断持续一段时间后进入half-open状态
  - half-open：尝试请求服务，如果能正常访问则恢复到closed状态
- 熔断策略
  - 慢调用：响应时间慢，一点数量的请求慢则熔断，可以设置响应时间阈值
  - 异常比例：在一定请求中抛异常的请求数量超过一定比例则熔断
  - 异常数：异常请求达到设置的异常数则熔断

# SpringSecurity

## 简介

### 身份验证

### 权限管理

- 按照安全规则或者安全策略控制用户只能访问被授权的资源
- 权限管理包括用户身份认证和授权，简称认证授权
  - 认证：身份认证，就是判断用户是否合法（登录、指纹、人脸等等）
  - 授权：即访问控制，对不同的用户分配不同的权限

#### 解决方案

- shiro
  - 轻量、简单、易于集成
  - 但在微服务和扩展能力没有优势

- SpringSecurity
  - 易于集成，微服务首选框架

- 自定义权限管理
  - 大项目、大公司需要，因为可以做到更安全

## 整体架构

![image-20230509162000402](../../../blog/source/images/image-20230509162000402.png)

- 认证器（Authentication）
  - AuthenticationManager主要的实现类为ProviderManager，该实现类中管理了很多AuthenticationProvider实例，用来允许实现多种认证方式
  - Authentication实现类用来保存认证以及认证成功的信息
  - SecurityContextHolder用于获取认证信息，
    - 原理是用的ThreadLocal实现与线程绑定
    - 第一次认证时的登录信息会保存到SecurityContextHolder中，使用完后会保存到session中，然后清空SecurityContextHolder
    - 之后再请求时会先查session，如果存在登录信息则放入SecurityContextHolder中，用完再放回session并清空
    - 方便在各个层级使用
- 授权器（Authorization）
  - AccessDecisionManager：访问决策管理器，用来决定此次访问是否被允许
  - AccessDecisionVoter：访问决策投票者，会检查用户是否有对应的角色并进行投票
  - ConfigAttribute：用来保存授权时的角色信息

### 认证 & 原理

- 本质上是用的一系列的spring提供的过滤器filter，并不是原生的filter
- spring提供了DelegatingFilterProxy，将spring的filter转换为原生的filter，使用了代理模式实现
- 使用了FilterChainProxy，管理了spring的filter，并使其按一定的顺序执行，使用了责任链的设计模式
- 通过FilterChainProxy嵌入到原生的filter过滤链中，由FilterChainProxy统一管理SecurityFilter

### 过滤器

- 由SpringSecurityFilterChain管理了所有需要使用的过滤器

![image-20230513114552313](../../../blog/source/images/image-20230513114552313.png)

- 默认会加载这15个filter

### 自定义认证

### 密码加密

### Remenber Me

### 会话管理

### CSRF

### 跨域CROS

### 异常处理

### 授权 & 授权模型

# OAuth2 & Jwt

### 单点登录

# 注解

**@Bean**

- 向容器注册组件
- 只能标注方法
- @Bean标注的方法不支持重载，只有参数最多的方法会执行

**@Autowired**

- spring提供的，先ByType后ByName，对象必须存在
- 失效的情况
  - 创建对象后创建BeanFactoryPostProcessor

- scope失效的情况
  - 单例对象中注入多例对象，多例会失效
    - 可以添加@Lazy解决（原理是代理）
    - 可以在@Scope(value = “prototype”, proxyMode = ScopedProxyMode.TARGET_CLASS)
    - 使用ObjectFactory<多例对象>解决
    - 使用容器对象的getBean方法获取多例对象


**@Qualifer**

- 不注入

**@Resource**

- 有JDK提供，ByType和ByName，可以指定Name

**@Value**

- 

**@Import**

- 向容器中创建组件，默认组件名字是全类名

**@Mapper**

- 扫描mapper包，获取其元信息，判断是否是注解
- 获取Bean定义，传入MapperFactoryBean类作为参数

**@Configuration**

- 被标注的类相当于一个工厂类，类中@Bean标注的方法相当于工厂方法
- 会给标注的类生成代理对象，目的是保证bean的单例特性
  - 默认是单例模式，可设置为多例


**@Nullable**

- 

**@Indexed**

- 在编译时将标注了该注解的Bean，会生成/META-INF/spring.components文件并加入其中
- 扫描包时会先查这个文件并加载其中的Bean
- 没有再走包sao'miao

**@Order**

- 数字越小的优先级越大

**@Lazy**

- 标注类上是延迟创建
- 标注方法参数或成员变量上，

**@primary**

- 优先注入该Bean

**@Conditional**

- 条件装配，有很多子实现
- 写在方法上有先后执行
- 设定条件，按条件装配容器

**@ImportResource**

- 导入资源（xml）

**@RequestMapping**

- 映射浏览器的请求

**@ResponseBody**

- 将方法返回的内容写给浏览器

**@RestController**

- 等于 @ResponseBody 加 @Controller

**@EnableConfigurationProperties**

- 启用 @ConfigurationProperties 的功能

**@ConfigurationProperties**

- 将.property文件中的键值信息和Bean的属性进行绑定

@RequestMapping

- method  请求方式
  - method = RequestMethod.请求方式
- params   指定限制请求参数的条件，支持简单的表达式
- 页面跳转
  - 直接返回字符串
    - 进行视图跳转
  - 通过ModelAndView对象返回
    - new一个ModelAndView对象进行返回

@RequestBody

- 将前端传递过来的Json数据映射到类上

@ResponseBody

- 将后端返回的数据封装为Json格式返回

@RequestParam

- 当请求的参数名称与业务方法上形参的名字不一致时，使用该注解命名
  - value                客户端请求的参数名
  - required           是否必须包含value值，默认时true，没有则报错
  - defaultValue    默认值

@PathVariable

- 在请求映射上添加占位符(请求方式为get)
  - @RequestMapping("/user/**{username}**");
    - 可以用method 来区分请求方式
  - 形参前加**@PathVariable**(value="username")
    - username 的值会自动赋值给形参

@RequestHeader

- 获得请求头信息
  - value       请求头名字
  - required  是否必须携带此请求头

@CookieValue

- 可以获得指定的Cookie的值

@RestController

- @ResponseBody  +  @Controller

@CrossOrigin

- 处理Ajax请求的跨域问题

@RequestPart

- 用于接收MultipartFile等复杂类型的入参

# 工具类

DigestUtils

- md5加密