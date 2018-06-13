## vue-router
Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。

用 Vue.js + Vue Router 创建单页应用，是非常简单的。使用 Vue.js ，我们已经可以通过组合组件来组成应用程序，当你要把 Vue Router 添加进来，我们需要做的是，将组件 (components) 映射到路由 (routes)，然后告诉 Vue Router 在哪里渲染它们。

可以查阅vue router文档来查看相应的用法。

vue-router最直接到就是将一个组件绑定一个路由，比如下面例子

``` js
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

router-link这个标签就可以理解为一个特殊的a标签，他是一个带路由的组件，点击这个组件，就会发生跳转。那么跳转之后会发生什么事呢？这就跟router-view有关了，点击路由以后，相应的router-view将会展示相应的组件

``` js
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义 (路由) 组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点再讨论嵌套路由。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // (缩写) 相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

上面是使用的基本流程。定义一个routes对象，把path字符串和component对应起来。最后new一个VueRouter(路由器)对象，然后把route对象传递给它。

### 路由匹配
假如我们有一个组件User展示用户信息，用户id各不相同，但是肯定都是要复用这个组件的，这个时候就要用到路由匹配了

``` js
const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

path里后面加了个:id字段

一个“路径参数”使用冒号 : 标记。**当匹配到一个路由时，参数值会被设置到 this.$route.params，可以在每个组件内使用。**

``` js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

zwlj:注意，用冒号标记的字段，才会成为变量记录到params中。

#### 嵌套路由
有时候我们组件之间会有嵌套的情况。这时候，就需要我们在设置router里的routes属性时，就设定childer

``` js
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

所以在访问/user/:id/profile的时候，就会显示子组件

同时，如果想保证/user/id也能访问到一个组件，那就要设定空的子路由

### 编程式导航
除了使用router-link来点击跳转，我们还可以用代码实现跳转。

用内建的`$router`对象里的push方法。

``` js
const userId = 123
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
```

### 路由懒加载
当打包构建应用时，Javascript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。


首先，可以将异步组件定义为返回一个 Promise 的工厂函数 (该函数返回的 Promise 应该 resolve 组件本身)：

``` js
const Foo = () => Promise.resolve({ /* 组件定义对象 */ })
```

第二，在 Webpack 2 中，我们可以使用动态 import语法来定义代码分块点 (split point)：

``` js
import('./Foo.vue') // 返回 Promise
```

zwlj:注意这里用的是import函数。
