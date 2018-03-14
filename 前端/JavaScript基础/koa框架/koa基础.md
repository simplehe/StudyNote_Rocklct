## koa
<A>http://javascript.ruanyifeng.com/nodejs/koa.html</a>

Koa是一个类似于Express的Web开发框架，创始人也是同一个人。它的主要特点是，使用了ES6的Generator函数，进行了架构的重新设计。也就是说，Koa的原理和内部结构很像Express，但是语法和内部结构进行了升级。

一个Koa应用就是一个对象，包含了一个middleware数组，这个数组由一组Generator函数组成。这些函数负责对HTTP请求进行各种加工，比如生成缓存、指定代理、请求重定向等等。

``` javascript
var koa = require('koa');
var app = koa();

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

**app.use方法用于向middleware数组添加Generator函数**

当然最新的koa2已经支持直接添加es7的asy await语法糖

``` javascript
app.use(async (ctx, next) => {
  const start = new Date();
  await next();
  const ms = new Date() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
```
以上方法更为推荐其实，接受了asy函数，函数有两个参数，一个是context一个是next。


### middleware
Koa的中间件很像Express的中间件，也是对HTTP请求进行处理的函数，但是必须是一个Generator函数(**koa2已经可以用ES7的asy函数**)。而且，Koa的中间件是一个级联式（Cascading）的结构，也就是说，属于是层层调用，第一个中间件调用第二个中间件，第二个调用第三个，以此类推。上游的中间件必须等到下游的中间件返回结果，才会继续执行，这点很像递归。

中间件通过当前应用的use方法注册。

``` javascript
app.use(function* (next){
  var start = new Date; // （1）
  yield next;  // （2）
  var ms = new Date - start; // （3）
  console.log('%s %s - %s', this.method, this.url, ms); // （4）
});
```

Generator函数内部使用yield命令，将程序的执行权转交给下一个中间件，即yield next，要等到下一个中间件返回结果，才会继续往下执行。上面代码中，Generator函数体内部，第一行赋值语句首先执行，开始计时，第二行yield语句将执行权交给下一个中间件，当前中间件就暂停执行。等到后面的中间件全部执行完成，执行权就回到原来暂停的地方，继续往下执行，这时才会执行第三行，计算这个过程一共花了多少时间，第四行将这个时间打印出来。

**只要有一个中间件缺少yield next语句，后面的中间件都不会执行，这一点要引起注意。**

由于Koa要求中间件唯一的参数就是next，导致如果要传入其他参数，必须另外写一个返回Generator函数的函数。

### Context上下文
Koa Context 将 node 的 request 和 response 对象封装在一个单独的对象里面，其为编写 web 应用和 API 提供了很多有用的方法。

context 在每个 request 请求中被创建，在中间件中作为接收器(receiver)来引用，或者通过 this 标识符来引用：

``` javascript
app.use(function *(){
  this; // is the Context
  this.request; // is a koa Request
  this.response; // is a koa Response
});
```