## call和apply方法
在 js 里，函数其实也是一个对象，那么函数自然也可以拥有它自己的方法，在 js 里，每个函数都有一个公共的 prototype —— Function，而这个原型自带有好几个属性和方法，其中就有这里困惑的 bind、call、apply 方法。

call() 方法和 apply() 方法很类似，它们的存在都是为了改变 this 的绑定

call 方法可以用来代替另一个对象调用一个方法。call 方法可将一个函数的对象上下文从初始的上下文改变为由 thisObj 指定的新对象。

call和apply方法大概仅仅是传参上的不同而已
区分apply,call就一句话,
foo.call(this, arg1,arg2,arg3) == foo.apply(this, arguments)==this.foo(arg1, arg2, arg3)


call的第一个参数即是要指定的上下文。

传入的第一个参数，则替换掉函数里面的this。

**其实可以这么看，即是把当前这个函数绑定为第一个参数里的一个临时属性。**

来看一个例子

``` javascript

function Class1()
{
    this.name = "class1";

    this.showNam = function()
    {
        alert(this.name);
    }
}

function Class2()
{
    this.name = "class2";
}

var c1 = new Class1();
var c2 = new Class2();

c1.showNam.call(c2);


```

这里可以理解为，在c2为上下文(this)的情况下执行了c1方法。也可以理解为，c2借了c1的showNam方法拿来用。所以这里显示的并不是c1的name，而是c2的name class2.正说明c2借了方法，本体是c2。

总结一下其实call的用途主要有两个

1. 通过改变上下文，借用别人的方法临时绑定到自己身上。
2. 可以做到实现继承

### 继承的例子

``` javascript

function Class1()
{
    this.showTxt = function(txt)
    {
        alert(txt);
    }
}

function Class2()
{
    Class1.call(this);
}

var c2 = new Class2();

c2.showTxt("cc");

```

这样 Class2 就继承Class1了，Class1.call(this) 的 意思就是使用 Class1 对象代替this对象，那么 Class2 中不就有Class1 的所有属性和方法了吗，c2 对象就能够直接调用Class1 的方法以及属性了，执行结果就是：alert（“cc”）;
