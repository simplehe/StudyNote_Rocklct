## koa-views
我们用这个koa-views中间件用于后端渲染模板。

<a>https://github.com/queckezz/koa-views</a>


我们先看一下一个简单的官方例子

``` javascript
var views = require('koa-views');

app.use(views(__dirname, { map: {html: 'nunjucks' }}))

// render `user.html` with nunjucks
app.use(async function (ctx) {
  await ctx.render('user.html')
})
```

我们这里use了views，views接收一个地址，和一个设置参数。好比这个设置对象(opts){ map: {html: 'nunjucks' }}，这个设置的意思是,s所有html结尾的文件，都会用nunjucks这个渲染引擎去渲染。

注意代码中的__dirname也必须要添加，是一个代表当前环境的变量


上面代码我们还看到，第二次use了一个中间件，调用了上下文对象ctx.render方法来渲染一个网页。

这里我们要注意两个事情了

 - **ctx原本没有render方法，我们再use了views(....)之后才添加进去的**
 - **由于我们必须要有render方法，才能顺利渲染，所以使用这个中间件的时候，必须在任何router路由分配之前注入执行这个中间件**