## node中的http模块

传统http服务器会由Aphche，Nginx，IIS之类的软件担任，但是nodejs并不需要，nodejs提供了http模块，自身就可以用来构建服务器，而且http模块是由C++实现的，性能可靠。我们在nodejs中的教程或者书籍中常常会通过一个简易的http服务器来作为开头的学习。

### Server对象
可以通过http.createServer()方法返回一个http.Server对象，这个对象默认添加了一个“request”事件监听。如若使用new Http.Server()来创建这个对象的话，如下

``` javascript
var http=require("http");
var server=new http.Server();

server.on("request",function(req,res){
    res.writeHead(200,{
        "content-type":"text/plain"
    });
    res.write("hello nodejs");
    res.end();
});
server.listen(3000);
```

所以直接用createServer相当于更方便。

#### Server对象基于事件
这个要图http.Server是一个基于事件的服务器，她是继承自EventEmitter。http.Server提供一下事件

 1. request:客户端请求到来时，该事件被触发，提供两个参数req和res，表示请求响应信息，是最常用的事件。
 2. connection：当TCP连接建立时，该事件被触发，提供一个参数socket，是net.Socket的实例
 3. close：当服务器关闭时，触发事件（注意不是在用户断开连接时）

request事件是最常用的，而参数req和res分别是http.IncomingMessage和http.ServerResponse的实例

### 常用模块
路由解析经常用到一些模块。

比如解析URL需要用到Node.js提供的url模块，它使用起来非常简单，通过parse()将一个字符串解析为一个Url对象：

``` js
'use strict';

var url = require('url');

console.log(url.parse('http://user:pass@host.com:8080/path/to/file?query=string#hash'));

```

```

结果如下：
Url {
  protocol: 'http:',
  slashes: true,
  auth: 'user:pass',
  host: 'host.com:8080',
  port: '8080',
  hostname: 'host.com',
  hash: '#hash',
  search: '?query=string',
  query: 'query=string',
  pathname: '/path/to/file',
  path: '/path/to/file?query=string',
  href: 'http://user:pass@host.com:8080/path/to/file?query=string#hash'
}

```

处理本地文件目录需要使用Node.js提供的path模块，它可以方便地构造目录：

``` js
'use strict';

var path = require('path');

// 解析当前目录:
var workDir = path.resolve('.'); // '/Users/michael'

// 组合完整的文件路径:当前目录+'pub'+'index.html':
var filePath = path.join(workDir, 'pub', 'index.html');
// '/Users/michael/pub/index.html'
```
