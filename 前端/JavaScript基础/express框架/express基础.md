## express基础
Express是目前最流行的基于Node.js的Web开发框架，可以快速地搭建一个完整功能的网站。

### HelloWorld
Express的底层其实是对node的http模块的封装。所以很简单就能启动一个http服务。

``` js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello world!');
});

app.listen(3000);
```

### 中间件

express有中间件这个概念，它最大的特点就是，一个中间件处理完，再传递给下一个中间件。在处理请求过程中，会调用一系列的中间件。

每个中间件可以从App实例，接收三个参数，依次为request对象（代表HTTP请求）、response对象（代表HTTP回应），next回调函数（代表下一个中间件）。每个中间件都可以对HTTP请求（request对象）进行加工，并且决定是否调用next方法，将request对象再传给下一个中间件。

一个不进行任何操作、只传递request对象的中间件，就是下面这样。

``` js
function uselessMiddleware(req, res, next) {
  next();
}
```

#### use方法
use是express注册中间件的方法，它返回一个函数。下面是一个连续调用两个中间件的例子。

``` js
var express = require("express");
var http = require("http");

var app = express();

app.use(function(request, response, next) {
  console.log("In comes a " + request.method + " to " + request.url);
  next();
});

app.use(function(request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Hello world!\n");
});

http.createServer(app).listen(1337);
```

#### 判断路由
中间件里可以判断路由来进行分发

``` js
app.use("/home", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Welcome to the homepage!\n");
});
```

#### use方法的别名
针对不同的请求，Express提供了use方法的一些别名。比如，上面代码也可以用别名的形式来写。

``` js
var express = require("express");
var http = require("http");
var app = express();

app.all("*", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  next();
});

app.get("/", function(request, response) {
  response.end("Welcome to the homepage!");
});

app.get("/about", function(request, response) {
  response.end("Welcome to the about page!");
});

app.get("*", function(request, response) {
  response.end("404!");
});

http.createServer(app).listen(1337);
```

复杂的模式匹配也支持

``` js
// 匹配/commits/71dbb9c
// 或/commits/71dbb9c..4c084f9这样的git格式的网址
app.get(/^\/commits\/(\w+)(?:\.\.(\w+))?$/, function(req, res){
  var from = req.params[0];
  var to = req.params[1] || 'HEAD';
  res.send('commit range ' + from + '..' + to);
});
```
