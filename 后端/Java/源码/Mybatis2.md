## 架构

![image-20230317103336946](image-20230317103336946.png)


## 组件及其调用关系

![image-20230317115152145](image-20230317115152145.png)

## 目录结构

![image-20230317115410587](image-20230317115410587.png)

## 解析配置文件

### 读取xml文件流

```java

public static InputStream getResourceAsStream(ClassLoader loader, String resource) throws IOException {
    InputStream in = classLoaderWrapper.getResourceAsStream(resource, loader);
    if (in == null) {
        throw new IOException("Could not find resource " + resource);
    } else {
        return in;
    }
}
```

```java
public InputStream getResourceAsStream(String resource, ClassLoader classLoader) {
    return this.getResourceAsStream(resource, this.getClassLoaders(classLoader));
}
```

```java
ClassLoader[] getClassLoaders(ClassLoader classLoader) {
    return new ClassLoader[]{
        classLoader, 
        this.defaultClassLoader, 
        Thread.currentThread().getContextClassLoader(), 
        this.getClass().getClassLoader(), 
        this.systemClassLoader};
}
InputStream getResourceAsStream(String resource, ClassLoader[] classLoader) {
    ClassLoader[] var3 = classLoader;
    int var4 = classLoader.length;
    for(int var5 = 0; var5 < var4; ++var5) {
        ClassLoader cl = var3[var5];
        if (null != cl) {
            InputStream returnValue = cl.getResourceAsStream(resource);
            //重试，尝试+"/"能不能获取到
            if (null == returnValue) returnValue = cl.getResourceAsStream("/" + resource);
            if (null != returnValue) return returnValue;
        }
    }
    return null;
}
```

### 构建SqlSessionFactory

```java
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    SqlSessionFactory var5;
    try {
        //
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        //
        //
        var5 = this.build(parser.parse());
    } catch (Exception var14) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", var14);
    } finally {
        ErrorContext.instance().reset();
        try {
            inputStream.close();
        } catch (IOException var13) {
        }

    }
    return var5;
}
```

```java
public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
    //
    this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
}
private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    //
    super(new Configuration());
    this.localReflectorFactory = new DefaultReflectorFactory();
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
}
```

```java
public Configuration parse() {
    //默认为false
    if (this.parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    } else {
        //这个解析方法只能调用一次，即配置文件只能解析一次
        this.parsed = true;
        
        this.parseConfiguration(this.parser.evalNode("/configuration"));
        return this.configuration;
    }
}
/*
    parser.evalNode 解析configuration节点及其子节点，封装到一个XNode对象中
    parseConfiguration 解析XNode对象（properties、settings、environments、mapper等等）
    1.将解析出的内容设置到configuration中
    2.解析mapper
*/
private void parseConfiguration(XNode root) {
    try {
        this.propertiesElement(root.evalNode("properties"));
        Properties settings = this.settingsAsProperties(root.evalNode("settings"));
        this.loadCustomVfs(settings);
        this.loadCustomLogImpl(settings);
        this.typeAliasesElement(root.evalNode("typeAliases"));
        this.pluginElement(root.evalNode("plugins"));
        this.objectFactoryElement(root.evalNode("objectFactory"));
        this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
        this.settingsElement(settings);
        this.environmentsElement(root.evalNode("environments"));
        this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        this.typeHandlerElement(root.evalNode("typeHandlers"));
        //解析mapper
        this.mapperElement(root.evalNode("mappers"));
    } catch (Exception var3) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
    }
}
```

```java
private void mapperElement(XNode parent) throws Exception {
    //获取<mappers>的子节点，解析属性
    //获取子节点中的 url/resources ，转化为字节流，交给 XMLMapperBuilder 解析
    //调用XMLMapperBuilder.parse 方法
}
```

```java
//根据传入的字节流，构建Document对象
private XMLMapperBuilder(XPathParser parser, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
    super(configuration);
    this.builderAssistant = new MapperBuilderAssistant(configuration, resource);
    this.parser = parser;
    this.sqlFragments = sqlFragments;
    this.resource = resource;
}
```

```java
public void parse() {
    if (!this.configuration.isResourceLoaded(this.resource)) {
        //获取mapper的命名空间，解析（cache、sql、resultmap、crud）
        this.configurationElement(this.parser.evalNode("/mapper"));
        this.configuration.addLoadedResource(this.resource);
        this.bindMapperForNamespace();
    }
    this.parsePendingResultMaps();
    this.parsePendingCacheRefs();
    this.parsePendingStatements();
}
```

```java
//解析crud标签
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    Iterator var3 = list.iterator();
    while(var3.hasNext()) {
        XNode context = (XNode)var3.next();
        XMLStatementBuilder statementParser = 
            new XMLStatementBuilder(this.configuration, this.builderAssistant, context, requiredDatabaseId);
        try {
            //解析，封装
            statementParser.parseStatementNode();
        } catch (IncompleteElementException var7) {
            this.configuration.addIncompleteStatement(statementParser);
        }
    }
}
```

```java
public void parseStatementNode() {
    //获取标签中的id
    //判断标签是crud中的哪一个，设置标签类型
    //解析属性
    //解析sql，替换占位符，保存占位符中的值（重点）
    SqlSource sqlSource = 
        langDriver.createSqlSource(this.configuration, this.context, parameterTypeClass);
    //解析属性
    //通过构建者助手，创建MappedStatement对象，存入configuration中的MapperStatementMap中
    this.builderAssistant.addMappedStatement(
        			id, sqlSource, statementType, 
                     sqlCommandType, fetchSize, timeout, 
                     parameterMap, parameterTypeClass, resultMap,
                     resultTypeClass, resultSetTypeEnum, flushCache, 
                     useCache, resultOrdered, (KeyGenerator)keyGenerator,
                     keyProperty, keyColumn, databaseId, 
                     langDriver, resultSets);
}
```

```java
public SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) {
    //初始化动态sql处理器，初始化节点处理器集合（when、where、if等等）
    XMLScriptBuilder builder = new XMLScriptBuilder(configuration, script, parameterType);
    /*
    	解析动态sql
    	将带有${}的sql封装到TextSqlNode
    	将带有#{}的sql封装到StaticTextSqlNode
    	将动态SQL标签中的SQL信息分别封装到不同的SqlNode中，对应不同的节点处理器去解析
    */
    return builder.parseScriptNode();
}
```

## 创建SqlSession对象

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
```

```java
public SqlSession openSession() {
    //参数：执行器类型、事务隔离级别、是否自动提交事务
    return 
        this.openSessionFromDataSource(
        	//默认是简单执行器
        	this.configuration.getDefaultExecutorType(),(TransactionIsolationLevel)null, false
    	);
}
private SqlSession openSessionFromDataSource(ExecutorType execType, 
                                             TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    DefaultSqlSession var8;
    try {
        //获取Environment对象
        Environment environment = this.configuration.getEnvironment();
        //获得事务工厂对象
        TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
        //创建事务对象
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        //创建执行器对象，默认是SimpleExecutor，如果开启缓存的话会是CacheingExecutor（装饰器模式）
        Executor executor = this.configuration.newExecutor(tx, execType);
        //创建DefaultSqlSession对象
        var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
    } catch (Exception var12) {
        this.closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
    } finally {
        ErrorContext.instance().reset();
    }
    return var8;
}
```

## SqlSession执行流程

1. 调用SqlSession对应的Api，传入statementId（namespace.id）和参数
2. 获取BoundSql，里面的sql已经经过处理和解析了
3. 如果有开启缓存则在查询时生成缓存key（默认是开启的）
   1. 对应的执行器去创建缓存key
   2. 根据缓存key去二级缓存中查数据，没有再去一级缓存中找，一级缓存没有再去查数据库，存到一个缓存中
      1. 查询数据库前会现在一级缓存中进行一个占位操作
      2. 然后查询数据库
         1. 获取configuration对象
         2. 调用configuration去构建StatementHandler相关处理器
            - 使用的是RoutingStatementHandler去选择，是一个装饰器，并且执行插件
            - 由StatementType决定（可配置），默认是PreparedStatementHandler
         3. 准备处理器，包括创建statement和动态参数的设置
            - 获取一个连接，通过事务对象获取
            - 创建Statement对象
            - 通过parameterHandler参数处理器去设置参数
              - 从boundSql获取每个占位符（#{}）中的值的集合
              - 遍历，判断是入参还是出差，出差不处理
              - 判断是否存在能处理该参数的类型处理器（TypeHandler）（入参类型）
              - 使用TypeHandler为对应占位符赋值
            - 返回statement对象
         4. 查询数据库
            - 将statement对象转换为PreparedStatement对象
            - 执行sql
            - 使用ResultSetHandler处理结果集
              1. 获取ResultSet对象
                 - 通过Statement获取ResultSetWrapper
              2. 获取映射关系
                 - 通过Statement获取ResultMap
                 - 遍历ResultMap获取ResultSet
              3. 根据映射关系封装实体
                 - 构建默认的结果集处理器（DefaultResultHandler）
                 - 对结果集进行映射封装，将封装后的对象存入DefaultResultHandler
                   - 判断是否有内置嵌套结果映射，并进行相应的处理
      3. 将数据填充到占位的缓存中

## SqlSession原理

- 门面模式

## 执行器原理

![image-20230319215902238](../../pic/image-20230319215902238.png)

### SimpleExecutor

- 简单执行器，每次执行sql都会进行预编译sql

### ReuseExecutor

- 重用执行器

## 分页原理

- 分页参数放到ThreadLocal中，拦截执行的sql，根据数据库类型重写sql，计算分页的各种数据

## 总结

- 源码中大量使用了工厂模式
  - SqlSessionFactory、DatasourceFactory、TransactionFactory
- 但是源码中很多地方都是使用的硬编码来和xml中的节点做匹配
- 
