## Socket.io
Socket.IO是一个开源的WebSocket库，它通过Node.js实现WebSocket服务端，同时也提供客户端JS库。Socket.IO支持以事件为基础的实时双向通讯，它可以工作在任何平台、浏览器或移动设备。

Socket.IO支持4种协议：WebSocket、htmlfile、xhr-polling、jsonp-polling，它会自动根据浏览 器选择适合的通讯方式，从而让开发者可以聚焦到功能的实现而不是平台的兼容性，同时Socket.IO具有不错的稳定性和性能。

### 基本用法
Websocket协议可以保证客户端和服务端实现双端通信，而Socket.io正可以做到这一点。所以我们不难知道他大致的用法，在服务端启用Socket并且监听，在客户端也启用Socket并且去连接服务端，这样就实现了一个WebSocket连接，可以进行一些实时的操作。

首先要注意，在node环境下，服务端跟客户端装的包是不一样的。

服务端需要install socket.io的包，具体用法可以参考文档
<a>https://socket.io/docs/</a>

用法有多中，比较经典的一个便是将socket.io绑定在httpServer的一个实例上。

``` javascript
var app = require('http').createServer()
var io = require('socket.io')(app);

app.listen(80);
```

上述代码先是创建了一个http server的实例，然后用socket.io服务绑定了它。

最后这个server实例监听窗口，这样便算是大功告成。

接着便是为socket.io绑定一些事件。

两个函数需要注意，on函数和emit函数。

on函数用来绑定对应的事件，emit函数用来触发一个事件。

socket.io 提供了三种默认的事件（客户端和服务器都有）：connect 、message 、disconnect 。当与对方建立连接后自动触发 connect 事件，当收到对方发来的数据后触发 message 事件（通常为 socket.send() 触发），当对方关闭连接后触发 disconnect 事件。

写了下代码测试
``` javascript
io.on('connection',function (socket) {
   socket.emit('hello',"hello client");
    socket.on('msg',function (data) {
        console.log("oh!I receieve a msg: "+data);
    });
});
```

io是socketio的实例，它先绑定监听了一个connection事件，一旦有客户端连接就启动回调函数，并且得到客户端的socket。

连接一旦成功以后，对这个socket发射一个hello事件，并且同时对这个socket监听一个msg事件，对方触发了msg事件的情况下将会执行回调函数。

以下为客户端代码
``` javascript
var io = require('socket.io-client');
var socket = io.connect('http://127.0.0.1:6666');
socket.on('hello',function (data) {
   console.log("a hello from server: " + data);
    socket.emit('msg','server I find you!!!!');
});
```

注意客户端导的包是**socket.io-client**,或者在浏览器也可以用script引入socket.io.js文件。

所以需要提前install 这个包。

这里io先连接了服务器对应端口，然后对这个服务器socket监听从服务器发过来的hello事件。一旦接受到这个时间，便像服务器socket发送信息。

