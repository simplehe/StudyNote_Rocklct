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

Vuex 通过 store 选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（需调用 Vue.use(Vuex)）：

通过在根实例中注册 store 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 this.$store 访问到。让我们更新下 Counter 的实现：

### Getter
getter用于从store中派生状态。

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

zwlj：也就是在vuex帮我们整合了逻辑，不需要我们在多个组件中，再写一次computed。
