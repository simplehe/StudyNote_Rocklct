## node框架对比

### sail.js
Sails.js 就像是 Node.js 平台上的 Rails 框架。这是一个可靠可伸缩的开发框架，面向服务的架构，提供数据驱动的 API 集合。用来开发多玩家游戏、聊天应用和实时面板引用非常方便，也可用于开发企业级 Node.js 应用。

Sails.js 基于 Node.js, Connect, Express 和 Socket.io 构建。

### express
Express 是一个简洁而灵活的 Node.js Web应用框架, 提供一系列强大特性帮助你创建各种 Web 应用。Express 不对 Node.js 已有的特性进行二次抽象，我们只是在它之上扩展了 Web 应用所需的功能。丰富的 HTTP 工具以及来自 Connect 框架的中间件随取随用，创建强健、友好的 API 变得快速又简单。

### koa

Koa 是下一代的 Node.js 的 Web 框架。由 Express 团队设计。旨在提供一个更小型、更富有表现力、更可靠的 Web 应用和 API 的开发基础。

Koa可以通过生成器摆脱回调，极大地改进错误处理。Koa核心不绑定任何中间件，但提供了优雅的一组可以快速和愉悦地编写服务器应用的方法。

### koa vs express
Koa 相对更为年轻， 是 Express 原班人马基于 ES7 新特性重新开发的框架，框架自身不包含任何中间件，很多功能需要借助第三方中间件解决，但是由于其基于 ES7 async 特性的异步流程控制，解决了 "callback hell" 和麻烦的错误处理问题，大受开发者欢迎。Koa 目前已经升级到了 2.x
