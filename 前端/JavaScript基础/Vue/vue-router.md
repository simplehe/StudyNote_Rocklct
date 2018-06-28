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


需要注意的是，父组件的template中要有对应的router-view标签用以显示子组件。这样当跳转到子路由的时候，会先显示父组件，再去渲染父组件里的子组件送到router-view那里去。

#### 命名视图
有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 sidebar (侧导航) 和 main (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 router-view 没有设置名字，那么默认为 default。

``` html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

几个router-view，然后其中两个被赋予了name属性a和b。

最后传递ab组件。

``` js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

这样做的好处是，如果你不传递进ab组件，那么对应的ab router-view就会被忽略而不会被渲染。构建单页面的特殊页面时，比较好用。比如我管理后台有典型的侧边栏sidebar，footer，header。但是我也有登录页面，这时不写死`<sidebar>` `<footer>`,而是通过命名视图，当要跳转到登录页面时，我就不传递sidebar组件，这样特殊的登录页就可以直接被展示。

### 编程式导航
除了使用router-link来点击跳转，我们还可以用代码实现跳转。

用内建的`$router`对象里的push方法。

``` js
const userId = 123
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
```

### 路由模式
Vue默认采用Hash模式。这也就是所谓的前端路由

所谓的hash，即地址栏 URL 中的 # 符号（此 hash 不是密码学里的散列运算）。比如这个 URL：http://www.abc.com/#/hello，hash 的值为 #/hello。它的特点在于：hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中，对后端完全没有影响，因此 **改变 hash 不会重新加载页面**。

还有一种是history模式， 利用了 HTML5 History Interface 中新增的 pushState() 和 replaceState() 方法。（需要特定浏览器支持）
这两个方法应用于浏览器的历史记录栈，在当前已有的 back、forward、go 的基础之上，它们提供了对历史记录进行修改的功能。只是当它们执行修改时，虽然改变了当前的 URL，但浏览器不会立即向后端发送请求。

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

### 导航守卫
vue-router允许在路由导航前后触发某些事件。

比如我们可以使用 router.beforeEach 注册一个全局前置守卫

``` js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时 **导航在所有守卫 resolve 完之前一直处于 等待中**。

每个守卫方法接收三个参数：

 - to: Route: 即将要进入的目标 路由对象

 - from: Route: 当前导航正要离开的路由

 - next: Function: 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。

其中next有几种调用方式需要注意：


- next(): 进行管道中的下一个钩子。**如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。**

- next(false): 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。

- next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 next 传递任意位置对象，且允许设置诸如 replace: true、name: 'home' 之类的选项以及任何用在 router-link 的 to prop 或 router.push 中的选项。

- next(error): (2.4.0+) 如果传入 next 的参数是一个 Error 实例，则导航会被终止且该错误会被传递给 router.onError() 注册过的回调。

### 路由元信息
定义路由时，可以配置meta标签

我们称呼 routes 配置中的每个路由对象为 **路由记录**。路由记录可以是嵌套的，因此，当一个路由匹配成功后，他可能匹配多个路由记录

``` js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      children: [
        {
          path: 'bar',
          component: Bar,
          // a meta field
          meta: { requiresAuth: true }
        }
      ]
    }
  ]
})
```

根据上面的路由配置，/foo/bar 这个 URL 将会匹配父路由记录以及子路由记录。也就是说此事URL匹配到了两条路由。

根据这个原理，我们就可以在路由中放一点元信息meta，然后结合守卫导航，我们来做一些check，好比权限校验，如下：

``` js
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
```

用一个全局的路由导航，在跳转之前，match一下这个路由的权限，看看meta里写着要不要auth认证。如果需要，且当前是未认证状态，则跳到login页面。否则都正常next
