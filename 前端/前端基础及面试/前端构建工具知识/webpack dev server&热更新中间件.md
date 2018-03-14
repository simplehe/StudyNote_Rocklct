## webpack dev server
webpack-dev-server是一个小型的静态资源开发服务器，只用于开发环境。

这个服务器包，通过

    npm install webpack-dev-server --save-dev

来安装

在项目目录下运行

    node_modules/.bin/webpack-dev-server

则会在默认8080端口开启一个开发服务器。这个开发服务器支持live reload，简单地在项目根目录运行webpack-dev-server，然后打开html，修改任意关联的源文件并保存，webpack编译就会运行，并在运行完成后通知浏览器刷新。

和直接在命令行里运行webpack不同的是，webpack-dev-server会把编译后的静态文件全部保存在内存里，而不会写入到文件目录内。这样，少了那个每次都在变的webpack输出目录

如果在请求某个静态资源的时候，webpack编译还没有运行完毕，webpack-dev-server不会让这个请求失败，而是会一直阻塞它，直到webpack编译完毕。这个对应的效果是，如果你在不恰当的时候刷新了页面，不会看到错误，而是会在等待一段时间后重新看到正常的页面，就好像“网速很慢”。

webpack-dev-server的功能看上去就是我们需要的，但如何把它加入到包含后端服务器的Express项目里呢？

### webpack-dev-middleware和webpack-hot-middleware

以上两个都是express的第三方中间件。

所谓express的中间件

**中间件函数能够访问请求对象 (req)、响应对象 (res) 以及应用程序的请求/响应循环中的下一个中间件函数。下一个中间件函数通常由名为 next 的变量来表示。**

<a>http://expressjs.com/zh-cn/guide/using-middleware.html</a>

当我们需要自己写服务器的时候，就不能用别人已经写好用来测试的小型服务器了。也就是说在自定义的express后台框架中间插入中间件，就可实现热替换。

常规的中间件使用方法如下：
``` javascript

var app = express();

app.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next();
});

```

app.use里传入中间件函数来作执行。

那么结合用法，如若使用第三方的webpack插件，则是


``` javascript

app.use(webpackDevMiddleware(compiler, {
  noInfo: true,
  // 如果false，将会给你列出一大堆无聊的信息。
  
  publicPath: config.output.publicPath,
  stats: {
    colors: true
  }
}));
app.use(webpackHotMiddleware(compiler));
});

```