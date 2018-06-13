## Vue实例与基本语法
都来源于vue文档，详细内容可以翻阅文档，这里记录浓缩版的笔记。

### vue实例
每个 Vue 应用都是通过用 Vue 函数创建一个新的 Vue 实例开始的

``` js
var vm = new Vue({
  // 选项
})
```

最常用的选项就是el和data，el表明这个vue实例绑定在哪个id的dom上，data表明这个vue实例(组件)持有的数据(局部)

**实例选项在本笔记的后面有续讲**


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

### 模板语法
可以普通的用文本插值

``` html
<span>Message: {{ msg }}</span>
```

也可以用js表达式：

``` js
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}
```

### 指令
vue中有带v-前缀的指令，可用在html标签(模板中)上。

比较常用的指令有：

#### v-if
值是一个bool,根据其真假来移除对应的html标签。

``` js
<p v-if="seen">现在你看到我了</p>
```

如果seen变量值为真，真p标签存在，否则将会被移除(看不到)

有if当然就有else

``` html
<h1 v-if="ok">Yes</h1>
<h1 v-else>No</h1>
```

v-else 则是标签分支

#### v-bind
响应式的更新html特性，所谓响应式，就是数据变更时，对应的dom也会产生变化。

``` html
<a v-bind:href="url">...</a>
```

v-bind绑定的也往往是html标签的一个属性，比如上面绑定的就是a标签的href属性，url是一个js变量，url变量一变，a标签对应的href也会变得(响应式)

**v-bind 的缩写是一个冒号：**

``` html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

显然，bind可以用来动态别更class和style，也就是动态变更css(因为css也是html的一个属性)，具体语法细节可以去再看看文档

##### key属性(重)
在vue场景中，经常需要我们用bind指令去绑定html的key属性，这个属性，基本上是方便我们做dom操作的。

经常有需要用bind指令去设定key，所以要着重注意一下

#### v-on
一个用于事件监听的指令，绑在dom元素上，监听一些事件，然后启动对应的函数。最典型的就是click

``` html
<a v-on:click="doSomething">...</a>
```

**v-on的缩写是一个@符号**

``` html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

#### v-for
很直观，用于列表渲染。

``` js
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>


var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

##### 注意点
**列表渲染时，直接通过索引改变值，不是响应式的。**

``` js
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```

想要改变，我们必须使用`Vue.set`方法

```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
```

还有其他诸如此类数据不相应的问题，请查阅文档相关章节处理。

#### v-model
可以用 v-model 指令在表单 <input\> 及 <textarea\> 元素上创建 **双向数据绑定**

除了input，其他表单控件诸如checkbox，radio都可以，细节可见文档。

不过也要注意：**v-model 会忽略所有表单元素的 value、checked、selected 特性的初始值而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 data 选项中声明初始值。**

``` html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

### 计算属性(Vue实例选项续)
vue实例中的computed属性，用来整合一些复杂的计算逻辑。

``` js
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>

var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

一旦data中的message发生改变，计算属性reversedMessage也会动态改变。

**不同的是计算属性是基于它们的依赖进行缓存的。计算属性只有在它的相关依赖发生改变时才会重新求值。这就意味着只要 message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不会再次执行函数。**

### 侦听器(Vue实例选项续)
watch也是vue实例中的一个选项，用来侦听数据变化。**watch 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。**

下面耐心看一个偏长一点的例子

``` html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>

<!-- 因为 AJAX 库和通用工具的生态已经相当丰富，Vue 核心代码没有重复 -->
<!-- 提供这些功能以保持精简。这也可以让你自由选择自己更熟悉的工具。 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created: function () {
    // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
    // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
    // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
    // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
    // 请参考：https://lodash.com/docs#debounce
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        .catch(function (error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>
```

input输入框的值绑定的是一个question字符串变量。question一旦发生变化，watch就能侦测到，就去用lodash调用getAnswer。一个异步操作
