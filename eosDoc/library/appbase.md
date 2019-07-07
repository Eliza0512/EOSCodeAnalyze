

> 为应用层和插件层提供基础能力，实现了区块链的底层关键技术，例如，交易处理，生产区块，加密功能，文件IO操作，网络通信能力等等

# appbase/application.cpp

应用层基础配置与启动

## 常见方法 

### version相关

- application::set_version(uint64_t version) 
- uint64_t application::version() const
- string application::version_string() const 

