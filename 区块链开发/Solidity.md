---
title: Solidity
categories: Web3
tags: [solidity]
---

## 简介

- 用于智能合约开发，是面向对象的，强类型语言
- 单线程的，线程安全
- 使用时需要先声明Solidity的版本号
- 使用remix开发
  - https://remix.ethereum.org

## 格式

```solidity
//声明使用的solidity版本
pragma solidity ^0.8.7;
//合约
contract A {
	
	//状态变量
	string public _string = "Hello Web3!";
	
	//方法
    function method1 public {
        
    }
}
```

## 关键字

| 关键字    | 作用                                                         | 说明                                                         |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| constant  | 定义常量                                                     | gas费较低                                                    |
| immutable | 将变量定义为常量                                             | gas费较低，需要赋值                                          |
| indexed   | 事件中用于修饰变量，他们会保存在以太坊虚拟机日志的 topics 中 | 相当于检索事件的索引“键”，方便搜索，大小为固定的256比特，超过这个大小会自动计算成哈希存储 |
| ether     | 以太坊代币单位                                               | 1 ether = 10^18 wei                                          |
| memory    | 将变量存储在内存                                             | 只在函数调用期间存在，函数执行结束就会被清除，消耗`gas`少    |
| Storage   | 变量会永久存储在区块链                                       | 函数之外声明的变量默认为`storage`类型，消耗`gas`多           |
| calldata  | 将变量存储在内存                                             | 只能修饰输入参数，不能修改，函数间传递参数消耗`gas`少        |
| public    | 公共的                                                       | 修饰的状态变量会生成访问函数，修饰的函数可以被调用，内外部都可以调用 |
| private   | 私有的                                                       | 合约内部可用，不可被继承，比public节约gas                    |
| external  | 外部的                                                       | 只能通过外部读取函数，可以通过this调用但消耗gas，不可被继承  |
| internal  | 内部的                                                       | 合约内部可用，可以被继承                                     |
| payable   | 付费                                                         | 调用合约方法需要转账，标记的地址可以发送代币                 |
| view      | 可视的                                                       | 可以读取状态变量，不能修改状态变量                           |
| pure      | 纯函数                                                       | 不能读取状态变量，只能拥有局部变量，用户调用pure函数不用gas，合约调用需要支付gas |
| virtual   | 可重写的                                                     |                                                              |
| override  | 重写的                                                       | 修饰 `public` 变量会重写该变量的 get 方法                    |
| returns   | 返回值                                                       |                                                              |

- 值传递还是引用传递
  - 函数外的storage状态变量赋值给函数内的storage变量时，修改函数内storage变量**会影响**函数外的状态变量
  - 函数外的storage状态变量赋值给函数内的memory变量时，修改函数内memory变量**不会影响**函数外的状态变量
  - memory变量赋值给memory变量时，会影响原变量
  - 变量赋值给storage变量，会创建独立的副本

- Storage
  - 以太坊中采用的是插槽 slot 存储，一个slot 32个字节
  - slot 空间不足时会自动存入下一个slot
  - 操作的时候也是对整个slot进行操作
  - 基本数据类型是从slot中的低位向高位存储数据
  - 结构体和定长数组的存储会单独创建一整块slot进行存储
  - 映射mapping会通过占位符占据整块slot数组，然后value存储在keccak256(slot下标.key)
  - 动态数组会用数组长度占据1块slot数组，然后数据存储keccak256(slot下标.value)
  - bytes和string
    - 数据长度 <= 31字节：数据存高31位，最低一位存数据长度 * 2
    - 数据长度 >= 31字节：会存储一个数据长度 * 2 + 1，数据会另外存储

  - EVM会将有空余空间的slot填充到32位，这增加了gas费
  - 编译器会将多个元素合并到一个slot，所以需要保证这些元素是可以紧密存储的
    - 变量和结构体存储时可以考虑占满一个slot

- memory
  - 只能创建定长数组
  - 创建数组时要指定长度，创建的数组不能进行大小的变更操作
  - 在返回值中使用memory的话，是不会存在函数的默认内存空间中的，会开辟一块新的内存空间
  - 不定长的数据类型作为返回值需要加上memory
    - Solidity中的函数默认返回值存储在堆栈上，而堆栈空间是有限的。如果函数返回的数据类型是不定长的，并且长度超过堆栈的大小限制，那么将会导致栈溢出错误


## 数据

### 变量

- 分为状态变量，局部变量和全局变量
  - 状态变量：写在合约内函数外，如果没有提供修改的方法，那么它将永远的写在区块链上了
  - 局部变量：写在函数内，调用函数时就会在虚拟机中产生
  - 全局变量：内置变量

- 状态变量默认是internal
- 局部变量不能使用public

### 基本类型

| 数据类型                  | 说明         | 备注               | 默认值  |
| ------------------------- | ------------ | ------------------ | ------- |
| byte、bytes8、bytes32     |              | 固定长度，十六进制 | 0x00... |
| int、uint、uint8、uint256 | 整型、正整数 | int默认是256位的   | 0       |
| bool                      | 布尔         | true / false       | false   |
| string                    | 字符串       | 消耗的gas多        |         |

### 引用类型

| 数据类型         | 说明          | 备注                           | 默认值   |
| ---------------- | ------------- | ------------------------------ | -------- |
| address          | 地址信息      | 十六进制                       | 0x00地址 |
| mapping          | 映射 / hash表 | 不支持遍历，默认值0            |          |
| bytes / bytes1[] | 字节数组      | bytes的gas更便宜               |          |
| struct           | 结构体        | 创建对象不用new                |          |
| enum             | 枚举          | 默认值是第一个值，取值是取下标 |          |

- address

  - 在以太坊中存储的是20字节的值

  - address有成员变量，用payable修饰的地址多了 `transfer` 和 `send` 成员变量

| 方法   | 作用                           | 备注                              |
| ------ | ------------------------------ | --------------------------------- |
| push   | 将元素插入数组尾部             | 动态数组才可以使用该方法          |
| delete | 将数组指定位置的元素变为默认值 | delete 数组[下标]，不改变数组长度 |
| pop    | 弹出数组尾部元素               | 数组的长度会-1                    |
| length | 获取数组中元素个数             |                                   |

### 内置变量

| 变量名          | 含义                      | 返回值  |
| --------------- | ------------------------- | ------- |
| msg.sender      | 调用者地址                | address |
| msg.value       | 获取payable的转账金额     |         |
| msg.data        | 完整的调用信息calldata    |         |
| msg.gas         | 剩余的gas                 |         |
| msg.sig         | calldata的前4个字节       |         |
| block.blockhash | 获得最近256个区块的哈希值 |         |
| block.coinbase  | 当前块矿工地址            |         |
| block.diffculty | 当前块难度                |         |
| block.gaslimit  | 当前块gas限制             |         |
| block.number    | 当前块区块号              | uint    |
| block.timestamp | 当前块时间戳              | uint    |
| now             | 当前区块                  |         |
| gasleft         | 剩余的gas                 |         |
| tx.gasprice     | 当前交易的gas价格         |         |
| tx.origin       | 交易的发送者              |         |

- msg.data一般包含2个部分
  - 函数签名：函数名称 + 参数类型 => hash 后取前四位bytes值
  - 参数

## 流程控制

```solidity
function test() public {
	//if-else
	if(条件判断){
    	...
    }else if(条件判断){
    	...
    }else{
    	...
    }
    
    //三元
    条件判断 ? 返回值2 : 返回值2
    
    //for循环
    for(uint i = 0; i < 10; i++){
    	...
    }
    
    //while循环
    while(i < 10){
        ...
    }
    
    //do-while循环
    do{
        ...
    }while(i < 10);
}
```

## 继承

### 格式

```solidity
contract A{	
	//需要子类重写的方法
	function a() virtual{
	
    }
}
contract B is A{
	function b(){
	
    }
    //重写父类方法
    function a() override{
	
    }
}
```

- 修饰器也可以被继承，也可以重写

- 调用父合约

  - 通过父合约名.函数名
  - 通过super.函数名
    - 会执行最远的父合约函数

- 父类构造函数需要传参的情况

  - 可以在继承时在父类合约名后加上参数

    ```solidity
    contract C is  A(参数), B(参数){}
    ```

  - 可以在子类构造函数中传值

    ```solidity
    contract C is A, B{
    	constructor(参数1, 参数2) A(参数1) B(参数2){}
    }
    ```

### 多继承

```solidity
contract a{

}
contract b is a{

}
contract c is a{

}
contract d is b, c{

}
```

- 允许多继承，同名函数遵循最远继承规则
  - 多继承时父类的顺序应该由高到低
  - 多个父类都有的函数子类必须重写
  - 重写多个父类都有的函数时，需要标注所有父合约的名字
    - `override(父a，父b ...)`
- 钻石继承
  - 即父合约之间继承于同一个合约
  - 使用super时每一个父合约的函数都会被执行，但顶级合约的函数只会执行一次

## 事件

- 是EVM上对日志的抽象
- 接口订阅和监听
- 存储数据，每个大概消耗2,000 gas，链上存储至少需要20,000 gas
- 是一种写入方法，所以不能被标记为view / pure
- 事件的参数可以使用indexed标记，标记过的变量就可以在链外进行搜索查询
  - 最多只能标记三个变量

- 用法
  - 发生代币转移时
  - 部署合约时返回合约地址

### 格式

```solidity
contract a {
	//建立事件
    event 事件名(类型 [indexed] 参数a ...)
    //使用 emit 激活 / 调用事件
    emit 事件名
}
```

## 异常

#### 方法

| 方法    | 参数           | 作用                                      |
| ------- | -------------- | ----------------------------------------- |
| require | 条件、异常描述 | 要求达到对应的条件，不会消耗gas并终止执行 |
| assert  | 条件           | 断言，会消耗gas并继续执行                 |
| revert  |                | 直接抛出异常，不需要写条件，会消耗gas     |
| error   | 方法           | 自定义错误，可以携带信息，gas更低         |

#### 自定义错误

```solidity
error thorized(类型 变量名);
contract a {
	function method() public {
		revert thorized(变量);
    }
}
```

- 节约gas费
- 能够在错误中定义变量，即抛出异常时可以抛出变量，报错信息详细
- 可以定义在合约中或者合约外

## 函数

- 函数默认会分配4个32字节的内存空间

### 格式

```solidity
pragma solidity ^0.8.7;
contract A {
    //构造函数
    //在合约部署的时候会调用一次
    constructor(){

    }
	//普通函数
	//有返回值名的情况下函数体内可以不用写return语句，隐式返回
    function 函数名 [修饰符...] [returns (类型 返回值1 [, 类型 返回值2, ...])] {
        
    }
    //fallback函数
    fallback() external payable{}
    //receive函数
    receive() external payable{}
}
```

- fallback 和 receive函数
  - 都是作为兜底函数，即外部合约调用了不存在的函数时会调用这两个函数
  - 一般会添加payable关键字防止恶意调用
  - 调用时如果没传入数据会执行receive函数，前提是有定义receive函数，其他情况都是调用的fallback函数


### 内置函数

| 函数名                  | 作用                                         | 参数                                               | 返回值    | 备注                                                         |
| ----------------------- | -------------------------------------------- | -------------------------------------------------- | --------- | ------------------------------------------------------------ |
| creationCode            | 合约代码                                     |                                                    |           |                                                              |
| address                 | 获取地址                                     |                                                    |           |                                                              |
| balance                 | 获取合约余额                                 |                                                    |           |                                                              |
| transfer                | 转账                                         |                                                    |           | 余额不足会报错，2260gas                                      |
| send                    | 转账                                         |                                                    | bool      | 余额不足情况需要处理，2260gas                                |
| call                    | 转账 / 调用合约                              | {[value:ETH数],[gas:gas数]}(转账金额 / 二进制编码) | bool,data | 余额不足情况需要处理，支付全部gas                            |
| callcode                | 转账 / 调用合约                              |                                                    |           | 调用底层                                                     |
| delegatecall            | 转账 / 调用合约                              |                                                    |           | 可以发gas，不能发eth                                         |
| keccak256               | 将任何长度的数据转换为256位的哈希值          | 数据                                               | bytes32   | 哈希算法                                                     |
| create                  | 支付的币，内存中机器码开始的位置，机器码大小 | 预测合约部署后的地址                               | 不准确    |                                                              |
| create2                 | 常数，创建者地址，盐，机器码                 |                                                    | 8.0新特性 |                                                              |
| abi.encode              | 压缩                                         |                                                    | bytes     | 会将多个参数之间补0，避免hash碰撞                            |
| abi.encodeWithSignature | 获得二进制编码                               | 函数(参数类型)，参数                               |           |                                                              |
| abi.encodePacked        | 压缩                                         |                                                    | bytes     | 不定长的，有一定的压缩，如果参数是连续的字符串可能会出现hash碰撞 |
| ecrecover               | 获取消息的签名者地址                         | 加密信息，签名                                     | address   |                                                              |
| selfdestruct            | 自毁合约                                     | address                                            | /         | 强制发送剩余代币                                             |

### 合约外函数

- 在8.0之后允许函数定义在合约外

### 修饰器

```solidity
//权限校验
modifier auth(){
	require(
		//检验逻辑，返回布尔类型
	);
	_;
}
//代码复用(三明治结构)
modifier rep([参数]){
	//代码
	_;//被修饰函数中的代码
	//代码
}
//对应的方法上添加修饰器即可生效
function test() public auth rep([参数]){
	
}
```

## 合约

### 合约创建

```solidity
//直接创建合约对象
合约名 合约对象 = new 合约名();

//间接创建合约对象
address addr = new 合约名();
合约名 合约对象 = 合约名(address);

//工厂合约创建合约对象
contract A{}
contract AFactory{
	A[] public As;
	function createA(){
		As.push(new A())
    }
}
```

### 合约调用

- 外部用户直接调用合约
  - 合约对象直接调用出现错误时，调用方会出现错误并回滚

- 外部用户间接调用合约
  - call
    - 格式： `合约地址.call(abi.encodeWithSignature("函数名(参数类型1, 参数类型2)", 参数1, 参数2))`
    - 调用合约出现错误时，调用方不会回滚
    - call通过触发`fallback`或`receive`函数发送`ETH`
    - call调用会切换上下文，即进入到被调用的合约内部
    - 不建议直接通过call来调用合约，因为主动权给了被调用的合约，有安全风险，推荐通过创建合约对象来调用
  - delegatecall
    - 用法和call一致
    - 调用合约出现错误时，调用方不会回滚
    - 不会切换上下文，即还在调用方合约内部
    - 不能使用value方法
    - delegatecall有安全隐患，需要保证当前合约和目标合约的状态变量类型相同，并且目标合约安全
    - delegatecall 调用目标合约时，调用者是调用delegatecall的源头

- 合约调用合约

  - 需要有被调用合约的地址，且知道其函数信息才可以调用

    ```solidity
    contract A {
    	function testA1(address _B){
    		B(_B).testB1()
        }
        function testA2(B _B){
    		_B.testB1()
        }
        function testA3(address _B){
    		B(_B).testB2{value: msg.value}()
        }
    }
    contract B {
    	function testB1(){
    	
        }
        function testB2() payable{
    	
        }
    } 
    ```

### 代理合约

- 为了解决合约的更新问题，因为链上代码不可变和数据迁移的费用昂贵
- 委托调用不会改变逻辑合约中的值，改变的是代理合约的值，即使用逻辑合约的逻辑
- 代理模式将合约数据和逻辑分开，分别保存在不同合约中
  - 代理合约Proxy：存储所有相关的变量、逻辑合约的地址
  - 逻辑合约Logic：存储所有的函数，通过`delegatecall`执行，逻辑合约也需要定义和代理合约完全一样的变量（类型 / 顺序 / 名称），在这些变量之后可以定义独有的变量

### 接口合约

- 不清楚目标合约的代码的具体信息，但知道其函数名称参数返回值等等，就可以通过接口合约调用
- 类似模拟了一个目标合约

```solidity
//目标合约
contract TargetContract{
	//具体细节不明
	function A(){
		//详细代码
    }
}
//接口合约
interface ITargetContract{
	function A(){};
}
contract Test{
	function test(address _TargetContract){
		ITargetContract(_TargetContract).A();
    }
}
```

### 库合约

- 提取复用性强的代码作为库合约以便复用
- 一般会将函数定义为internal
- 可以使用 `using 库合约 for 类型` 使该类型可以直接调用库合约方法

### 异常处理

- 合约执行具有原子性，发生异常时消耗的汽油费不会退回
- 常见错误情况
  - 汽油费不够
  - 出现异常

## 内联汇编

### 简介

- 使用内联汇编自由度更高，性能更强，节约gas费
- 格式为使用 assembly{} 代码块
- 批量操作适合使用内联汇编
- 因为EVM时基于栈的虚拟机，想要难道栈顶之外的数据比较困难，所以要使用内联汇编
- 可以获取合约的代码，并加载到bytes变量中
- 可以判断一个地址是合约地址还是用户地址
- 函数操作需要结合地址和数据

### 格式

```solidity
//汇编代码块
assembly{
	//通过let关键字来定义变量，作用域只在当前大括号中
	let a := 1;
	//for循环
	for{let i := 0} lt(i, x) {i := add(i ,1)}{
		//循环体
	}
	//if
	if slt(x, 0){
		//。。。
    }
}
//例子：获取地址中的代码
function getCode(address _addr) public view returns (bytes memory code){
	assembly{
		//获取代码长度
		let size := extcodesize(_addr)
		//分配一块内存空间
		code := mload(0x40)
		//
		mstore(0x40, add(code, add(size, 0x20)))
		//把size存入该内存空间
		mstore(code, size)
		//获取代码并保存
		extcodecopy(_addr, add(code, 0x20), )
	}
}

```

### 方法

1. **算术和逻辑操作**：
   - **add**：将栈上的两个元素相加。
   - **sub**：从栈上的第二个元素中减去第一个元素。
   - **mul**：将栈上的两个元素相乘。
   - **div**：将栈上的第二个元素除以第一个元素（整数除法）。
   - **sdiv**：带符号的整数除法。
   - **mod**：取模运算。
   - **smod**：带符号的取模运算。
   - **addmod**：(x + y) mod z，加法后取模。
   - **mulmod**：(x * y) mod z，乘法后取模。
   - **exp**：指数运算。
   - **signextend**：符号扩展。
2. **比较和位操作**：
   - **lt**、**gt**、**slt**、**sgt**、**eq**：比较栈上的两个元素，分别对应小于、大于、带符号小于、带符号大于、等于。
   - **iszero**：如果栈顶元素为0，则返回1，否则返回0。
   - **and**、**or**、**xor**、**not**：位运算。
   - **shiftleft**、**shiftright**：位移操作。
3. **内存和存储操作**：
   - **mstore**、**mstore8**：将值存储到内存中。
   - **mload**：从内存中加载值，32字节
   - **sstore**：将值存储到合约存储中。
   - **push**、**pop**：栈操作，用于将值压入栈或从栈中弹出。
4. **控制流**：
   - **jump**、**jumpi**：跳转到指定位置。
   - **return**：结束执行并返回结果。
   - **revert**：停止执行。
   - **selfdestruct**（原名为`suicide`）：销毁合约并发送资金。
5. **栈操作**：
   - **dup**：复制栈顶元素。
   - **swap**：交换栈上的两个元素。
6. **日志和事件**：
   - **log0**、**log1**、**log2**、**log3**、**log4**：生成日志事件，用于外部监听和分析。
7. **系统操作**：
   - `blockhash`：获取指定区块的哈希。
   - `coinbase`：获取当前区块的矿工地址。
   - `timestamp`：获取当前区块的时间戳。
   - `number`：获取当前区块号。
   - `difficulty`：获取当前区块的难度。
   - `gaslimit`：获取当前区块的燃气上限。
   - `origin`：获取交易的发起者地址。
   - `caller`：获取当前函数的调用者地址。
   - `callvalue`：获取当前交易的金额（以wei为单位）。
   - `calldataload`、`calldatasize`、`calldatacopy`：访问输入数据。
   - `codesize`、`codecopy`：访问合约代码。
   - `gasprice`：获取当前交易的燃气价格。
   - `extcodesize`、`extcodecopy`：访问外部合约的代码。
   - `returndatasize`、`returndatacopy`：访问返回值数据。

## 8.0特性

### 安全数学 SafeMath

- 提高安全的数学运算
- 自动检测溢出

### Create2

- 加盐部署，可以获得合约部署出的合约地址

## 小案例

### 签名验证

```solidity
contract Verify{
    function verify(address _signer, string memory _message, bytes memory _sig) external pure returns(bool){
    	//1. 将消息进行哈希
    	bytes32 msgHash = getMsgHash(_message);
    	//2. 将哈希消息拼接上一段字符串进行二次哈希
    	bytes32 ethMsgHash = getEthMsgHash(msgHash);
        //3. 验证签名和签名者是否一致
        return recover(ethMsgHash, _sig) == _signer;
    }
    //哈希
    function getMsgHash(string memory _message) public pure returns(bytes32) {
    	return keccak256(abi.encodePacked(_message));
    }
    //二次hash
    function getEthMsgHash(bytes32 _message) public pure returns(bytes32) {
    	return keccak256(abi.encodePacked(
    		"\x19Ethereum Signed Message:\n32", _message
        ));
    }
    //取得加密消息中的签名用户地址
    function recover(bytes32 _msg, bytes memory _sig) public pure returns(address){
    	(bytes32 r, bytes32 s, uint8 v) = _split(_sig);
    	return ecrecover(_msg, v, r, s);
    }
    //分割签名取出r, s, v
    function _split(bytes memory _sig) internal pure returns (bytes32 r, bytes32 s, uint8 v){
    	//验证参数是否是65位的字符串
    	require(_sig.length == 65, "invalid signature length!")
    	//sig = r + s + v
    	assembly{
    		r := mload(add(_sig, 32))
    		s := mload(add(_sig, 64))
    		v := byte(0, mload(add(_sig, 96)))
    	}
    }
}
```

### 权限控制

```solidity
contract AccessControl{
	event GrantRole(bytes32 indexed role, address indexed account);
	event RevokeRole(bytes32 indexed role, address indexed account);
	//角色映射集合
	mapping(bytes32 => mapping(address => bool)) public roles;
	
	//角色
	bytes32 private constant ADMIN = keccak256(abi.encodePacked("ADMIN"));
	bytes32 private constant USER = keccak256(abi.encodePacked("USER"));
	
	//函数修饰器
	modifier OnlyRole(bytes32 _role){
		require(roles[_role][msg.sender], "Not Authorized!");
		_;
    }
    //给合约部署者管理员权限
    constructor(){
    	_grantRole(ADMIN, msg.sender);
    }
	
	//升级角色-内部调用
	function _grantRole(bytes32 _role, address _account) internal {
		roles[_role][_account] = true;
		emit GrantRole(_role, _account);
	}
	//升级角色
	function grantRole(bytes32 _role, address _account) external OnlyRole(ADMIN){
		_grantRole(_role, _account);
	}
	
	//撤销角色-内部调用
	function _revokeRole(bytes32 _role, address _account) internal {
		roles[_role][_account] = false;
		emit RevokeRole(_role, _account);
	}
	//撤销角色
	function revokeRole(bytes32 _role, address _account) external OnlyRole(ADMIN){
		_revokeRole(_role, _account);
	}
}
```

### 多签钱包

```solidity
contract MultiSigWallet {
    event Deposit(address indexed sender, uint amount);//收款事件
    event Submit(uint indexed txId);//提交交易事件
    event Approve(address indexed owner, uint indexed txId);//批准交易事件
    event Revoke(address indexed owner，uint indexed txId);//撤销批准事件
    event Execute(uint indexed txId);//交易执行事件
    
    //拥有者列表
    address[] public owners;
    //拥有者地址映射
    mapping(address => bool) public isOwner;
    //交易确认数
    uint public required;
    
    //交易结构
    struct Transaction {
    	address to;
    	uint value;
    	bytes data;
    	bool executed;
    }
    //交易记录
    Transaction[] public transactions;
    //同意交易的拥有者
    mapping(uint => mapping(address => bool)) public approved;
    //设置持有者和交易确认数
    constructor(address[] memory _owners, uint _required){
    	require(_owners.length > 0，"owners required");
    	require(
    		_required > 0 && _required <= owners.length,
    		"invalid required number of owners"
		);
        for (uint i; i < owners.length; i++) {
        	address owner = _owners[il;
            require(owner != address(0)，"invalidowner");
            require(!isOwner[owner]，"owner is not unique");
            isOwner[owner] = true;
            owners.push(owner);
        }
        required = required;
    } 
    //接受主币
    receive() external payable{
    	emit Deposit(msg.sender, msg.value);
    }
    
    modifier onlyOwner(){
    	require(isOwner[msg.sender], "Not Owner!");
    	_;
    }
    modifier txExists(uint _txId){
    	require(_txId < transactions.length, "Tx does not exist!");
    	_;
    } 
    modifier notApproved(uint _txId){
    	require(!approved[_txId][msg.sender], "Tx already approved!");
    	_;
    }
    modifier notExecuted(uint _txId){
    	require(!transactions[_txId].executed, "Tx already executed!");
    	_;
    }
    
    //提交交易
    function submit(address _to, uint _value, bytes calldata _data) external onlyOwner{
        transactions.push(Transaction({
        to: to,
        value: value,
        data: data,
        executed: false
        }));
        emit Submit(transactions.length - 1);
    }
    //批准交易
    function approve(uint txId) external
    	onlyOwner 
    	txExists( txId)
        notApproved( txId)
        notExecuted( txId)
    {
		approved[ txId][msg.sender] = true;
		emit Approve(msg.sender, txId);
	}	
	//获取某个交易的批准数量
    function _getApprovalCount(uint _txId) private view returns (uint count){
        for(uint i; i < owners.length; i++) {
            if (approved[_txId][owners[i]]) {
            	count += 1;
            }
        }
    }
    //执行交易
    function execute(uint txId) external txExists( txId) notExecuted( txId){
        require(_getApprovalCount(_txId) >= required, "approvals < required");
        Transaction storage transaction = transactions[_txId];
        transaction.executed = true;
        (bool success，) = 
        	transaction.to.call{value: transaction.value}(transaction.data);
        require(success，"tx failed");
        emit Execute(_txId);
    }
    //撤销批准
    function revoke(uint _txId) external onlyOwner txExists( txId) notExecuted(_txId){
        require(approved[_txId][msg.sender],"tx not approved");
        approved[_txId][msg.sender] = false;
        emit Revoke(msg.sender，_txId);
    }
}
```

### 荷兰拍卖

```solidity
//随着时间价格越来越低
//使用NFT标准ERC721
contract DutchAuction {
	uint private constant DURATION = 7 days;
    IERC721 public immutable nft;
    uint public immutable nftId;
    address payable public immutable seller;
    uint public immutable startingPrice;
    uint public immutable startAt;
    uint public immutable expiresAt;
    uint public immutable discountRate;
    
    constructor(
        uint _startingPrice;
        uint _discountRate,
        address _nft,
        uint _nftId
    ){
        seller = payable(msg.sender);
        startingPrice = _startingPrice;
        discountRate = _discountRate;
        startAt = block.timestamp;
        expiresAt = block.timestamp + DURATION;
        require(
            _startingPrice >= _discountRate * DURATION,
            "starting price < discount"
        );
        nft = IERC721(_nft);
        nftId = _nftId;
    }
}
```

## 开发

- 智能合约是用来编写控制逻辑的
- 发布到区块链上的交易不一定都是成功执行的
  - 为了扣除汽油费
- 智能合约可以获得的信息比较有限
  - 因为每个节点的环境不同

### 部署

1. 将代码完成后编译成bytecode
2. 发起一个转账交易到0x0地址
   - 转账金额为0
   - 支付汽油费
   - 合约的代码放在data域中

### 测试

- 估算gas费
  1. remix在部署后执行函数调用会估算gas费
  2. 使用hardhat-gas-reporter插件生成gas报告

## 安全

### 常见漏洞

- 重入攻击

- 权限管理漏洞

### 注意事项

- 调用其他合约的方法时要小心被反调用
- 要记得写fallback函数
- 转账
  - 需要先把账户清零，再进行转账

## 审计
