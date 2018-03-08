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

#### OnJoin
收到Join命令时，说明有Manager节点请求加入President节点，这时President要保存Manager的信息，并且新创建一个CommandClient，然后这个Client会向Manager发送心跳信息 **HeartBeat**。每隔 **一秒钟** 发送一条心跳信息，超过 **五次** 超时没有回应，则被认为是失去了连接lost connection。

最后，如果发现已经有足够数量的Manager加入到集群中，则开始分发topology。先获取TopologyLoader的一个实例，然后读取用户编写的so文件，获取GetTopology方法。获取Topology。接下来就要分发任务了

#### submitTopology
当manager数量齐全的时候，我们就要分发任务给各个节点。

执行submitTopology时，会把Topology里set的SpoutDeclaer和BoltDeclaer全部取出，这里**一个Declaer其实就是一个大Task，大Task可以分解成具体每个Excutor执行的小Task**。

从而可以得到两个list，一个list是Spout任务的list，一个是bolt任务的list。这里要注意，我们拿到topology之后，还会根据设置的对应的ParallismHint来决定某个任务由多少个线程来执行。

然后我们就可以分配这些task。分配任务的流程是，分别先得到Spout和Bolt的TaskDeclaer的List。然后我们要做的是遍历President中注册的Manager，然后查找看这个Manager有没有没有使用的Executor，有空闲的Executor的话，就从任务List中取出一个Executor然后分配过去。

President维护了两个Map，nameToSpout和nameToBolt，用来绑定taskName和对应的TaskInfo。所以我们知道Declarer名的时候，就可以检索这两个map，从而得到Declarer对应的几个TaskInfo进而找到ManagerContext。

所以AllocateTask最后都会返回一个Map(就是上面说的NameToXX)，这个map的key键是DeclarerName，对应的value值是一个vector容器，这个容器里装的就是Declarer对应Tasks的TaskInfo。所以我们可以检索

分配好任务以后，会执行CalculateTaskPaths来对每个任务进行路径的计算。计算的时候会拿到BoltDeclarers，然后枚举每一个Declarer。对于每一个Declarer来说，有soureTask和destTask，sourceTask就是这个Declarer的上一个Declarer的名字，比如也许是某个Spout.而destTask其实是指代自身。

##### 数据发送策略
在CalculateTaskPath之中

数据的从一个节点传输到下一个节点，分为三种策略：

 - Global
 - Field
 - Random


Global模式下，任务会随机选择合法的下一个Executor。首先，我们先从NameToMap中检索出sourceTask的全部小task(对应着某个Manager节点的某个Executor),Global策略下，就是从destTask，也就是自己，随机选择几个合法的Executor(小task)然后分配给各个小sourceTask。 **注意，这里的destExecutor在President最开始就给分配好了，是固定不变的。** 也就是说，一旦President执行完CalculatePath之后，Global策略下某个Executor传给的下一个Executor是固定的。只是一开始分配的时候是随机的。


Field模式下，会先给sourceTask设定字段，设定的字段就是当前destTask所属Declarer(当前task)的field字段。

Random策略，跟Global有区别，那就是在分配路径时，会先直接把destTask的所有tasks都设置进去sourceTask的destExecutors容器中，等到**真正传输时再随机选择**。

#### ConnectionChecker
当提交完topology，也就是同步节点数据完成之后。就会开始CheckConnection的操作，也就是试探各个Manager的连接有没有恢复。

这个操作会新建一个线程来执行，新建的线程会进入一个无限循环，然后枚举每一个ManagerContext元数据，找出对应的Manager发送心跳数据包。
