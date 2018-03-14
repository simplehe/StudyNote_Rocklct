## co函数库
<a>http://www.ruanyifeng.com/blog/2015/05/co.html</a>

co 函数库是著名程序员 TJ Holowaychuk 于2013年6月发布的一个小工具，用于 Generator 函数的自动执行。

比如，有一个 Generator 函数，用于依次读取两个文件。

``` javascript
var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

co 函数库可以让你不用编写 Generator 函数的执行器。

``` javascript
var co = require('co');
co(gen);
```

上面代码中，Generator 函数只要传入 co 函数，就会自动执行。
co 函数返回一个 Promise 对象，因此可以用 then 方法添加回调函数。

``` javascript
co(gen).then(function (){
  console.log('Generator 函数执行完成');
})
```

### co的原理
为什么 co 可以自动执行 Generator 函数？
Generator 函数就是一个异步操作的容器。它的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。

- 回调函数。将异步操作包装成 Thunk 函数，在回调函数里面交回执行权。
- Promise 对象。将异步操作包装成 Promise 对象，用 then 方法交回执行权。

co 函数库其实就是将两种自动执行器（**Thunk 函数和 Promise 对象**），包装成一个库。使用 co 的前提条件是，Generator 函数的 yield 命令后面，只能是 Thunk 函数或 Promise 对象。

***个人理解:其实用promise或者Thunk函数都是大同小异，因为一个promise对象也是要接受一个回调函数来执行，从这个角度来看，这简直就是Thunk函数嘛。***

详见Thunk函数笔记或者
<a>http://www.ruanyifeng.com/blog/2015/05/co.html</a>

### 总结
就是个Generator的自动执行next工具。