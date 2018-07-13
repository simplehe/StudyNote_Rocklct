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

``` typescript
// 编译通过
let isDone: boolean = false;

//编译不通过 index.ts(1,5): error TS2322: Type 'Boolean' is not assignable to type 'boolean'.
let createdByNewBoolean: boolean = new Boolean(1);

```

#### 空值
JavaScript 没有空值（Void）的概念，在 TypeScirpt 中，可以用 void 表示没有任何返回值的函数：

``` typescript
function alertName(): void {
    alert('My name is Tom');
}
```

### 参考
![TypeScript入门教程](https://legacy.gitbook.com/book/xcatliu/typescript-tutorial)

![知乎-如何评价TypeScript](https://www.zhihu.com/question/21879449/answer/57236840)

![TypeScript VS JavaScript 深度对比](https://juejin.im/entry/5a52ed336fb9a01cbd586f9f)
