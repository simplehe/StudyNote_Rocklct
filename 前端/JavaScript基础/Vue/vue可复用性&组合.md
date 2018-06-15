## vue可复用性和组合
在创建component的时候，我们之前讲到都是创建一个vue.component。我们可以通过创建一个普通对象，**vue称其为混入对象mixins**，来创建组件

``` js
// 定义一个混入对象
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 定义一个使用混入对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"
```

本质上，就是把一个对象中的选项“混合”进组件中，所以叫混入对象。

### 过滤器
Vue.js 允许你自定义过滤器，可被用于一些常见的文本格式化。

**过滤器可以用在两个地方：双花括号插值和 v-bind 表达式 (后者从 2.1.0+ 开始支持)。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示**

``` js
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```

可以在一个组件的选项中定义本地的过滤器：

``` js
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}

```
