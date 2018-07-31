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







