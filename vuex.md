## VUEX Store

#### state

- 最好通过mutation去改变存储在store里的state，而不是直接去修改store.state.count，这样可以清楚的追踪到状态的变化

- 简单说，需要根据store.state来计算状态值的时候，肯定放在computed里面是吧，如果你要改变它，那肯定是放在methods

- 如果，当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性

  ```js
  // 在单独构建的版本中辅助函数为 Vuex.mapState
  // 没有了$store.state 的字眼
  import { mapState } from 'vuex'

  export default {
    // ...
    computed: mapState({
      // 箭头函数可使代码更简练
      count: state => state.count,

      // 传字符串参数 'count' 等同于 `state => state.count`
      countAlias: 'count', //注意这个

      // 为了能够使用 `this` 获取局部状态，必须使用常规函数
      countPlusLocalState (state) {
        return state.count + this.localCount
      }
    })
  }

  // 当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 mapState 传一个字符串数组。
  computed: mapState([
    // 映射 this.count 为 store.state.count
    'count'
  ])
  ```

- `mapState` 函数返回的是一个对象。我们如何将它与局部计算属性混合使用呢？

  ```js
  computed: {
    localComputed () { /* ... */ },
    // 使用对象展开运算符将此对象混入到外部对象中
    ...mapState({
      // ...
    })
  }
  ```

#### getter

- 就是一个计算属性，store的状态计算方法，如果你不想在每个组件上写计算方法，那最好的肯定在源头设置一个总的计算属性啦。getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

- Getter 接受 state 作为其第一个参数：Getter 也可以接受其他 getter 作为第二个参数：

- 通过方法访问，达到接受参数

  > ```js
  > getters: {
  >   // ...
  >   getTodoById: (state) => (id) => {
  >     return state.todos.find(todo => todo.id === id)
  >   }
  > }
  > store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
  > ```

- `mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性

  ```js
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
  // 更换变量名称
  mapGetters({
    // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
    doneCount: 'doneTodosCount'
  })
  ```

  ​

#### Mutation

- 更改 Vuex 的 store 中的状态的唯一方法是提交 mutation

- 这个选项更像是事件注册：“当触发一个类型为 `increment` 的 mutation 时，调用此函数。”要唤醒一个 mutation handler，你需要以相应的 type 调用 **store.commit** 方法

  > ```js
  > store.commit('increment')
  > ```

- Mutation 需遵守 Vue 的响应规则

- 一条重要的原则就是要记住 **mutation 必须是同步函数**，就是一个更改缓存参数的地方，同步的，你要想异步，就放在action里面去处理逻辑，有点强迫症

  >```js
  >import { mapMutations } from 'vuex'
  >
  >export default {
  >  // ...
  >  methods: {
  >    ...mapMutations([
  >      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
  >
  >      // `mapMutations` 也支持载荷：
  >      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
  >    ]),
  >    ...mapMutations({
  >      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
  >    })
  >  }
  >}
  >```

  ​

#### action

- 为了解决异步操作，提交的是mutation，而不是直接更改状态，说白了，就是有一个统一异步的地方

- 触发方式不一样

  > ```js
  > store.dispatch('increment')
  >
  > // a example
  > // types status management
  > actions: {
  >   checkout ({ commit, state }, products) {
  >     // 把当前购物车的物品备份起来
  >     const savedCartItems = [...state.cart.added]
  >     // 发出结账请求，然后乐观地清空购物车
  >     commit(types.CHECKOUT_REQUEST)
  >     // 购物 API 接受一个成功回调和一个失败回调  购物 API 购物 API
  >     shop.buyProducts(
  >       products,
  >       // 成功操作
  >       () => commit(types.CHECKOUT_SUCCESS),
  >       // 失败操作
  >       () => commit(types.CHECKOUT_FAILURE, savedCartItems)
  >     )
  >   }
  > }
  > ```

- 映射

  > ```js
  > import { mapActions } from 'vuex'
  >
  > export default {
  >   // ...
  >   methods: {
  >     ...mapActions([
  >       'increment', 
  >       // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
  >       // `mapActions` 也支持载荷：
  >       'incrementBy' 
  >       // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
  >     ]),
  >     ...mapActions({
  >       add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
  >     })
  >   }
  > ```

- dispatch机制，一般返回Promise对象，接受回调函数，你需要明白 `store.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise，于是就有了这样

  > ```js
  > actions: {
  >   actionA ({ commit }) {
  >     return new Promise((resolve, reject) => {
  >       setTimeout(() => {
  >         commit('someMutation')
  >         resolve()
  >       }, 1000)
  >     })
  >   }
  > }
  >
  > // 业务
  > store.dispatch('actionA').then(() => {
  >   // ...
  > })
  >
  > // 更是深度异步
  > actions: {
  >   // ...
  >   actionB ({ dispatch, commit }) {
  >     return dispatch('actionA').then(() => {
  >       commit('someOtherMutation')
  >     })
  >   }
  > }
  > ```

#### Module

- API

  > ```JS
  > const moduleA = {
  >   // ...
  >   actions: {
  >     incrementIfOddOnRootSum ({ state, commit, rootState }) {}
  >   },
  >   getters: {
  >     sumWithRootCount (state, getters, rootState) {}
  >   }
  > }
  > ```

- 模块动态注册

  > ```js
  > // 注册模块 `myModule`
  > store.registerModule('myModule', {
  >   // ...
  > })
  > // 注册嵌套模块 `nested/myModule`
  > store.registerModule(['nested', 'myModule'], {
  >   // ...
  > })
  > ```

- `store.unregisterModule(moduleName)` 来动态卸载模块。注意，你不能使用此方法卸载静态模块（即创建 store 时声明的模块）

- 对于大型应用，我们会希望把 Vuex 相关代码分割到模块中。下面是项目结构示例：

  > ```
  > ├── index.html
  > ├── main.js
  > ├── api
  > │   └── ... # 抽取出API请求
  > ├── components
  > │   ├── App.vue
  > │   └── ...
  > └── store
  >     ├── index.js          # 我们组装模块并导出 store 的地方
  >     ├── actions.js        # 根级别的 action
  >     ├── mutations.js      # 根级别的 mutation
  >     └── modules
  >         ├── cart.js       # 购物车模块
  >         └── products.js   # 产品模块
  > ```