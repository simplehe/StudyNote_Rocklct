## Vue实例与基本语法
都来源于vue文档，详细内容可以翻阅文档，这里记录浓缩版的笔记。

### vue实例
每个 Vue 应用都是通过用 Vue 函数创建一个新的 Vue 实例开始的

``` js
var vm = new Vue({
  // 选项
})
```

#### 响应式数据
vue实例中，key值为data所代表的对象，也被绑定到自身了。

``` js
// 我们的数据对象
var data = { a: 1 }

// 该对象被加入到一个 Vue 实例中
var vm = new Vue({
  data: data
})

// 获得这个实例上的属性
// 返回源数据中对应的字段
vm.a == data.a // => true

// 设置属性也会影响到原始数据
vm.a = 2
data.a // => 2

// ……反之亦然
data.a = 3
vm.a // => 3
```

数据改变，vue相应的页面渲染也会改变。

#### 生命周期
vue实例当然有生命周期，不同的生命周期阶段可以执行对应函数。这些函数叫 **生命周期钩子函数**

``` js
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```

![](image/vue2.png)
