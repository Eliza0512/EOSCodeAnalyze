EOS整体构架

## 用户与链的交互过程

![EOSArchitecture](pics/582e059-411_DevRelations_NodeosGraphic_Option3.png)

###  program (应用层)
- `nodeos` (node + eos = nodeos) - 节点核心
- `cleos` (cli + eos = cleos) - 用户指令集
- `keosd` (key + eos = keosd) - 秘钥管理，签名等

### plugins（插件层）

支持动态加载相关组件，实现了应用层的业务逻辑和区块链底层实现的解耦，同时为应用开发者提供友好的API接口，比较重要的有以下几个插件：

chain_plugin

http_plugin

net_plugin

producer_plugin

### libraries（库函数层）

为应用层和插件层提供基础能力，实现了区块链的底层关键技术，例如，交易处理，生产区块，加密功能，文件IO操作，网络通信能力等等；

- appbase：为一系列的插件编译提供了一个框架，他可以确保插件正常配置、初始化、启动、关闭这一个流程。 [详细展开](appbase.md)

- builtins：compiler

- chain：这里面包含有eos作为区块链的核心内容 [详细展开](chain.md)

- chainbase：是为了满足区块链应用设计的一个数据库，但是也使用于任意需要一个鲁棒性较高的交易数据库。[详细展开](chainbase.md)

- fc  基础功能函数（与链无关）[详细展开](fc.md)

  - compress
  - container
  - crypto
  - exception
  - inerprocess
  - io
  - log
  - network
  - reflect
  - rpc

- softfloat：ARM代码编译时的软浮点(soft-float)  与<u>虚拟机</u>相关，不做重点展开

  > 编译器把浮点运算转换成浮点运算的函数调用和库函数调用，没有FPU的指令调用，也没有浮点寄存器的参数传递。浮点参数的传递也是通过ARM寄存器或者堆栈完成。 现在的Linux系统默认编译选择使用hard-float，即使系统没有任何浮点处理器单元，这就会产生非法指令和异常。因而一般的系统镜像都采用软浮点以兼容没有VFP的处理器。

- testing：测试

- wabt：The WebAssembly Binary Toolkit <u>虚拟机</u>相关，不做重点展开

- wasm-jit：This is a standalone VM for WebAssembly. <u>虚拟机</u>

- yubihsm：外部硬件安全模块


## 用户与dapp交互的过程

![eosDapp](pics/a7aba6a-Intro_Diagram_-_web_app__development.svg)

- eosjs：Javascript API SDK 用于和eosio底层的RPC API通讯

- Demux：将EOS事件路由到 可query的数据库中，同时可能会触发一下附效应



# 分部介绍

## nodeos

官方文档：https://developers.eos.io/eosio-nodeos/reference

从main函数开始，程序大致分为三部分：选项配置、加载插件、启动程序

## cleos

cleos是一个命令行工具，用于和区块链数据交互以及管理钱包，从main函数开始，

程序大致分为三部分：创建主命令和选项、创建子命令和选项、解析用户参数后调用对应命令的回调函数。

所有命令都必须包含主命令cleos，然后可以创建子命令和选项，例如cleos create，同时可以为子命令继续创建子命令和选项，例如：

``` shell
./cleos create account [OPTIONS] creator name OwnerKey ActiveKey
```


## keosd

keosd钱包管理模块的处理流程和nodeos类似，从main 函数开始，程序大致分为三部分：选项配置、加载插件、启动程序，主要的功能由wallet_plugin、wallet_api_plugin、http_plugin这三个插件完成，具体流程不再赘述。







**参考文献**

https://www.8btc.com/article/202965

https://www.8btc.com/article/206182

https://www.8btc.com/article/209141

https://www.8btc.com/article/213989