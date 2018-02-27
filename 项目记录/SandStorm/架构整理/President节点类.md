## President节点类
President负责对整个集群进行管理

主要功能有：

 - 负责集群的启动和调度
 - 负责监视Manager的执行情况
 - 负责向Manager分配任务
 - 负责根据集群的执行情况，决定消息分发策略的执行方式。

President和Manager之间的关系图：

![](image/structure2.jpg)

这里不采取President主动获取节点状态，而是由**Manager主动推送状态**。Manager负责对Executor进行管理。

### 继承自CommandServer类

President节点毫无疑问充当着server的角色，所以自然继承自CommandServer类()

### 封装变量
内部封装了一些成员变量

 - _configuration变量，一个Map，用来存储配置文件的key和value的。
 - 整形ManagerCount，表示需要管理几个Manager。


### 初始化\&构造
President有两个构造函数，一个是根据Configuration进行构造。一个根据NetAddress进行构造。

利用配置文件构造的话，也会先从配置文件中读取到ip和port然后调用NetAddress的构造方法。

然后会从配置文件中读取到需要管理的Manager数量ManagerCount

President的构造方法中，用OnCommand方法注册了几个消息(Message),这样一旦节点收到对应的Message就会去调用相应的方法。

### 成员方法

President实现了几个成员方法，OnJoin,
