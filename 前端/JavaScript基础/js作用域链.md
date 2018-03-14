## js作用域链
JS中的作用域，大家都知道的，分为全局作用域和局部作用域，没有块级作用域

**除了Global Scope，只有function可以创建新作用域（Function Scope）。**不过这已经是老黄历了，ES6引入了Block Scope。


在js中，函数也是对象，在函数这个对象内，有一个[[scope]]属性，这个属性的指就是函数被创建的作用域中对象的集合，其实即是函数(闭包)的上下文。直观一点，就是假如定义下面这个函数：


``` javascript
function add(num1,num2) {
    var sum = num1 + num2;
    return sum;
}
```

在一个函数被定义的时候, 会将它定义时刻的scope chain链接到这个函数对象的[[scope]]属性，注意这里是将整一条链都放进去了。

上面的代码，定义这个函数的时候就会在函数内的[[scope]]属性里，传递一个global对象，因为这个add函数属于gloabl环境下，传入的是这个函数定义的环境所在的环境。

![](image/scope_chain1.jpg)

这就完成了函数声明的初始化，传入全局对象。

这只是声明函数被创建未执行的时候。

但是如若每个函数被创建出来执行，如
``` javascript
var total = add(5,10);
```

执行此函数时会创建一个称为“运行期上下文(execution context)”的内部对象，运行期上下文定义了函数执行时的环境。每个运行期上下文都有自己的作用域链，用于标识符解析，当运行期上下文被创建时，而它的作用域链初始化为当前运行函数的[[Scope]]所包含的对象。

这些值按照它们出现在函数中的顺序被复制到运行期上下文的作用域链中。它们共同组成了一个新的对象，叫“活动对象(activation object)”，该对象包含了函数的所有局部变量、命名参数、参数集合以及this，然后此对象会被推入作用域链的前端，当运行期上下文被销毁，活动对象也随之销毁。新的作用域链如下图所示
![](image/scope_chain2.jpg)

### 尝试总结
总之就是函数在声明时，会将他定义时刻的scope chain链接到这个函数对象的[[scope]]属性(定义时上一个函数的scope属性)


而函数在创建执行调用时，会生成一个活动对象AO，里面包含有函数的形参值对应，这个活动对象会被添加到[[scope]]中的最前端，方便之后进行变量查表。

一般自己的活动对象会放在作用域链中的第一个位置，上一个位置是上一个调用函数的活动对象，一直到全局对象。如果检索变量，这些对象里都没有，那么undefined。

### 经典例子
``` javascript
function factory() {
     var name = 'laruence';
     var intro = function(){
          alert('I am ' + name);
     }
     return intro;
}

function app(para){
     var name = para;
     var func = factory();
     func();
}

app('eve');
```
app函数和factory函数定义的时候，都会先把global函数给推进他们的scope属性中。

当运行app('eve')的时候，会生成app的活动对象并推送到app scope属性的前端，如下
``` javascript
[[scope chain]] = [
{
     para : 'eve',
     name : undefined,
     func : undefined,
     arguments : []
}, {
     window call object
}
]
```
活动对象包括形参的赋值还有自身内部定义的一些变量。
接着func需要执行factory函数，则factory函数被执行的时候。

``` javascript
[[scope chain]] = [
{
     name : undefined,
     intro : undefined
}, {
     window call object
}
]
```
这时候，factory的活动对象被建立，内部有name和intro属性。然后自己的上一个是global对象。

接下来factory里面定义了一个函数，这里注意这个函数将会在定义的时候被赋予factory此时函数的scope chain。

``` javascript
[[scope chain]] = [
{
     name : 'laruence',
     intor : undefined
}, {
     window call object
}
]
```
此时注意，最前端的是factory的活动对象，window为从chain里继承过来的全局对象。

然后返回这个intro函数给app函数内的func。

func在执行的时候，会生成intro函数的活动对象，但是由于intro函数没有参数，里面也没有定义变量，所以这个活动对象几近是空的，那么实际上就是
``` javascript
[[scope chain]] = [
{
     intro call object
}, {
     name : 'laruence',
     intor : undefined
}, {
     window call object
}
]
```
所以运行结果是
I am laruence

注意，每个函数在被调用时，其活动对象都会自动取得两个特殊变量：**this和arguments**。内部函数在搜索这个变量时，**只会搜索到其活动对象为止**，因此永远不可能直接访问外部函数中的这两个变量。this一定是创建活动对象的时候动态获取的。

还有匿名函数的this一般都是指向全局。
