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

President节点毫无疑问充当着server的角色，所以自然继承自CommandServer类

### 封装变量
内部封装了一些成员变量

 - \_configuration变量，一个Map，用来存储配置文件的key和value的。
 - 整形ManagerCount，表示需要管理几个Manager。


### 初始化构造
President有两个构造函数，一个是根据Configuration进行构造。一个根据NetAddress进行构造。

利用配置文件构造的话，也会先从配置文件中读取到ip和port然后调用NetAddress的构造方法。

然后会从配置文件中读取到需要管理的Manager数量ManagerCount

President的构造方法中，用OnCommand方法注册了几个消息(Message),这样一旦节点收到对应的Message就会去调用相应的方法。

### 成员方法

President实现了几个成员方法，OnJoin,OnAskField,OnOrderID等等，这些方法都是用来和对应的Command进行绑定的。在President的初始化函数中，会调用OnCommand方法把这些成员函数的实现一一绑定到handlers map中。这样当接收到具体命令时，就会根据具体情况调用这些函数。

### President启动流程
当从tool包下的StartPresident启动了President以后。

President会进入初始化，也就是就读取configure文件的配置，然后绑定回调方法到自身，包括OnConnection和OnCommand方法。

然后会调用President继承自CommandServer的StartListen方法。StartListen方法会让自身具备的NetListener(用来监听信息的组件)绑定OnConnection方法，这个方法当Connection以建立好之后便会执行。

这个绑定的OnConnection方法的内容，就是继续在Connection上绑定一个OnData函数，当收到数据时立刻验证它是不是一个Command，如果是，那就进行相应的处理(交给之前绑定的handlers)，然后发回Response。

这基本就是President的整体流程
