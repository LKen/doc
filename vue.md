## VUE

#### Prop 

- 如果不是用template来编写代码，prop[Object]里存在camelCase的属性写法，在HTML上必须是kebab-case 

  的写法，但是在template就不需要这个限制 


- 有挺多种类型

  ```js
  props: {
    title: String,
    likes: Number,
    isPublished: Boolean,
    commentIds: Array,
    author: Object
  }
  ```

- 在传给组件上的Prop值，如果没有给定值，都意味着 true

- 也可以传达整一个Object类型当作Prop用，而不是具体某一个自定义变量

- 如果一个值，只是初始化的值，之后会被其他地方用到，就应该缓存下来

  ```js
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
  ```

  如果，这个值更是在初始值之上有所变化，应该这样

  ```js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

  记住一点，对象和数组，这都是地址引用的，尽管被用在子组件，但是改变他的值，父组件也会改变，所以避免混淆，切记深度复制后才使用，不要用它来作为改变值的机制

- 一些属性值直接写在components标签上，会当作写在组件的根元素上，问题来了，属性值绝大多部分会被覆盖，但是只有class，style，会合并属性值

- 由于写在组件上的值会覆盖根元素上的值，所以就会有一个属性`inheritAttrs: false`控制是否覆盖，

  但是有人喜欢自己控制每一个属性的展示，所以利用 `$attrs`可以获取组件上赋予的特殊值，

  ```js
  Vue.component('base-input', {
    inheritAttrs: false,
    props: ['label', 'value'],
    template: `
      <label>
        {{ label }}
        <input
          v-bind="$attrs"
          v-bind:value="value"
          v-on:input="$emit('input', $event.target.value)"
        >
      </label>
    `
  })
  ```

  ​

#### 自定义事件

- 跟组件和 prop 不同，事件名不会被用作一个 JavaScript 变量名或属性名，所以就没有理由使用 camelCase 或 PascalCase 了。并且 `v-on` 事件监听器在 DOM 模板中会被自动转换为全小写 (因为 HTML 是大小写不敏感的)，所以 `v-on:myEvent` 将会变成 `v-on:myevent`——导致 `myEvent` 不可能被监听到。

  因此，我们推荐你**始终使用 kebab-case 的事件名**。

- 一个组件上的 `v-model` 默认会利用名为 `value` 的 prop 和名为 `input` 的事件，但是像单选框、复选框等类型的输入控件可能会将 `value` 特性用于[不同的目的](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#Value)，不一定根元素就是 `<input>`，只需要利用 value，以及input事件

- 自定义 v-model

  ```js
  Vue.component('base-checkbox', {
    model: {
      prop: 'checked',
      event: 'change'
    },
    props: {
      checked: Boolean
    },
    template: `
      <input
        type="checkbox"
        v-bind:checked="checked"
        v-on:change="$emit('change', $event.target.checked)"
      >
    `
  })

  <base-checkbox v-model="lovingVue"></base-checkbox>
  ```

  ​

- 有时候，我们习惯上层编程，在组件上赋予绑定事件，但是由于事件往往会作用于根元素，不现实；所以如果可以根据需要将自定义的事件传达给指定的子元素，那就好。所以有了这个 `$listeners` 属性

  ```js
  Vue.component('base-input', {
    inheritAttrs: false,
    props: ['label', 'value'],
    computed: {
      inputListeners: function () {
        var vm = this
        // `Object.assign` 将所有的对象合并为一个新对象
        return Object.assign({},
          // 我们从父级添加所有的监听器
          this.$listeners,
          // 然后我们添加自定义监听器，
          // 或覆写一些监听器的行为
          {
            // 这里确保组件配合 `v-model` 的工作
            input: function (event) {
              vm.$emit('input', event.target.value)
            }
          }
        )
      }
    },
    template: `
      <label>
        {{ label }}
        <input
          v-bind="$attrs"
          v-bind:value="value"
          v-on="inputListeners"
        >
      </label>
    `
  })
  ```

- 如果，你需要数据双向绑定，推荐以 `update:my-prop-name` 的模式触发事件取而代之

  ```html
  <text-document v-bind:title.sync="doc.title"></text-document>
  ```






#### 组件基础

- 当用在组件上时，`v-model` 则会这样

  > 将其 `value` 特性绑定到一个名叫 `value` 的 prop 上
  >
  > 在其 `input` 事件被触发时，将新的值通过自定义的 `input` 事件抛出
  >
  > ```js
  > Vue.component('custom-input', {
  >   props: ['value'],
  >   template: `
  >     <input
  >       v-bind:value="value"
  >       v-on:input="$emit('input', $event.target.value)"
  >     >
  >   `
  > })
  > ```

- 动态组件

  - 通过 Vue 的 `<component>` 元素加一个特殊的 `is` 特性来实现

    ​

#### 表单输入绑定

- v-model 是一个语法糖，有以下这几种用法

  > 对于单选按钮，复选框及选择框的选项，`v-model` 绑定的值通常是静态字符串 (对于复选框也可以是布尔值)：
  >
  > ```html
  > <!-- 当选中时，`picked` 为字符串 "a" -->
  > <input type="radio" v-model="picked" value="a">
  >
  > <!-- `toggle` 为 true 或 false -->
  > <input type="checkbox" v-model="toggle">
  >
  > <!-- 当选中第一个选项时，`selected` 为字符串 "abc" -->
  > <select v-model="selected">
  >   <option value="abc">ABC</option>
  > </select>
  > ```



#### 异步组件获取

- 使用`keep-alive`保持切换组件的状态

  > ```html
  > <!-- 失活的组件将会被缓存！-->
  > <keep-alive>
  >   <component v-bind:is="currentTabComponent"></component>
  > </keep-alive>
  > ```

- 在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。为了简化，Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染

  > ```js
  > // 一个推荐的做法是将异步组件和 webpack 的 code-splitting 功能一起配合使用：
  > Vue.component('async-webpack-example', function (resolve) {
  >   // 这个特殊的 `require` 语法将会告诉 webpack
  >   // 自动将你的构建代码切割成多个包，这些包
  >   // 会通过 Ajax 请求加载
  >   require(['./my-async-component'], resolve)
  > })
  >
  > // 找到了，局部注册
  > new Vue({
  >   // ...
  >   components: {
  >     'my-component': () => import('./my-async-component')
  >   }
  > })
  > ```

#### 混入 mixins

- 数据对象		数据对象在内部会进行浅合并 (一层属性深度)，在和组件的数据发生冲突时以组件数据优先
- 钩子函数         同名钩子函数将混合为一个数组，因此`都将被调用`。另外，混入对象的钩子将在组件自身钩子**`之前`** 调用
- 对象的选项       例如 `methods`, `components` 和 `directives`，将被混合为同一个对象。两个对象键名冲突时，`取组件对象的键值对`

#### 插槽

- slot  要有默认值

- 父组件模板的所有东西都会在父级作用域内编译；子组件模板的所有东西都会在子级作用域内编译

- 神奇的地方，比如说，有一个公用的组件，一般模样，但是不同应用的部分需要一点点的内部区别，不可能为每一个不同（特别的地方）而去创建一个新的component，这时候可以选择为`待办项<slot v-bind:todo="xxx"></slot>`定义一个不一样的 `<template>` 作为替代方案，并且可以通过 `slot-scope = yyy` 特性从子组件获取数据`yyy.todo.mm`来定制特别的 地方

  > 可能比较绕
  >
  > ```html
  > // template.vue
  > <ul>
  >   <li
  >     v-for="todo in todos"
  >     v-bind:key="todo.id"
  >   >
  >     <!-- 我们为每个 todo 准备了一个插槽，-->
  >     <!-- 将 `todo` 对象作为一个插槽的 prop 传入。-->
  >     <slot v-bind:todo="todo">
  >       <!-- 回退的内容 -->
  >       {{ todo.text }}
  >     </slot>
  >   </li>
  > </ul>
  >
  > // main.html
  > <todo-list v-bind:todos="todos">
  >   <!-- 将 `slotProps` 定义为插槽作用域的名字 -->
  >   <template slot-scope="slotProps">
  >     <!-- 为待办项自定义一个模板，-->
  >     <!-- 通过 `slotProps` 定制每个待办项。-->
  >     <span v-if="slotProps.todo.isComplete">✓</span>
  >     {{ slotProps.todo.text }}
  >   </template>
  > </todo-list>
  >
  > // 可以在支持的环境下 (单文件组件或现代浏览器)，在这些表达式中使用 ES2015 解构语法
  > <todo-list v-bind:todos="todos">
  >   <template slot-scope="{ todo }">
  >     <span v-if="todo.isComplete">✓</span>
  >     {{ todo.text }}
  >   </template>
  > </todo-list>
  > ```




#### VSCode 断点调试VUE

​	之前在Chrome上调试vue，发现一个麻烦的问题，就是在找文件的时候，会比较花时间，所以研究下了vscode下的vue调试模式

> // 且看代码
>
> "version": "0.2.0",
>
>   "configurations": [
>
> ​    {
>
> ​      "type": "chrome",
>
> ​      "request": "launch",
>
> ​      "name": "Vue Launch",
>
> ​      "url": "http://localhost:9527/#/dashboard",
>
> ​      "webRoot": "${workspaceFolder}/src",
>
> ​      "sourceMaps": true,
>
> ​      "sourceMapPathOverrides": {
>
> ​        "webpack:///src/*": "${webRoot}/*"
>
> ​      }} ]}
>
> 这是vscode下的调试.json配置文件
>
> 有一个缺点，就是因为使用node去驱动程序运行的，所以没有检测程序启动，只能监听浏览器的的运行程序
>
> 就是说要按两次调试（绿色按钮），第一次会失败，第二次才成功