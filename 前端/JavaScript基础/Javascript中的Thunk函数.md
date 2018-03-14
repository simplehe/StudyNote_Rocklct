## JavaScript语言中的Thunk函数

### 参数求值策略
Thunk函数早在上个世纪60年代就诞生了。

那时，编程语言刚刚起步，计算机学家还在研究，编译器怎么写比较好。一个争论的焦点是"求值策略"，即函数的参数到底应该何时求值。

    var x = 1;
    
    function f(m){
      return m * 2;
    }
    
    f(x + 5)
上面代码先定义函数f，然后向它传入表达式x + 5。请问，这个表达式应该何时求值？

一种意见是"传值调用"（call by value），即在进入函数体之前，就计算x + 5的值（等于6），再将这个值传入函数f 。C语言就采用这种策略。

    f(x + 5)
    // 传值调用时，等同于
    f(6)
另一种意见是"传名调用"（call by name），即直接将表达式x + 5传入函数体，只在用到它的时候求值。Haskell语言采用这种策略。

    f(x + 5)
    // 传名调用时，等同于
    (x + 5) * 2
传值调用和传名调用，哪一种比较好？回答是各有利弊。传值调用比较简单，但是对参数求值的时候，实际上还没用到这个参数，有可能造成性能损失。
    
    function f(a, b){
      return b;
    }
    
    f(3 * x * x - 2 * x - 1, x);
上面代码中，函数f的第一个参数是一个复杂的表达式，但是函数体内根本没用到。对这个参数求值，实际上是不必要的。因此，有一些计算机学家倾向于"传名调用"，即只在执行时求值。

### Thunk函数的意义
编译器的"传名调用"实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数。

    function f(m){
      return m * 2;
    }
    
    f(x + 5);
    
    // 等同于
    
    var thunk = function () {
      return x + 5;
    };
    
    function f(thunk){
      return thunk() * 2;
    }
上面代码中，函数f的参数x + 5被一个函数替换了。凡是用到原参数的地方，对Thunk函数求值即可。

这就是Thunk函数的定义，**它是"传名调用"的一种实现策略**，用来替换某个表达式。

### JavaScript语言的Thunk函数
JavaScript语言是传值调用，它的Thunk函数含义有所不同。在JavaScript语言中，Thunk函数替换的不是表达式，而是多参数函数，将其替换成单参数的版本，且只接受回调函数作为参数。

```
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
var Thunk = function (fileName){
  return function (callback){
return fs.readFile(fileName, callback);
  };
};

var readFileThunk = Thunk(fileName);
readFileThunk(callback);
```

上面代码中，fs模块的readFile方法是一个多参数函数，两个参数分别为文件名和回调函数。经过转换器处理，它变成了一个单参数函数，只接受回调函数作为参数。这个单参数版本，就叫做Thunk函数。

**(个人理解)，总之，就是把一个参数里有回调函数的多参数函数，转换为一个作用相同，但是只接受一个回调函数参数的函数。**


任何函数，只要参数有回调函数，就能写成Thunk函数的形式。下面是一个简单的Thunk函数转换器,将一个函数转换为Thunk。

```
// ES5版本
var Thunk = function(fn){
  return function (){
    var args = Array.prototype.slice.call(arguments);
    return function (callback){
      args.push(callback);
      return fn.apply(this, args);
    }
  };
};

// ES6版本
var Thunk = function(fn) {
  return function (...args) {
    return function (callback) {
      return fn.call(this, ...args, callback);
    }
  };
};
```

使用上面的转换器，生成fs.readFile的Thunk函数。

```
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);
```

**(个人理解)，这里就是先把其余的参数传给Thunk，然后则会给你返回一个只需传入回调函数的函数，思想有点像逆柯丽化**


下面是另一个完整的例子。

```
function f(a, cb) {
  cb(a);
}
let ft = Thunk(f);

let log = console.log.bind(console);
ft(1)(log) // 1

```

### Thunk函数用于Generator函数的自动流程管理
这是Thunk函数其中一个重要的作用。

前面已经说到Thunk函数其实就是一个只接受callback函数的函数，所有参数中带有回调函数的函数，最终也都可以被转换成一个Thunk函数。

那么这样就可以通过Thunk来实现Generator的自动流程管理了

``` javascript
var fs = require('fs');
var thunkify = require('thunkify');
var readFile = thunkify(fs.readFile);

var gen = function* (){
  var r1 = yield readFile('/etc/fstab');
  console.log(r1.toString());
  var r2 = yield readFile('/etc/shells');
  console.log(r2.toString());
};
```

先看例子，我们知道Generator函数返回一个指针，每一次执行next，返回的指针里有两个属性value和done。value便是yield后面的内容，而上述代码readFile正好可以说是一个Thunk函数，因为他要接受一个回调函数来进行调用。

所以常规的流程控制可以写为
``` javascript
var g = gen();

var r1 = g.next();
r1.value(function(err, data){
  if (err) throw err;
  var r2 = g.next(data);
  r2.value(function(err, data){
    if (err) throw err;
    g.next(data);
  });
});
```

可以看到每次取得value以后都调用同一个回调函数来执行next。那么我们就可以写得自动化了。

``` javascript
function run(fn) {
  var gen = fn();

  function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }

  next();
}

run(gen);
```

上面代码的 run 函数，就是一个 Generator 函数的自动执行器。内部的 next 函数就是 Thunk 的回调函数。 next 函数先将指针移到 Generator 函数的下一步（gen.next 方法），然后判断 Generator 函数是否结束（result.done 属性），如果没结束，就将 next 函数再传入 Thunk 函数（result.value 属性），否则就直接退出。
有了这个执行器，执行 Generator 函数方便多了。不管有多少个异步操作，直接传入 run 函数即可。**当然，前提是每一个异步操作，都要是 Thunk 函数，也就是说，跟在 yield 命令后面的必须是 Thunk 函数。**

而且通过上图，result=gen.next(data)不难看出，每次promise函数执行得到的resolve值，都会再次传递会gen函数中。**简而言之便是，yield和await表达式的返回值，都是后面所跟promise对象完成后的resolve值。**
