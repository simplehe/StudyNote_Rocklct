## Flux

<A>http://www.ruanyifeng.com/blog/2016/01/flux.html</a>

Flux是Facebook用来构建客户端Web应用的应用架构。它利用单向数据流的方式来组合React中的视图组件。它更像一个模式而不是一个正式的框架，开发者不需要太多的新代码就可以快速的上手Flux。

简单说，Flux 是一种架构思想，专门解决软件的结构问题。它跟MVC 架构是同一类东西，但是更加简单和清晰。
Flux存在多种实现（至少15种），本文采用的是Facebook官方实现。

首先，Flux将一个应用分成四个部分。
 
- View： 视图层
- Action（动作）：视图层发出的消息（比如mouseClick）
- Dispatcher（派发器）：用来接收Actions、执行回调函数
- Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面

![](image/flux1.png)

Flux 的最大特点，就是数据的"单向流动"。

- 用户访问 View
- View 发出用户的 Action
- Dispatcher 收到 Action，要求 Store 进行相应的更新
- Store 更新后，发出一个"change"事件
- View 收到"change"事件后，更新页面

上面过程中，数据总是"单向流动"，任何相邻的部分都不会发生数据的"双向流动"。这保证了流程的清晰。