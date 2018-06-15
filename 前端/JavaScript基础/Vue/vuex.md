## vuex
Vuex 是一个专为 Vue.js 应用程序开发的 **状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种 **可预测的方式发生变化**。Vuex 也集成到 Vue 的官方调试工具 devtools extension，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

状态自管理应用有以下几个部分：

 - state，驱动应用的数据源
 - view 以声明方式将state映射到视图
 - actions，响应在view上的用户输入导致状态变化

以下是一个表示“单向数据流”理念的极简示意：

![](image/vuex0.png)

**有时候多个视图会依赖于一个状态(数据源),那么这种简易数据流就会被破坏。**

因此我们可以把状态(数据源)抽取出来，以一个全局单例的模式进行管理。这就是vuex的基本思想。

![](image/vuex1.png)

所以vuex就是在管理一个全局的状态

### store的创建
每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state)。

vuex是响应式的，store中的状态改变，那么对应的view也会改变。但是我们要显示地commit mutation(变化)。

我们可以简单创建store

``` js
// 如果在模块化构建系统中，请确保在开头调用了 Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

store.commit('increment')

console.log(store.state.count) // -> 1
```

### vue中获取state
 Vuex 的状态存储是响应式的，从 store 实例中读取状态最简单的方法就是在计算属性中返回某个状态

 ``` js
 // 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

**Vuex 通过 store 选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（需调用 Vue.use(Vuex)）：**

通过在根实例中注册 store 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 this.$store 访问到。让我们更新下 Counter 的实现：

### Getter
getter用于从store中派生状态。

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

zwlj：也就是在vuex帮我们整合了逻辑，不需要我们在多个组件中，再写一次computed。


**Getter 接受 state 作为其第一个参数：**

``` js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

#### mapGetters辅助函数
其实就是一个简单的辅助函数，讲store对象里的getter函数一一映射到当前组件中。这里运用了三个点号

``` js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

辅助函数的目的是，不需要我们在组件里再手写一个computed函数来手动获取store里的这个getter函数。直接一个mapGetter展开，store里的getter方法就自动放到组件中来了。

### Mutation
更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，**并且它会接受 state 作为第一个参数**,可以回看笔记上面出现mutation的例子

还可以向 store.commit 传入额外的参数，即 mutation 的 载荷（payload）

``` js
store.commit({
  type: 'increment',
  amount: 10
})

```

``` js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

**一个重要原则是，Mutation必须是同步函数。**

来看一个错误的例子：

``` js
// 错误例子
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

**实质上任何在回调函数中进行的状态的改变都是不可追踪的。** Mutation都应该要是同步事务。

### Action
Action 类似于 mutation，不同在于：

 - Action 提交的是 mutation，而不是直接变更状态。
 - Action 可以包含任意异步操作。

可以理解为，action其实就是store帮助用户多在mutation上加了一层封装，使其可以异步调用。否则的话，如果想要异步调用，就需要在组件中异步调用的回调里再commit mutation。store帮助我们写了这个封装方便复用

``` js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象(理解成拿到一个store实例就ok了)，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。

**对于组件，Action 通过 store.dispatch 方法触发：**,其实就是类似于在组件中调用commit来执行mutation，而这里action则是通过dispatch来触发。

``` js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

``` js
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

我们甚至可以组合Action，用async和await语法

``` js
// 假设 getData() 和 getOtherData() 返回的是 Promise

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

以上，actionB异步调用等待了actionA的完成。

### Module模块(重)
如果所有的state都集中到一个store里，那应用非常复杂的时候，store就会变得非常臃肿。

Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

``` js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

对于状态的细节，可以看文档。
