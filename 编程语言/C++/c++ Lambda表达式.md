## C++ Lambda表达式

### 可调用对象
在c++中，**可以使用函数调用运算符()的对象是可调用对象**.比如函数，函数指针，还有重载了函数调用运算符的伪函数(可见相关笔记)，以及这里将要介绍的 **Lambda表达式**。

### Lambda表达式
一个Lambda表达式表示一个可调用代码单元，可以理解为是一个 **匿名内联函数**。也就是说当然一个Lambda具有一个返回类型，一个参数列表，一个函数体。

形式如下：

```
[capture list](parameters)->return-type{body}
```

capture list,代表lambda函数中定义的局部变量列表。通常为空，其余parameters为函数形参，return-type为返回类型。

#### Example
Lambda表达式类型难以表示，所以我们一般用auto让编译器自行进行类型推断。**可以理解为，Lambda被创建时，定义了一个新类型和创建了一个该类型实例。**

``` c++
auto f = [](int x, int y) { return x + y; } // 隐式返回类型
auto f2 = [](int& x) { ++x; }   // 没有return语句 -> lambda 函数的返回类型是'void'
auto f3 = []() { ++global_x; }  // 没有参数,仅访问某个全局变量
auto f4 = []{ ++global_x; }     // 与上一个相同,省略了()
```

需要注意的是，**参数列表和返回类型可以省略，单捕获列表和函数体不能省略。**

#### 局部变量捕获
显然Lambda函数可以捕获局部变量，放进自己的局部变量列表。捕获分为两种，值捕获或者是引用捕获。

##### 值捕获

``` c++
void fcn(){
  int v1 = 42;
  auto f = [v1]{return v1};
  v1 = 0;
  auto j = f(); // j=42
}
```

如上代码可知，lambda表达式捕获了局部变量v1。而且要知道 **捕获发生在lambda创建时而非运行时。**

##### 引用捕获

``` c++
void fcn(){
  int v1 = 42;
  auto f = [&v1]{return v1};
  v1 = 0;
  auto j = f(); // j=0
}
```

使用引用，则求得j为0.

#### 可变Lambda mutable
一般情况下，捕获后的值不会再改变。但是我们如果想让他改变，可以用mutable关键字。

``` c++
void fcn(){
  int v1 = 42;
  auto f = [v1] mutable {return ++v1;};
  v1 = 0;
  auto j = f(); // j=43
}
```

### 泛型Lambda
auto 关键字不能够用在参数表里，这是因为这样的写法会与模板的功能产生冲突。但是 Lambda 表达式并不是普通函数，所以 Lambda 表达式并不能够模板化。这就为我们造成了一定程度上的麻烦：参数表不能够泛化，必须明确参数表类型。

幸运的是，这种麻烦只存在于 C++11 中，从 C++14 开始，Lambda 函数的形式参数可以使用 auto 关键字来产生意义上的泛型：


``` c++
void learn_lambda_func_4(){
    auto generic = [](auto x, auto y) {
        return x+y;
    };

    std::cout << "generic(1,2) = " << generic(1, 2) << std::endl;
    std::cout << "generic(1.1,2.2) = " << generic(1.1, 2.2) << std::endl;
}

```
