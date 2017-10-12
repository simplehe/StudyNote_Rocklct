## uml概述
UML 作为一种模型语言，它使开发人员专注于建立产品的模型和结构，而不是选用什么程序语言和算法实现

UML 的目标就是 UML 被定义为一个简单的建模机制，帮助我们按照实际情况或者按照我们需要的样式对系统进行可视化；提供一种详细说明系统的结构或行为的方法；给出一个指导系统构造的模板；对我们所做出的决策进行文档化。

UML支持一下9中建图模型。

 - 用例图(Use case diagram)
 - 类图(Class diagram)
 - 对象图(Object diagram)--COMET没有使用该图
 - 顺序图(Sequence diagram)
 - 协作图(Communication diagram)
 - 状态机图(State Machine diagram)
 - 活动图(Activity diagram)
 - 组件图(Component diagram)
 - 配置图(deployment diagram)


### 协作图/通信图(collaboration diagram/communication diagram)

通信图在UML1.x中被称为协作图。

协作图也是互动的图表。他们像序列图一样也传递相同的信息，但他们不关心什么时候消息被传递，只关心对象的角色。

![](image/uml12.gif)

对象角色矩形上标有类或对象名（或者都有）。类名前面有个冒号（：）。

协作图的每个消息都有一个序列号。顶层消息的数字是1。同一个等级的消息（也就是同一个调用中的消息）有同样的数字前缀，再根据他们出现的顺序增加一个后缀1，2等等。
