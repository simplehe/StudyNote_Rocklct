## Typescript基础
TypeScript 是 JavaScript 的一个超集，主要提供了 **类型系统** 和对 **ES6** 的支持，它由 Microsoft 开发，代码开源于 GitHub 上。

### TypeScript优势
总的来说有3点：

 1. TypeScript 增加了代码的可读性和可维护性 (**类型系统是最好的文档**)
 2. Js的超集,支持ES6语法,且兼容第三方库。
 3. 活跃的社区(Angular2就是使用TypeScript编写)

#### 为什么需要静态类型
TypeScript 是 JavaScript 的静态类型版本。为什么需要静态类型？

 - **错误检测**：静态类型化是一种功能，可以在开发人员编写脚本时检测错误。查找并修复错误是当今开发团队的迫切需求。有了这项功能，就会允许开发人员编写更健壮的代码并对其进行维护，**以便使得代码质量更好、更清晰**。

 - **更好维护**：有时为了改进开发项目，需要对代码库进行小的增量更改。这些小小的变化可能会产生严重的、意想不到的后果，因此有必要撤销这些变化。使用TypeScript工具来进行重构更变的容易、快捷。

#### 支持ES6/ES7

除此之外，TypeScript全面支持ES6与ES7特性。ES6/7是JS的新标准，编程语言的进化无疑会大大提高程序的健壮性、可维护性，提升程序员的开发效率。**尤其是面向对象、模块化等特性的语言级别支持，为JS开发大型应用提供了更多便捷**。

### 安装

```
npm install -g typescript
```

以上命令会在全局环境下安装 tsc 命令，安装完成之后，我们就可以在任何地方执行 tsc 命令了。

```
tsc hello.ts
```

我们约定使用 TypeScript 编写的文件以 .ts 为后缀。

### 数据类型
TS数据类型跟JS对齐,分为两种:原始数据类型和对象类型。

原始数据类型包括：布尔值、数值、字符串、null、undefined 以及 ES6 中的新类型 Symbol。

以下给出TS类型定义的示例

``` js
// 编译通过
let isDone: boolean = false;

//编译不通过 index.ts(1,5): error TS2322: Type 'Boolean' is not assignable to type 'boolean'.
let createdByNewBoolean: boolean = new Boolean(1);

//undefined 类型的变量只能被赋值为 undefined，null 类型的变量只能被赋值为 null。
let u: undefined = undefined;
let n: null = null;

let fibonacci: number[] = [1, 1, 2, 3, 5];//最简单的方法是使用「类型 + 方括号」来表示数组

```

**undefined 和 null 是所有类型的子类型。也就是说 undefined 类型的变量，可以赋值给 number 类型的变量**

#### 空值
JavaScript 没有空值（Void）的概念，在 TypeScirpt 中，可以用 void 表示没有任何返回值的函数：

``` typescript
function alertName(): void {
    alert('My name is Tom');
}
```

#### 任意值
任意值（Any）用来表示允许赋值为任意类型。

``` js
let myFavoriteNumber: any = 'seven';
myFavoriteNumber = 7;
```

**可以认为，声明一个变量为任意值之后，对它的任何操作，返回的内容的类型都是任意值。**

**变量如果在声明的时候，未指定其类型，那么它会被识别为任意值类型**

``` js
let a;// 没有赋初值，则不会类型推断，类型默认为any
```

#### 类型推断

一开始不指定类型，但是赋了初值，那就会进行类型推断。

``` js
let myFavoriteNumber = 'seven';
myFavoriteNumber = 7;

// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.
```

#### Union类
表示取值可以为多种类型中的一种。

``` js
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

使用联合类的时候要注意，

当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候,此时只能访问所有联合起来类别的共有属性或者方法：

``` js
function getLength(something: string | number): number {
    return something.length;
}

// index.ts(2,22): error TS2339: Property 'length' does not exist on type 'string | number'.
//   Property 'length' does not exist on type 'number'.
```

**但是联合类型如果经过了类型推断，就可以访问非共有方法了**,联合类型的变量在被赋值的时候，会根据类型推论的规则推断出一个类型

``` js
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
console.log(myFavoriteNumber.length); // 5
myFavoriteNumber = 7;
console.log(myFavoriteNumber.length); // 编译时报错
```

#### Interface接口
在面向对象语言中，接口（Interfaces）是一个很重要的概念，它是对行为的抽象，而具体如何行动需要由类（classes）去实现（implements）。

TypeScript 中的接口是一个非常灵活的概念，除了可用于对类的一部分行为进行抽象以外，也常用于对「对象的形状（Shape）」进行描述。

接口一般首字母大写。有的编程语言中会建议接口的名称加上 I 前缀。

``` js
interface Person {
    name: string;
    age: number;
}

let tom: Person = {
    name: 'Tom',
    age: 25
};
```

定义的变量比接口少了一些属性是不允许的,多一些属性也是不允许的，依旧是上面Person接口，下面这样写就不行：

``` js
let tom: Person = {
    name: 'Tom'
};
// index.ts(6,5): error TS2322: Type '{ name: string; }' is not assignable to type 'Person'.
//   Property 'age' is missing in type '{ name: string; }'.
```

有时我们希望不要完全匹配一个形状，那么可以用可选属性：

``` js
interface Person {
    name: string;
    age?: number;
}
```

除此只玩，readonly关键字还能帮我们定义只读属性。

#### 函数
在js中，我们早已知道函数是js的一等公民。ts中，对函数类型也做了约束，函数的类型由输入参数的类型和输出变量的类型来决定：

``` js
function sum(x: number, y: number): number {
    return x + y;
}

let mySum: (x: number, y: number) => number = function (x: number, y: number): number {
    return x + y;
};
```

上面这个`(x: number, y: number) => number`就是函数的类型，代表这个函数接受两个number类参数，返回一个number类值。

同时，函数也有**可选参数**,需要注意的是，可选参数必须接在必需参数后面。除此之外，TypeScript 会将添加了默认值的参数识别为可选参数

``` js
function buildName(firstName: string, lastName?: string) {...}
```

ES6 中，可以使用 ...rest 的方式获取函数中的剩余参数（rest 参数）

``` js
function push(array, ...items) {
    items.forEach(function(item) {
        array.push(item);
    });
}

let a = [];
push(a, 1, 2, 3);
```

Typescript甚至支持类型重载，重载允许一个函数接受不同数量或类型的参数时，作出不同的处理。不过要注意，，TypeScript 会优先从最前面的函数定义开始匹配，所以多个函数定义如果有包含关系，需要优先把精确的定义写在前面。

#### 元组
定义一对值分别为 string 和 number 的元组：

``` js
let xcatliu: [string, number] = ['Xcat Liu', 25];

//当赋值或访问一个已知索引的元素时，会得到正确的类型：
xcatliu[0] = 'Xcat Liu';
xcatliu[1] = 25;

//当赋值给越界的元素时，它类型会被限制为元组中每个类型的联合类型：
xcatliu = ['Xcat Liu', 25, 'http://xcatliu.com/']; //数组的第三项满足联合类型 string | number。

```

#### 枚举
枚举使用 enum 关键字来定义,枚举成员会被赋值为从 0 开始递增的数字，同时也会对枚举值到枚举名进行反向映射：

``` js
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 0); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true

console.log(Days[0] === "Sun"); // true
console.log(Days[1] === "Mon"); // true
console.log(Days[2] === "Tue"); // true
console.log(Days[6] === "Sat"); // true
```

#### 类Class
TypeScript中类的定义基本参考了ES6/7中Class的规范，所以可以去查阅相关章节。

![ECMAScript 6 入门 - Class](http://es6.ruanyifeng.com/#docs/class)

TypeScript实现了implement，extends，泛型等特性。跟Java C#等面向对象语言的设计高度相似，熟悉面向对象编程模型的人可以快速上手。

这里稍微回顾一下泛型

``` js
function swap<T, U>(tuple: [T, U]): [U, T] {
    return [tuple[1], tuple[0]];
}

swap([7, 'seven']); // ['seven', 7]
```

就像Java一样，上面定义了一个函数swap，接受两个泛型参数T和U,参数名为tuple类型是个元组，且元组的两个内部类型分别就是T和U。最后返回个元组\[U,T\]

#### 类型断言
类型断言（Type Assertion）可以用来手动指定一个值的类型。

``` js
function getLength(something: string | number): number {
    if ((<string>something).length) {
        return (<string>something).length;
    } else {
        return something.toString().length;
    }
}
```

#### 引用第三方库
TypeScript 2.0 推荐使用 \@types 来管理。

\@types 的使用方式很简单，直接用 npm 安装对应的声明模块即可，以 jQuery 举例：

```
npm install @types/jquery --save-dev
```

### 总结
以上便是TypeScript入门所要了解的基础知识，

### 参考
![TypeScript入门教程](https://legacy.gitbook.com/book/xcatliu/typescript-tutorial)

![知乎-如何评价TypeScript](https://www.zhihu.com/question/21879449/answer/57236840)

![TypeScript VS JavaScript 深度对比](https://juejin.im/entry/5a52ed336fb9a01cbd586f9f)

![ECMAScript 6 入门 - Class](http://es6.ruanyifeng.com/#docs/class)
