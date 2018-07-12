## Promise
Promise是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6将其写进了语言标准，统一了用法，原生提供了Promise对象。

所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。

 - Promise对象代表一个异步操作，有三种状态：**Pending（进行中）**、**Resolved（已完成，又称Fulfilled）**和**Rejected（已失败）**。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
 - 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从Pending变为Resolved和从Pending变为Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的


### 基本用法

``` javascript
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由JavaScript引擎提供，不用自己部署。

resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从Pending变为Resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从Pending变为Rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

Promise实例生成以后，可以用then方法分别指定Resolved状态和Reject状态的回调函数。

``` javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});

```

Promise对象的简单例子
``` javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});

```
上面代码中，timeout方法返回一个Promise实例，表示一段时间以后才会发生的结果。过了指定的时间（ms参数）以后，Promise实例的状态变为Resolved，就会触发then方法绑定的回调函数。

Promise新建后就会立即执行。

### .catch和
要注意

Promise.prototype.catch()

*It behaves the same as calling Promise.prototype.then(undefined, onRejected).*

这里说，catch()是then()的语法糖。

意思就是在promise后面接.catch(fn),就是相当于接了一个then(undefined,fn)

跟传统的try/catch代码块不同的是，如果没有使用catch方法指定错误处理的回调函数，Promise对象抛出的错误不会传递到外层代码，即不会有任何反应。

### .then
万分注意，.then返回的，依旧是一个promise。

``` js
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
```

**上面的代码使用then方法，依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。**


采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

``` javascript
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});
```

### Promise.resolve方法
promise.resolve方法用来包装一个新的promise对象，参数比较多样，需要注意，具体参阅手册。

Promise.resolve等价于下面的写法。
```
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

返回一个promise对象，并且resolve的值就是Promise.resolve的参数。

#### 参数是Promise实例
如果参数是Promise实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。

#### 参数是一个带then方法的对象
thenable对象指的是具有then方法的对象，比如下面这个对象。

``` javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
```

Promise.resolve方法会将这个对象转为Promise对象，然后就立即执行thenable对象的then方法。

#### 没有then方法的对象或根本不是对象
如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为Resolved。

``` javascript
var p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```

上面代码生成一个新的Promise对象的实例p。由于字符串Hello不属于异步操作（判断方法是它不是具有then方法的对象），返回Promise实例的状态从一生成就是Resolved，所以回调函数会立即执行。Promise.resolve方法的参数，会同时传给回调函数。

#### 不带参数
Promise.resolve方法允许调用时不带参数，直接返回一个Resolved状态的Promise对象。

所以，如果希望得到一个Promise对象，比较方便的方法就是直接调用Promise.resolve方法。

#### 重点注意

**立即resolve的Promise对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时。**

``` javascript
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
```

上面代码中，setTimeout(fn, 0)在下一轮“事件循环”开始时执行，Promise.resolve()在本轮“事件循环”结束时执行，console.log(’one‘)则是立即执行，因此最先输出。

#### promise.all
当需要多个promise执行完成后才继续执行某个人物，那就要用到promise.then.

Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

``` js
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

上面代码中，promises是包含 6 个 Promise 实例的数组，只有这 6 个实例的状态都变成fulfilled，或者其中有一个变为rejected，才会调用Promise.all方法后面的回调函数。

promise数组里，每一个promise完成后的返回值组成一个数组传递给all后面的回调函数。
