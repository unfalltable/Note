### 简介

- Flow平台开发
- 面向资源编程
- 使用playground开发
  - https://play.flow.com/


### 结构

```Cadence
//合约
pub contract HelloWorld {
	//初始化函数
    init(){
    	self.str = "hello!"
    }
}
```

### 调用

```Cadence
//引入合约
import HelloWorld from 0x05

//主函数
pub fun main(){
  //主函数中调用合约
}
```

## 