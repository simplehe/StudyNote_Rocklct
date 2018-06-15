## 权限管理
参考：<a>https://segmentfault.com/a/1190000009506097</a>

做后台系统，最绕不开的就是权限管理。对于vue来说，不同的权限对应着不同的路由，侧边栏也需要根据不同的权限，异步生成。

在vue-admin-element中，权限验证思路如下:

 - 登录：客户端发送密码向服务端验证用户名和密码，验证如果通过，服务端会返回一个token，这个token会被客户端存储到cookie中，这样用户刷新页面也能记住登录状态，前端根据这个token还会向服务端拉取一个user_info的接口来获取用户的详细信息(如用户权限,用户名等等)

 - 权限验证：通过token获取用户对应的 role，**动态根据用户的 role 算出其对应有权限的路由**，通过 router.addRoutes 动态挂载这些路由。

上述所有的数据和操作都是通过vuex全局管理控制的。(补充说明：刷新页面后 vuex的内容也会丢失，所以需要重复上述的那些操作)

所以现在先要来捋一捋，权限登录的流程

### 登录流程

#### 按下按钮

首先用户点击登录按钮，click 事件触发登录操作：

``` js
this.$store.dispatch('LoginByUsername', this.loginForm).then(() => {
  this.$router.push({ path: '/' }); //登录成功之后重定向到首页
}).catch(err => {
  this.$message.error(err); //登录失败提示错误
});
```

dispatch方法调用了store的一个action，然后传递进入一个loginForm对象作为表单参数。

具体的action：

``` js
LoginByUsername({ commit }, userInfo) {
  const username = userInfo.username.trim()
  return new Promise((resolve, reject) => {
    loginByUsername(username, userInfo.password).then(response => {
      const data = response.data
      Cookies.set('Token', response.data.token) //登录成功后将token存储在cookie之中
      commit('SET_TOKEN', data.token)
      resolve()
    }).catch(error => {
      reject(error)
    });
  });
}
```

action的逻辑也很直接，接收到userInfo这个param之后，获取用户名密码。然后返回一个promise，注意到按下按钮时调用dispatch之后后面有then，可见这个异步任务当然是一个promise。

loginByUsername就是一个向服务器拿数据的过程了，拿到response之后，里面会返回一个token(服务器给回来的),接着我们会把它放到Cookies中，并且调用commit方法，把这个token字符串保存到store的state里面。

##### 一个option cookie有效期方案
为了保证安全性，可以让所有token有效期(Expires/Max-Age)都是Session，就是当浏览器关闭了就丢失了。重新打开游览器都需要重新登录验证，后端也会在每周固定一个时间点重新刷新token，让后台用户全部重新登录一次，确保后台用户不会因为电脑遗失或者其它原因被人随意使用账号。

#### 获取用户信息
捋一下上一步流程，在dispatch异步任务调用完，token被设置好(设置到state中了)以后。router会跳转到根页面/.

接下来就涉及到生命周期里的钩子函数了，全局router.beforeEach里，会先拦截路由，判断是否获得了token，获得token之后就可以去获取用户信息了。

``` js
//router.beforeEach
if (store.getters.roles.length === 0) { // 判断当前用户是否已拉取完user_info信息
  store.dispatch('GetInfo').then(res => { // 拉取user_info
    const roles = res.data.role;
    next();//resolve 钩子
  })
}
```

以上过程，放到守卫导航beforeEach里执行。每次进行路由跳转，都要进行权限校验的，判断当前有没有获取到用户数据，如果没有获取到，就dispatch触发getinfo方法。获取参数以后设置组件的role，然后释放钩子。

##### 更多考虑
就如前面所说的，我只在本地存储了一个用户的token，并没有存储别的用户信息（如用户权限，用户名，用户头像等）。有些人会问为什么不把一些其它的用户信息也存一下？主要出于如下的考虑：

假设我把用户权限和用户名也存在了本地，但我这时候用另一台电脑登录修改了自己的用户名，之后再用这台存有之前用户信息的电脑登录，它默认会去读取本地 cookie 中的名字，并不会去拉去新的用户信息。

所以现在的策略是：页面会先从 cookie 中查看是否存有 token，没有，就走一遍上一部分的流程重新登录，如果有token,就会把这个 token 返给后端去拉取user_info，保证用户信息是最新的。

当然如果是做了单点登录得功能的话，用户信息存储在本地也是可以的。当你一台电脑登录时，另一台会被提下线，所以总会重新登录获取最新的内容。

### 权限控制流程
之前简述了登录流程。现在来讲一讲权限控制，前端会有一份路由表，它表示了每一个路由可访问的权限。当用户登录之后，通过 token 获取用户的 role ，动态根据用户的 role 算出其对应有权限的路由，再通过router.addRoutes动态挂载路由。

但这些控制都只是页面级的，说白了前端再怎么做权限控制都不是绝对安全的，后端的权限验证是逃不掉的。

不同权限的用户显示不同的侧边栏和限制其所能进入的页面(也做了少许按钮级别的权限控制)，后端则会验证每一个涉及请求的操作，验证其是否有该操作的权限，每一个后台的请求不管是 get 还是 post 都会让前端在请求 header里面携带用户的 token，后端会根据该 token 来验证用户是否有权限执行该操作。若没有权限则抛出一个对应的状态码，前端检测到该状态码，做出相对应的操作。


#### addRoutes
好在vue2.2.0以后新增了router.addRoutes。有了这个我们就可相对方便的做权限控制了。

具体实现：

1. 创建vue实例的时候将vue-router挂载，但这个时候vue-router挂载一些登录或者不用权限的公用的页面。

2. 当用户登录后，获取role，将role和路由表每个页面的需要的权限作比较，生成最终用户可访问的路由表。
调用router.addRoutes(store.getters.addRouters)添加用户可访问的路由。

3. **使用vuex管理路由表，根据vuex中可访问的路由渲染侧边栏组件。**

``` js
//实例化vue的时候只挂载constantRouter
export default new Router({
  routes: constantRouterMap
});

//异步挂载的路由
//动态需要根据权限加载的路由表
export const asyncRouterMap = [
  {
    path: '/permission',
    component: Layout,
    name: '权限测试',
    meta: { role: ['admin','super_editor'] }, //页面需要的权限
    children: [
    {
      path: 'index',
      component: Permission,
      name: '权限测试页',
      meta: { role: ['admin','super_editor'] }  //页面需要的权限
    }]
  },
  { path: '*', redirect: '/404', hidden: true }
];
```

用vue-router的元信息和守卫导航来实现。这里有一个需要非常注意的地方就是 404 页面一定要最后加载

``` js
// main.js
router.beforeEach((to, from, next) => {
  if (store.getters.token) { // 判断是否有token
    if (to.path === '/login') {
      next({ path: '/' });
    } else {
      if (store.getters.roles.length === 0) { // 判断当前用户是否已拉取完user_info信息
        store.dispatch('GetInfo').then(res => { // 拉取info
          const roles = res.data.role;
          store.dispatch('GenerateRoutes', { roles }).then(() => { // 生成可访问的路由表
            router.addRoutes(store.getters.addRouters) // 动态添加可访问路由表
            next({ ...to, replace: true }) // hack方法 确保addRoutes已完成 ,set the replace: true so the navigation will not leave a history record
          })
        }).catch(err => {
          console.log(err);
        });
      } else {
        next() //当有用户权限的时候，说明所有可访问路由已生成 如访问没权限的全面会自动进入404页面
      }
    }
  } else {
    if (whiteList.indexOf(to.path) !== -1) { // 在免登录白名单，直接进入
      next();
    } else {
      next('/login'); // 否则全部重定向到登录页
    }
  }
});
```

这里还有一个小hack的地方，就是router.addRoutes之后的next()可能会失效，因为可能next()的时候路由并没有完全add完成,这样我们就可以简单的通过next(to)巧妙的避开之前的那个问题了。这行代码重新进入router.beforeEach这个钩子，这时候再通过next()来释放钩子，就能确保所有的路由都已经挂在完成了。

#### store部分
接下来也看看store部分的代码

``` js
// store/permission.js
import { asyncRouterMap, constantRouterMap } from 'src/router';

function hasPermission(roles, route) {
  if (route.meta && route.meta.role) {
    return roles.some(role => route.meta.role.indexOf(role) >= 0)
  } else {
    return true
  }
}

const permission = {
  state: {
    routers: constantRouterMap,
    addRouters: []
  },
  mutations: {
    SET_ROUTERS: (state, routers) => {
      state.addRouters = routers;
      state.routers = constantRouterMap.concat(routers);
    }
  },
  actions: {
    GenerateRoutes({ commit }, data) {
      return new Promise(resolve => {
        const { roles } = data;
        const accessedRouters = asyncRouterMap.filter(v => {
          if (roles.indexOf('admin') >= 0) return true;
          if (hasPermission(roles, v)) {
            if (v.children && v.children.length > 0) {
              v.children = v.children.filter(child => {
                if (hasPermission(roles, child)) {
                  return child
                }
                return false;
              });
              return v
            } else {
              return v
            }
          }
          return false;
        });
        commit('SET_ROUTERS', accessedRouters);
        resolve();
      })
    }
  }
};

export default permission;
```

这里的代码说白了就是干了一件事，通过用户的权限和之前在router.js里面asyncRouterMap的每一个页面所需要的权限做匹配，最后返回一个该用户能够访问路由有哪些。

zwlj:vuex来保存可访问的路由。

#### 侧边栏生成

无非就是根据vuex的数据，来生成侧边栏。详细可以参见具体代码和文档。
