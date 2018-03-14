## koa路由
一般写koa路由，我们可以采取一般写法
``` javascript
app.use(function *(){
    //我是首页
    if(this.path==='/'){
    
    }
});

app.use(function *(){
    //我是详情页
    if(this.path==='/detail'){
    
    }
});
```

采用this.path分配处理方式，但是这种写法很挫不健壮。

所以我们一般采用koa-router这个中间件来处理。

注入koa-router中间件:

``` javascript
var router = require('koa-router');
app.use(router(app));
```

有了 koa-router 我们就可以像写 express 路由一样，写 koa 应用页面

``` javascript
app.get('/', function *(next) {
    //我是首页
    //this 指向请求
});
app.get('/detail/:id', function *(next) {
    //我是详情页面
    //:id 是路由通配规则，示例请求 /detail/123 就会进入该 generator function 逻辑
    var id = this.params.id; //123
});
```

除了最常用的get()外，还有 post() 、put() 、patch() 、delete()，后面会结合具体业务场景讲解。

koa-router 拥有丰富的 api 细节，用好这些 api ，可以让页面代码更为优雅与可维护。


### 更具体的koa-router用法
有关这个中间件，请看
<A>https://github.com/alexmingoia/koa-router/tree/master/</a>

基于koa2的基本用法

``` javascript
var Koa = require('koa');
var Router = require('koa-router');

var app = new Koa();
var router = new Router();

router.get('/', function (ctx, next) {...});

app
  .use(router.routes())
  .use(router.allowedMethods());
```

router也有use方法
```
// session middleware will run before authorize
router
  .use(session())
  .use(authorize());

// use middleware only with given path
router.use('/users', userAuth());

// or with an array of paths
router.use(['/users', '/admin'], userAuth());

app.use(router.routes());
```


其他用法具体参见文档