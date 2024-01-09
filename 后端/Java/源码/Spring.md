---
title: Spring
categories: 源码
tags: [源码]
---

# Spring

## IOC

## AOP

### JDK代理

- 被代理对象需要实现接口

- JDK代理和被代理对象同级，兄弟关系

- JDK代理前16次通过反射调用方法，后续都是

```java
ClassLoader loder = 被代理类.class.getClassLoader();
//被代理对象
Target target = new Target();
接口 proxy = (接口) Proxy.newProxyInstance(loader, new Class[]{接口.class}, new InvacationHandler(){
@Override
    public Object invoke(Object proxy, Method method, Object[] args) throw Throable{
        //执行增强
        //调用被代理对象方法
        Object result = method.invoke(target, args);
        return result;
    }
});
```

### CGLIB代理

- CGLIB代理和被代理对象是父子关系，所以被代理对象不能是final修饰的
- CGLIB可以通过方法代理直接调用被代理方法，不走反射
- 被代理对象的方法是final修饰的话不会有增强效果

```java
//被代理对象
Target target = new Target();
Target proxy = (Target) Enhancer.create(Target.class, new MethInteceptor(){
    @Override
    public Object intercept(Object p, Method method, Object[] args, MethodProxy methodProxy) throw Throwable{
        //执行增强
        
        //调用被代理对象方法（反射）
        Object result = method.invoke(target, args);
        //调用被代理对象方法（直接调用）(需要被代理对象)（Spring使用）
        Object result = methodProxy.invoke(target, args);
        //调用被代理对象方法（直接调用）(不需要被代理对象)
        Object result = methodProxy.invokeSuper(p, args);
        
        return result;
    }
});
```

## 容器创建（Refresh方法）

1. prepareRefresh
   - 创建Environment对象
   - 为@Value注入信息
2. obtainFreshBeanFactory
   - 获取或创建BeanFactory
   - 通过BeanDefinition把对象信息存放在map中
   - 创建需要的对象
3. prepareBeanFactory
   - 初始化BeanFactory中的成员变量
   - 解析Spel表达式
   - 注册类型转换器，解析#{}
   - 进行依赖注入
   - 准备后处理器集合beanPostProcessors         
4. postProcessBeanFactory
   - web环境下的ApplicationContext要利用它注册新的scope，完善Web下的BeanFactory
5. invokeBeanFactoryPostProcessors
   - 对BeanFactory做扩展
   - 解析@Configuration、@Bean、@Impor、@PropertySource
6. registerBeanPostProcessors
   - 创建并加入更多的后处理器，主要看BeanDefinitionMap中对象是否实现了PostProcessor接口，加入beanPostProcessors集合中
7. initMessageSource
   - 创建MessageSource
   - 实现国际化功能
8. initApplicationEventMulticaster
   - 创建事件广播器
9. onRefresh
   - 空实现，留给子类实现
   - SpringBoot中的子类可以在这里准备内嵌的web容器
10. registerListeners
    - 创建事件监听器
11. finishBeanFactoryInitialization
    - 初始化BeanFactory中的成员变量
    - 解析${}
    - 创建beanDefinitionMap中保存的对象
12. finishRefresh
    - 创建生命周期处理器

## BeanFactory

**实现类**

- DefaultListableBeanFactory

  - 提供控制反转、基本的依赖注入、Bean生命周期的各个功能

  - 默认无其他后处理器，解析不了注解，需要添加

    ```java
    //添加后处理器
    AnnotationConfigUtils.registerAnnotationConfigProcessors(容器对象);
    //启动后处理器
    容器对象.getBeanOfType(BeanFactoryPostProcessor.class)
        ,values().stream().forEach(beanFactoryPostProcessor -> {
        beanFactoryPostProcessor.postProcessBeanFactory(容器对象);
    })
    ```

  - 容器中的Bean对象在使用时才会创建（延迟创建），也可以提前创建

    ```java
    //提前创建
    容器对象.preInstantiateSingletos()
    ```

  - 默认不会解析 ${}、#{}

## ApplicationContext

**实现类**

- ClassPathXmlApplicationContext、FIleSystemXmlApplicationContext
  - 创建DefaultListableBeanFactory
  - 通过XmlBeanDefinitionReader中的loadBeanDefinitions()

  - 读取Xml中的Bean
- AnnotationConfigApplicationContext
  - 通过配置类创建容器

  - 会自动添加常用的后处理器

- AnnotationConfigServletWebServerApplicationContext
  - 借助了内嵌的Tomcat
  - 需要提供一些Bean
    - ServletWebServerFactory
    - DispatcherServlet
    - DispatchServletRegistrationBean
  - 非必须Bean
    - Controller（web.servlet.mvc）
      - 处理Request、Response
      - 可以指定访问的url

## 生命周期

1. 处理名称，检查缓存
  - 解析别名，解析特殊符号，查询三级缓存看是否有创建好的Bean

2. 处理父子容器
  - 如果有父容器，回去父容器是否有创建好的Bean
    - 父子容器的bean名称可以重复
    - 优先找子容器Bean，子容器没有再找父容器

3. dependsOn
  - 判断是否用使用dependsOn指定Bean的创建顺序

4. 按不同的Scope创建Bean
5. 实例化
  - 通过构造函数创建
  - 反射生成对象，堆中申请空间，给成员变量赋默认值
6. 依赖注入
7. 初始化
  - Bean属性注入
    - 包括自定义属性和容器属性
  - Bean功能扩展
    - BeanPostProcessor
      - AOP在此处实现
8. 使用，通过容器对象调用getBean()方法获取Bean对象
9. 销毁

## 三级缓存 / 循环依赖

- 用于解决循环依赖的问题
  - 循环依赖指的是对象创建时需要另一个对象，正好那个对象也在创建并等待该对象创建导致卡住
    - 主要在设值注入、构造注入时出现
    - 设值注入单例情况下使用三级缓存解决
    - 原型对象不使用三级缓存，因为会大量存储于缓存中无法被gc
    - 构造注入出现循环依赖是无解的，所以只能通过检测来避免
      - 用一个容器存储，当对象创建时放入该容器，每当在需要依赖的时候就检测这个容器中是否有对应的对象，有的话就是发生了循环依赖
  
- 一级缓存：singletonObjects
  - 存成品对象
  - createBeanInstances之后放置
  - Bean生命周期中后置处理器处理时存入
- 二级缓存：earlySingletonObjects
  - 存半成品对象
  - 第一次从三级缓存中确定对象是代理对象还是普通对象的时候放置，同时删除三级缓存
  - Bean生命周期中填充属性时存入
- 三级缓存：singletonFactories
  - 存的是工厂对象，也是lambda表达式，value值是ObjectFactory，key是beanname
  - 生成完整对象之后放到一级缓存，删除二三级缓存
  - Bean生命周期中调用构造方法时存入

## 事务原理

​		事务是由AOP来实现的，首先生成代理对象，通过TransactionInterceptor，调用invoke实现具体逻辑

1. 解析方法上事务的相关属性，判断是否开启新事务
2. 获取数据库连接，关闭自动提交，开启事务
3. 执行具体的sql逻辑
   - 成功：通过commitTransactionAfterReturning提交，通过doCommit实现
   - 失败：通过commitTransactionAfterThrowing执行回滚，通过doRollBack实现
4. 之后清除相关的事务信息，cleanupTransactionInfo

## 后处理器

### BeanPostProcessor

- AutowiredAnnotationBeanPostProcessor
  - 解析@Autowired、@Value
  - 内部调用的postProcessprProperties方法解析注解
    1. 通过findAutowiringMetadata找到加了@Autowired的成员信息metadata
    2. 调用metadata.inject方法进行依赖注入
       - 获取成员信息的类型
       - 封装为DependencyDescriptor对象
       - 然后通过BeanFactory.doResolveDependency到容器里找对应的Bean
- ContextAnnotationAutowiredCandidateResolver
  - 解析${}
- CommonAnnotationBeanPostProcessor
  - 解析@Resource、@postConstruct、@PreDestroy
- ConfigurationPropertiesBindingPostProcessor
  - 解析@ConfigurationProperties

### BeanProcessor

- ConfigurationClassPostProcessor
  - 解析@Bean、@ComponentScan、@Import、@ImportResource
- MapperScannerConfigurer
  - 讲mapper加入容器，解析@MapperScan

## 注解

### @Autowired

### @ComponentScan

1. 获取包路径
2. 转化为classpath的格式
3. 通过容器的getResources方法获得路径下所有的类
4. 判断类是否直接或间接加了@Component
   - 通过CachingMetadataReaderFactory对象的getMetadataReader方法获得MetadataReader
   - MetadataReader通过getAnnotationMetadata方法获得AnnotationMetadata对象
   - AnnotationMetadata对象调用hasAnnotation和hasMetaAnnotation判断是否直接或间接标注了@Component注解
5. 加了注解就把这个bean封装到BeanDefinition
   - 使用BeanDefinitionBuilder通过Bean名称获取BeanDefinition
   - 通过AnnotationBeanNameGenerator生成Bean的名称
   - 加入容器

### @Bean

1. 获取包路径
2. 转化为classpath的格式
3. 通过容器的getResources方法获得路径下所有的类
4. 判断类是否直接或间接加了@Component
   - 通过CachingMetadataReaderFactory对象的getMetadataReader方法获得MetadataReader

# SpringMVC

## 自动配置

- 主要是通过@EnableAutoConfiguration及其内部注解实现
  - @AutoConfigurationPackage
    - @Import(AutoConfigurationPackage.Registrar.class)
      - 利用Registrar批量注册组件
  - @Import(AutoConfigurationImportSelector.class)
    - 利用getAutoConfigurationEntry(annotationMetadata a) 给容器批量导入一些组件
    - 调用getCandidateConfigurations(annotationMetadata, attributes) 获取所有需要导入的容器中的配置类/组件
      - 利用了SpringFactoriesLoader（Spring工厂加载器）的loadFactoryNames() 这个方法返回loadSpringFactories的Map得到所有的组件
        - loadSpringFactories方法会扫描当前系统里面所有的META-INF/spring.factories

## 执行流程

1. 用户发起HttpRequest请求，请求进入到 DispatcherServlet
   - 默认路径是 “ / ”，会匹配到所有的url
   - SpringBoot会自动创建并加载DispatcherServlet的Bean
     - 会创建Spring容器并执行refresh方法
   - 第一次请求来时会初始化DispatcherServlet，然后到容器中找它依赖的组件，没有就用默认的
     - HandlerMapping、HandlerAdapter、HandlerExceptionResolver、ViewResolver 
2. DispatcherServlet 将请求转发给所有的 HandlerMapping，HandlerMapping 会找到能处理这些请求的Handler方法
   - 这些Handler方法会被封装成HandlerMethod对象，并结合匹配到的拦截器，合并成HandlerExecutionChain（调用链）对象，然后一起返回给DispatcherServlet 
   - HandlerMapping 会在初始化时就会建立好请求路径和处理器的映射关系
3. DispatcherServlet 收到这条调用链后 
   - 调用所有拦截器的preHandle方法，返回true才继续后续的调用
   - 调用HandlerAdapter解析HandlerMethod并调用对应的Handler方法，准备数据绑定工厂、模型工厂，将HandlerMethod完善为ServletInvocableHandlerMethod
     - 使用HandlerMethodArgumentResolver参数解析
     - 调用ServletInvocableHandlerMethod
     - 调用HandlerMethodReturnValueHandler处理返回值
       - 有@ResponseBody注解的话直接返回Json
       - 否则返回的是 ModelAndView ，然后进行视图解析和渲染
   - 调用拦截器的postHandle方法
4. 视图渲染或处理异常
   - 如果1 - 3出现异常，ExceptionHandlerExceptionResolver处理
     - @ControllerAdvice增强5：@ExceptionHandler异常处理
   - 视图解析即渲染
     -  将 ModelAndView 转发给 ViewResolver 处理并返回 View 对象给 DispatcherServlet
     -  DispatcherServlet 再将渲染后的页面返回给客户端
5. 调用拦截器的afterCompletion方法

# SpringBoot

## 执行流程

1. 调用构造方法创建SpringApplication对象
   - 根据各种来源添加BeanDefinition
   - 推断应用程序类型(servlet, none, reacter)
   - 准备初始化器，监听器
   - 进行主类推断
2. 执行run方法
   1. 得到SpringApplicationRunListeners事件发布器
      - 发布7个事件
        - starting：开始启动
        - environmentPrepared：准备环境
        - contextPrepared：容器创建并调用初始化器
        - contextLoaded：Bean定义加载
        - started：refresh方法执行
        - running：springboot启动完成
   2. 封装args为ApplicationArgument对象
      - 用DefaultApplicationArguments(args)封装
   3. 创建Environment对象
      - 有系统属性和系统环境变量，优先找系统属性
   4. 对Environment中命名规范统一处理
   5. 对Environment做扩展增强
      - 使用EnvironmentPostProcessor后处理器做增强
   6. 对Environment中以"spring.main"为前缀的key与SpringApplication容器对象做绑定
   7. 打印Banner
   8. 创建spring容器
   9. 应用初始化器的增强
   10. 得到所有的BeanDefinition
   11. 调用Application的refresh方法
   12. 调用所有实现ApplicationRunner和CommandLineRunner接口的Bean

## 自动装配

1. 通过@EnableAutoConfiguration注解

# SpringCloud

## Nacos

### 动态配置刷新

- 主要依赖推送和拉取
  - 推送机制是指Nacos服务器在检测到配置变化时，主动通知客户端更新配置。Nacos服务器使用了**长轮询**的方式来实现推送
  - 拉取机制是客户端每30秒会执行一次心跳检测，此时客户端也会执行一次配置拉取操作，如果发现配置有变化，则会触发配置更新的回调函数，可以保证nacos推送失败或者网络问题异常的情况下依然能获取最新的配置
    - 回调函数是ContextRefresher类的refresh方法，该方法会触发一个RefreshEvent事件，该事件会被springcloud的组件监听并执行相应的刷新逻辑
- 原理是基于Environment、@RefreshScope、@Value
  - 通过Environment获取的配置会自动刷新
  - 通过@RefreshScope标注的Bean会重新创建并注入容器
  - 通过@Value的字段会重新赋值

### 服务注册

1. 调用register接口，传入ip、namespace、groupName、serviceName等等
2. 解析request请求取得参数

   - 如果没传namespace会使用默认的namespace

   - 服务名称：groupName@@serviceName

     - 解析request请求获得参数，封装为Instance对象

     - 内部有两个集合，一个放临时实例，一个放永久实例
3. 将参数封装为Instance对象，然后调用registerInstance注册该实例

   1. 创建空的服务service，只有第一次会创建，为了确保有服务对象
      - 尝试从注册表中获取service
        - 为空则创建，会记录服务最后一次更新时间
        - 存进注册表，双重检查
   2. 添加实例到服务service中
      - 为服务生成唯一标识
      - 从注册表中拿到服务service
      - 加锁，多实例间只能串行添加
        - 拷贝注册表中旧的实例列表然后加上新注册的实例组成完整的实例列表