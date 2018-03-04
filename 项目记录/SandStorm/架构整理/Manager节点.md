## Manager节点类

Manager节点类，用来管理Executor。注意Manager类不单纯是个Client，同时也是个Server，也要监听来自其他节点的消息。

内部也封装了许多成员变量：

 - \_managerConfiguration 保存manager配置
 - \_presidentConnector 和President节点进行连接的NetConnector
 - \_presidentClient 基于命令模式的Client类
 - spoutExecutors/boltExecutors的 Executors spout和Executor容器
 - spoutCollectors/boltCollectors 消息源和消息处理器的数据收集器


### Manager初始化流程

Manager会在初始化时依次调用4个方法，InitPresident，InitSelfContext，ReserveExecutors，InitEvents。

InitPresident的任务是初始化自身的NetConnector组件，并创建出一个基于消息回调机制的CommandClient

InitSelfContext用来创建出保存自身信息的上下文对象。并且初始化Manager自身的几个Spout，bolt的Executor容器(vector).

ReserveExecutors用来重置Executor容器。把容器初始化成对应的数量。并且初始化出bolt数量的消息队列，比如有3个bolt，就初始化出3个消息队列。然后初始化Manager内部封装的OutputDispatcher
