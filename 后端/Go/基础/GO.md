---
title: Go
categories: Go
tags: [language, go]
---

## 简介

- 速度快，易学，支持高并发
- 适合网络编程、区块链开发
- 类c语言
- 成员属性和方法如果是大写开头就是public的，小写开头的就是private的

## 命令

| 命令  | 功能                                         | 备注 |
| ----- | -------------------------------------------- | ---- |
| build | 编译包和依赖                                 |      |
| run   | 编译并运行                                   |      |
| clean | 移除当前源码包和关联源码包里面编译生成的文件 |      |
| env   | 打印Go的环境信息                             |      |
| fix   | 将旧的API替换成新的API                       |      |
| mod   | 模块维护                                     |      |
| get   | 将依赖项添加到当前模块并安装它们             |      |

## 基本数据类型

| 类型       | 默认值   | 字节 | 格式符 | 备注                      |
| ---------- | -------- | ---- | ------ | ------------------------- |
| byte       |          | 1    | %c     | 单个字符                  |
| string     |          |      | %s     | \0结束，中文3字符，不可变 |
| bool       | false    |      | %t     |                           |
| int        | 0        | 8    | %d     | 有符号                    |
| uint       |          | 8    |        | 无符号                    |
| float32    |          |      | %f     | 小数点后7                 |
| float64    |          |      | %f     | 小数点后15，常用          |
| double     |          |      |        |                           |
| complex64  |          |      | %v     | 复数，64 位实数+64 位虚数 |
| complex128 |          |      | %v     | 64 位实数+64 位虚数       |
| 地址值     | 十六进制 |      | %p     |                           |
| 类型       |          |      | %T     |                           |

## 复合数据类型

**切片**

- 截取集合中的一部分数据

- 结构

  - 一个指针，指向数组中 slice 指定的开始位置
  - 长度，即 slice 的长度
  - 最大长度，也就是 slice 开始位置到数组的最后位置的长度

- 内置函数

  - len：获取长度
  - cap：获取最大容量
  - append：
    - 追加一个或者多个元素，然后返回一个和 slice 一样类型的slice
    - 追加元素会影响到原数组和其他指向该数组的切片
    - 当追加元素超过切片最大长度时会创建一个新的数组空间，不会影响到其他

  - copy：copy 从源 slice 的 src 中复制元素到目标 dst，并且返回复制的元素的个数

**数组**

- 函数传递数组是深拷贝，函数传切边是浅拷贝
- 切边截取是浅拷贝
- 切边扩容x2，大于1024字节后x 1/4

```go
//定义
var 数组名 [元素数量]类型

//自动推导,数量可以省略用...替代
数组名 := [数量]类型{对应的值}

//嵌套数组
数组名 := [2][4]int{
    [4]int{1, 2, 3, 4}, 
    [4]int{5, 6, 7, 8}
}
//可省略，需要数组类型一致
数组名 := [2][4]int{
    {1, 2, 3, 4}, 
    {5, 6, 7, 8}
}

//初始化
var 数组名[元素数量] 类型 = [数量]类型{对应的值}
var 数组名[元素数量] 类型 = [数量]类型{下标:对应的值}

//切片创建
//1
var 切片名 []类型
//2，自动推导
切片名 := []类型{}
//3,长度是初始化长度，容量是最大长度
make([]类型,长度,容量)
//4，获取数组切片
切片名 = 数组名[开始下标:结束下标]
```

**map**

- 无序，kv存储
- k不重复
- 函数中是浅拷贝

```go
//创建 & 初始化 
var map名 map(key类型)value类型 = map[key类型]value类型{k:v}
map名 := map(key类型)value类型{k:v}
map名 := make(map(key类型)value类型)
map名[k] = v

//判断value是否存在, 如果值存在
a, b = map名[k]
```

**结构体**

- 函数传递结构体是深拷贝，函数传递切片是浅拷贝

```go
//创建
type 结构体名 struct{
    成员变量 类型 `标签:值`
    //如果需要序列化
    成员变量 类型 `json:成员变量名`
}

//切片
var 切片名[] 类型

//转json
import "encoding/json"

//结构体转json
hsonStr, err := json.Marshal(结构体)
//json转结构体
err = json.Unmarshal(json, 结构体对象)
```

**指针**

- 数组指针取值时应该 `(*p)[下标]`， 因为括号的优先级高
  - Go优化成了：`p[下标]` 
- 指针数组取值时应该 `*p[下标]` 
- 多级指针

```go
//创建普通指针
var 指针名 *类型 =  &对象
//数组指针
var 指针名 *[数量]类型 = &数组
//指针数组
var 指针名 [数量]*类型

//切片
var 指针名 *[]类型
```

**channel**

- 解决协程之间的同步问题
- 没有设置容量的channel为无缓冲channel
  - 需要同时读写，会阻塞，适合同步任务

- 有缓冲channel适合异步任务
- channel关闭后不能写数据，但是可以读数据
- 创建时默认是双向channel，可以设置单向channel
  - 双向channel可以转换为任意单向channel，反之不行
- 有一个定时器的类


```go
//创建双向channel
var ch chan channel中数据的类型
ch := make(chan channel中数据的类型，容量)
//单向写channel
var ch chan <- channel中数据的类型
ch := make(chan <- channel中数据的类型，容量)
//单向读channel
var ch <- chan channel中数据的类型
ch := make(<- chan channel中数据的类型，容量)

//写入
ch <-
//读取
s <- ch

//关闭
close(ch)
```

## 常量 / 变量

- 常量格式

  - const 名称  【类型】= 值

- 变量格式

  - var 名称 类型 【 = 值】
  - var 名称1，名称2，名称3  类型 【 = 值1，值2，值3】
  - 名称 := 值 （简短声明 / 自动推导）（只能用在函数中）

- _（下划线）是个特殊的内置变量，任何赋予它的值都会被丢弃

- 以声明未使用的值会在编译阶段报错

- 字符串不可变，怎么修改

  ```go
  //1转为字节数组 []byte(c)
  s := "hello"
  c := []byte(s) 
  c[0] = 'c'
  s2 := string(c) // 再转换回 string 类型
  fmt.Printf("%s\n", s2)
  
  //2用切片拼接
  s := "hello"
  s = "c" + s[1:] // 字符串虽不能更改，但可进行切片操作
  fmt.Printf("%s\n", s)
  ```

- 如果想输出多行的字符串，需要使用 ` 括起来

  ```go
  str := `abc
  		def`
  ```

- 大写字母开头的变量/函数，是可导出的，其它包可以读取，小写字母开头的则不行

## 函数 / 方法

```go
//定义
func method(参数 类型)(返回值1 返回值类型, ...){
    
}
//使用，多个返回值
a, b = method();
```

- 如果方法是公共的，返回值最好命名
- 函数也是一种变量，可以通过type来定义它，可以用作参数进行传递

## 关键字

- **defer**
  - 相当于java中的final，c中的析构，即函数结束前执行的操作
  - 执行顺序是后进先出，栈式执行，逆序执行，先执行最下面的
  - return比defer先执行
- select
  - 监听channel上的数据流动
  - 语法类似switch
  - 多个满足条件的随机执行一个
  - 如果一直走到default会导致忙轮询，不建议使用default
- iota
  - 自动枚举，默认值为0，调用一次加1
  - 每遇到一个 const 关键字，iota 就会重置

- fallthrough
  - 在switch中使用，强制执行满足条件之后的所有条件，包括default

## 面向对象

- 封装
  - 定义结构体时的this最好指向对象的地址，这样是浅拷贝
- 继承
  
  ```go
  type 子类 struct{
      父类
  }
  ```
  
- 多态

  - 父类指针指向子类对象

## 接口

```go
//接口定义
type 接口名 interface{
    方法名() 【返回值类型】
}

//接口实现
type 实现 struct{
    成员变量
}
func (this &实现) 接口方法名 【返回值类型】{
	实现方法
}

//万能接口，可接收任意类型
func method(arg interface{}){
    
}
//万能接口提供了类型断言
value, ok := arg.(string) //判断arg是否是字符串
```

## 反射

```go
//获取对象的type
reflect.TypeOf(对象)
//获取对象的value
reflect.ValueOf(对象)
//获取对象中的成员的标签
t := reflect.TypeOf(对象).Elem()
tagIt.Field(下标).Tag.Get("标签名")
```

## 异常

- 实际发生异常时系统内部是通过panic函数去抛出这个异常，尽量少用
- 可以使用errors中的方法对异常进行处理
- 延迟函数defer会正常执行

## 文本处理

```go
//创建文件
file, err := os.Create("路径")

str = "内容"
//写入文件1
length, err := file.WriteString(str)
//写入文件2，接收字节数组
length, err := file.Write([]byte(str))
//写入文件3，写入指定位置，会覆盖数据，
length, err := file.WriteAt([]byte(str), 下标)

//定位文件内容下标
length, err := file.Seek(追加空格数，io.SeekEnd)


//读取文件1
file, err := os.OpenFile("路径"， 模式， 权限[0-7])
//读取文件2，只读，无权限
file, err := os.Open("路径")

//读取文件内容，buffer为读取缓冲区，空余区域会用0填充
//buffer存储的A
buffer := make([]byte, 1024 * 2)
length, err := file。Read(buffer)

//关闭文件
defer file.close
```

## 协程 gorountine

- 提高CPU的利用率

- 主要是gorountine和channel

- 内存占用几kb

- 早期的调度器需要获取锁才能执行go协程，性能比较差

- 调度器设计策略
  - 复用线程
  - 利用并行
  - 抢占
  - 全局G队列


## GMP



## Go调用以太坊

- 启动rpc端口

  - `geth --identity "XZ" --http --http.port 8545 --datadir "C:\Go\test2\testBlock" --http.api "net,eth,personal,db,web3" --http.corsdomain "*" console`
    - identity：节点身份标识
    - http：开启rpc
    - http.port ：rpc端口
    - http.api：对外提供的api
    - http.corsdomain：允许连接的url，*无限制，默认只有本机能访问
    - datadir：区块数据文件夹
    - networkid：net_version的id，即网络类型
      - 1是主网络，其余是测试网络
    - port：监听其他端口
    - nodiscover：隐藏节点，需手动添加

- 在区块数据文件夹下创建Go工程

- 导入以太坊相关api

  ```go
  //获取连接Client
  var client Client
  client = rpc.Dial("ip：端口")
  
  
  //获取网络类型networkid
  var networkid string
  client.Call(&networkid, "net_version")
  //获取是否监听
  var is_listing bool
  client.Call(&is_listing, "net_listening")
  //获取节点数量
  var count string
  client.Call(&count, "net_peerCount")
  
  //获取账户
  var account []string
  client.Call(&account, "eth_accounts")
  //获取账户余额
  var balance string
  client.Call(&balance, "eth_getBalance", 账户地址，区块编号(latest、))
  //获取当前gas价格
  var gas string
  client.Call(&gas, "eth_gasPrice")
  //获取挖矿地址账户
  var coinbase string
  client.Call(&coinbase, "eth_coinbase")
  //获取当前以太坊协议版本
  var protocolVersion string
  client.Call(&protocolVersion, "eth_protocolVersion")
  //获取当前是否在挖矿
  var isMining bool
  client.Call(&isMining, "eth_mining")
  //获取挖矿速率
  var hashrate string
  client.Call(&hashrate, "eth_hashrate")
  //获取指定地址发生的交易数量
  var transactionCount string
  client.Call(&transactionCount, "eth_getTransactionCount", "地址"，区块编号(latest、))
  //获取节点当前块编号
  var blockNum string
  client.Call(&blockNum, "eth_blockNumber")
  
  //创建账户
  var account string 
  var pwd string // 参数
  client.Call(&account, "personal_newAccount", pwd)
  //获取账户列表
  var accounts []string 
  client.Call(&accounts, "personal_listAccounts")
  //锁定指定账户
  var lock bool
  client.Call(&lock, "personal_lockAccount", "地址")
  //解锁指定账户
  var lock bool
  client.Call(&lock, "personal_unlockAccount", "地址", "密码")
  
  //写入数据库
  
  ```

## Go调用智能合约

1. 通过remix部署合约获取abi信息，新建一个abi后缀名的文件
   - 或者通过工具solc生成
     - `solc -bin test.sol -o test.abi`
2. 使用abigen工具，根据abi文件生成对应的go文件
   - `abigen --abi abi文件 --pkg 包名 --type 结构体名 --out 文件名.go`
