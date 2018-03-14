## JavaScript 中对变量和函数声明的“提前（hoist）

http://www.bootcss.com/article/variable-and-function-hoisting-in-javascript/

### 变量声明“被提前”
JavaScript 的语法和 C 、Java、C# 类似，统称为 C 类语法。有过 C 或 Java 编程经验的同学应该对“先声明、后使用”的规则很熟悉，如果使用未经声明的变量或函数，在编译阶段就会报错。然而，JavaScript 却能够在变量和函数被声明之前使用它们。

``` javascript
(function() {
  // Outputs: undefined
  console.log(declaredLater);

  var declaredLater = "Now it's defined!";

  // Outputs: "Now it's defined!"
  console.log(declaredLater);
})();
```

首先，上面这段代码是正确的，没有任何问题。但是，为什么不报错了？declaredLater 变量是在调用语句后面定义的啊？为什么居然输出的是 undefined？


这其实是 JavaScript 解析器搞的鬼，解析器将当前作用域内声明的所有变量和函数都会放到作用域的开始处，但是，只有变量的声明被提前到作用域的开始处了，而赋值操作被保留在原处。上述代码对于解析器来说其实是如下这个样子滴：

``` javascript
(function() {
  var declaredLater; //声明被提前到作用域开始处了！

  // Outputs: undefined
  console.log(declaredLater);

  declaredLater = "Now it's defined!"; //赋值操作还在原地!

  // Outputs: "Now it's defined!"
  console.log(declaredLater);
})();
```

由于 JavaScript 具有这样的“怪癖”，所以你会看到很多编码指南建议大家将变量声明放在作用域的最上方，这样就能时刻提醒自己注意了。

### 函数声明“被提前”
前边说的是变量，接下来我们说说函数。

函数的“被提前”还要分两种情况，一种是函数声明，第二种是函数作为值赋值给变量。

JavaScript 解释器允许你在函数声明之前使用，也就是说，函数声明并不仅仅是函数名“被提前”了，整个函数的定义也“被提前”了！所以上述代码能够正确执行。
