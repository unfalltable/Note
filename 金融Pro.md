## 项目结构

```yaml
- srt
	- common	#统一的处理，工具
	- base		#配置信息
	- core		#业务逻辑
```

## Mybatis-plus

### 代码生成器

```java
public void genCode() {

    // 1、创建代码生成器
    AutoGenerator mpg = new AutoGenerator();

    // 2、全局配置
    GlobalConfig gc = new GlobalConfig();
    String projectPath = System.getProperty("user.dir");
    gc.setOutputDir(projectPath + "/src/main/java");
    gc.setAuthor("jianhong.xu");
    gc.setOpen(false); //生成后是否打开资源管理器
    gc.setServiceName("%sService");	//去掉Service接口的首字母I
    gc.setIdType(IdType.AUTO); //主键策略
    gc.setSwagger2(true);//开启Swagger2模式
    mpg.setGlobalConfig(gc);

    // 3、数据源配置
    DataSourceConfig dsc = new DataSourceConfig();
    dsc.setUrl("jdbc:mysql://localhost:3306/loan?serverTimezone=GMT%2B8&characterEncoding=utf-8");
    dsc.setDriverName("com.mysql.cj.jdbc.Driver");
    dsc.setUsername("root");
    dsc.setPassword("123456");
    dsc.setDbType(DbType.MYSQL);
    mpg.setDataSource(dsc);

    // 4、包配置
    PackageConfig pc = new PackageConfig();
    pc.setParent("xu.srb.core");
    pc.setEntity("pojo.entity"); //此对象与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
    mpg.setPackageInfo(pc);

    // 5、策略配置
    StrategyConfig strategy = new StrategyConfig();
    strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略
    strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略
    strategy.setEntityLombokModel(true); // lombok
    strategy.setLogicDeleteFieldName("is_deleted");//逻辑删除字段名
    strategy.setEntityBooleanColumnRemoveIsPrefix(true);//去掉布尔值的is_前缀（确保tinyint(1)）
    strategy.setRestControllerStyle(true); //restful api风格控制器 返回json
    mpg.setStrategy(strategy);

    // 6、执行
    mpg.execute();
}
```

## 统一返回结果

- 创建一个枚举类

  - ```java
    @Getter
    public enum ResponseEnum{
        //返回的状态码
        private Integer code;
        //返回的信息
        private String message;
        
        //定义各种枚举
        SUCCESS(0, "成功"),
        ERROR(-1, "失败"),
        ...;
        
    }
    ```

- 创建一个类作为统一的返回结果

  - ```java
    @Data
    class R {
        //返回的状态码
        private Integer code;
        //返回的信息
        private String message;
        //返回的数据
        private Map<String, Object> data = new Map<>();
        //构造函数私有化
        private R(){}
        //返回成功
        public static R ok(){
            R r = new R();
            r.setCode(ResponseEnum.SUCCESS.getCode());
            r.setMessage(ResponseEnum.SUCCESS.getMessage());
            return r;
        }
        //返回失败
        public static R error(){
            R r = new R();
            r.setCode(ResponseEnum.ERROR.getCode());
            r.setMessage(ResponseEnum.ERROR.getMessage());
            return r;
        }
        //返回特定的结果
        public static R setResult(ResponseEnum responseEnum){
            R r = new R();
            r.setCode(responseEnum.getCode());
            r.setMessage(responseEnum.getMessage());
            return r;
        }
        //给Data赋值
        public R data(String key, Object value){
            this.data.put(key, value);
            return this;
        }
        public R data(Map<String, Object> map){
            this.setData(map);
            return this;
        }
        //设置特定的消息
        public R message(String message){
            this.setMessage(message);
            return this;
        }
        //设置特定的code
        public R code(Integer code){
            this.setCode(code);
            return this;
        }
    }
    ```

## 统一异常处理

### 创建统一异常处理类

- ```java
  @RestControllerAdvice
  public class UnifiedExceptionHandler{
      //处理非指定的异常
  	@ExceptionHandler(value = Exception.class)
      public R handlerException(Exception e){
          //打印日志
          return R.error();
      }
      //指定异常处理，可以是自定义异常
      @ExceptionHandler(value = 指定的异常.class)
      public R handlerException(指定的异常 e){
          //打印日志
          return R.error();
      }
      //定义controller上层异常
      @ExceptionHandler({
          //上层异常，这些异常预先设置了枚举
      })
  }
  ```

### 自定义异常

- 需要继承RuntimeException
  - 定义错误码和错误消息，还有各种参数的构造函数，包含异常信息，在抛异常的时候传入

### 优化

- 使用断言优化
  - 创建一个断言工具类用来替代一些判断条件

## 统一日志处理（LogBack）

## Swagger

- 生成接口文档
