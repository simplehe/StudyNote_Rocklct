## Manager节点类

Manager节点类，用来管理Executor。注意Manager类不单纯是个Client，同时也是个Server，也要监听来自其他节点的消息。

内部也封装了许多成员变量：

 - _managerConfiguration 保存manager配置
 - _presidentConnector 和President节点进行连接的NetConnector
 - _presidentClient 基于命令模式的Client类
 - spoutExecutors/boltExecutors的 Executors spout和Executor容器
 - spoutCollectors/boltCollectors 消息源和消息处理器的数据收集器
 -
