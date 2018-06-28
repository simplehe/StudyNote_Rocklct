## vue组件
浓缩笔记中，组件相关的知识点。

组件是 **可复用的 Vue 实例**，且带有一个名字

zwlj：所以组件无非也是一个vue实例(见vue实例笔记)，不过是可复用的。

vue组件示例：

``` js
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

但是还是有需要注意,组件跟大Vue实例有一些不同的地方

### data属性是函数
作为组件的vue，选项中的data属性，必须是一个函数：

这样，每个实例才会维护一份自己的独立data拷贝

``` js
data: function () {
  return {
    count: 0
  }
}
```

### props属性设置
可以通过设置props属性，从父组件那里获取一个Object变量。

``` js
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})

new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' },
    ]
  }
})

<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>


```

**注意，一个props被定义以后，就可以通过组件html标签的自定义特性传入** 所以通过设定html属性给到组件，很方便。

### 插槽slot
Vue 实现了一套内容分发的 API，将 `<slot>` 元素作为承载分发内容的出口。

它允许你像这样合成组件:

当我们写父组件的template时，假如我们想调用一个叫navigation-link的组件，我们这样写：

``` html
<navigation-link url="/profile">
  Your Profile
</navigation-link>
```

在navigation-link组件中，我们这样写template

``` html
<a
  v-bind:href="url"
  class="nav-link"
>
  <slot></slot>
</a>
```

在navigation-link中放了一个slot，那么我们在父组件中调用此组件时，内部的内容就会被传递到这个slot当中。当组件渲染的时候，这个 `<slot>` 元素将会被替换为“Your Profile”。

**插槽内可以包含任何模板代码，包括 HTML**

#### 具名插槽
有时候我们可以给slot命名，这样在调用的时候就可以更具体：

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

因此父组件在传递内容的时候，就需要用template标签来指定了：

``` html
<base-layout>
  <template slot="header">
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template slot="footer">
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

如此一来，template中的内容就会传递到子组件base-layout里对应的slot。没有标记template的内容默认传递给无名slot。



### emit方法进行事件传递
结合v-on指令，我们可以用vue内建的 **$emit** 方法来向父组件传递一个事件。

``` html
<!--  在子组件里 -->
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>

<!--  在父组件里 -->
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>

```

如上，子组件设定了要emit一个事件。父组件设定v-on指令，捕获这个自定义事件。

### 动态组件
有时候我们不确定某个区块显示哪一个组件(比如一个多tab页面)

这个时候可以依赖vue的<component\>标签来实现，用<component\>标签的is属性来决定。

``` html
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<component v-bind:is="currentTabComponent"></component>
```

这里把is绑定到了一个变量上

``` js
Vue.component('tab-home', {
	template: '<div>Home component</div>'
})
Vue.component('tab-posts', {
	template: '<div>Posts component</div>'
})
Vue.component('tab-archive', {
	template: '<div>Archive component</div>'
})

new Vue({
  el: '#dynamic-component-demo',
  data: {
    currentTab: 'Home',
    tabs: ['Home', 'Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
```

这个is变量是个字符串，动态改变，对应成自己写的组件的name，这样component标签就会动态变更成对应的组件。


### 模块系统
写vue组件时，可以单独把组件写成一个后缀是vue的文件，然后主组件通过import来引入。

### 异步组件
异步组件主要应用于下面一个场景： **在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。**

为了简化，Vue 允许你以一个工厂函数的方式定义(**注册**)你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会被触发，且会把结果缓存起来供未来重渲染。

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // 向 `resolve` 回调传递组件定义
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

这个function，就是你要传入的工厂函数，接受参数有一个resolve,是一个回调函数。
