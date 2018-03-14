## JavaScript的构造函数

JavaScript 中的构造函数和其它语言中的构造函数是不同的。 通过 new 关键字方式调用的函数都被认为是构造函数。

在构造函数内部 - 也就是被调用的函数内 - this 指向新创建的对象 Object。 这个新创建的对象的 prototype 被指向到构造函数的 prototype。

如果被调用的函数没有显式的 return 表达式，则隐式的会返回 this 对象 - 也就是新创建的对象。

**也就是说，被new 调用函数以后，函数就被认为是一个构造函数，被隐式返回this。**

``` javascript

function Foo() {
    this.bla = 1;
}

Foo.prototype.test = function() {
    console.log(this.bla);
};

var test = new Foo();

```

上面代码把 Foo 作为构造函数调用，并设置新创建对象的 prototype 为 Foo.prototype。

**显式的 return 表达式将会影响返回结果，但仅限于返回的是一个对象。**

``` javascript

function Bar() {
    return 2;
}
new Bar(); // 返回新创建的对象

function Test() {
    this.value = 2;

    return {
        foo: 1
    };
}
new Test(); // 返回的对象

```

new Bar() 返回的是新创建的对象，而不是数字的字面值 2。 因此 new Bar().constructor === Bar，但是如果返回的是数字对象，结果就不同,则是那个数字对象。所以new可以使得构造函数自定义返回的对象。