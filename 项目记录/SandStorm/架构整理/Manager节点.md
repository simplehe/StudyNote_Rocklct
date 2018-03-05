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

ReserveExecutors用来重置Executor容器。把容器初始化成对应的数量。并且初始化出bolt数量的消息队列，比如有3个bolt，就初始化出3个消息队列。然后初始化Manager内部封装的OutputDispatcher并且启动它

InitEvent用来初始化绑定回调函数。

#### Manager加入集群
Manager初始化完成后，会立刻调用JoinPresident方法，试图加入集群。

JoinPresidetn会调用NetConnector的Connect方法，并且传递进去一个回调函数。

Connect方法会连接President节点，连接成功后就会调用回调函数。回调函数的内容就是向President节点发送Join命令。新建一个JoinCommand然后发送过去。

当这个Command发送成功时，必然会回收到一个Response类，我们将接受到的字节数据转换为Response。然后我们会调用回调方法处理这个Response。

如果发现这个Response是Successful，那这时说明Manager正式加入集群了，此时调用Manager的StartListen(继承自CommandServer)开始监听。
