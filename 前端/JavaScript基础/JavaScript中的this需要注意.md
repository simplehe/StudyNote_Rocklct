## js中的this

很简单的一句话理解

 - 当一个函数被调用时，拥有它的object会作为this传入。在global下，就是window or global，其他时候就是相应的object。

一种误区需要认清

``` javascript
var bfunc = function(){return this};
var b = {name:"b",fun:bfunc);

b.fun();
//=> return global object
```

上述代码返回的是一个global object，并不是b。因为bfunc是在外部定义的函数，被传进来的是一个引用，当这个函数被调用的时候，this指向的还是拥有bfunc的对象，即是全局对象。 
