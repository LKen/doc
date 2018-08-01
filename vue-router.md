# VUE-Router

#### 起步

- 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)


- 每个路由映射到一个组件

- component，可以是通过Vue.extend()去创建，也可以只是一个组件的配置对象

  > ```js
  > const routes = [
  >   { path: '/foo', component: Foo },
  >   { path: '/bar', component: Bar }
  > ]
  > ```

#### 动态匹配路由

- 路由的页面导航到另外的页面的时候，原来的组件会被复用，因为内部机制问题，两个路由渲染同一个组件的时候，会发生复用，但是就会导致一个问题，就是，组件的生命周期钩子不会再被调用 created mount 等等

- 就是说我们可以监听路由变化，来作出变化

  > ```js
  > const User = {
  >   template: '...',
  >   watch: {
  >     '$route' (to, from) {
  >       // 对路由变化作出响应...
  >     }
  >   }
  > }
  >
  > const User = {
  >   template: '...',
  >   beforeRouteUpdate (to, from, next) {
  >     // react to route changes...
  >     // don't forget to call next()
  >   }
  > }
  > ```

#### 嵌套路由

- **要注意，以 / 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。**

- 如果你I轩昂为某一路由下的空路径渲染点什么，可以这样做

  > ```
  > const router = new VueRouter({
  >   routes: [
  >     {
  >       path: '/user/:id', component: User,
  >       children: [
  >         // 当 /user/:id 匹配成功，
  >         // UserHome 会被渲染在 User 的 <router-view> 中
  >         { path: '', component: UserHome },
  >
  >         // ...其他子路由
  >       ]
  >     }
  >   ]
  > })
  > ```


#### 编程式的导航

- 想要导航到不同的 URL，则使用 `router.push` 方法。这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL

  > 声明式  <router-link :to="...">
  >
  > 编程式  router.push(...)
  >
  > 声明式  <router-link :to="..." replace>
  >
  > 编程式  router.replace(...)
  >
  > 编程式 router.go(1)

- 注意 path 与 params 在一起，params无效

  > ```js
  > const userId = 123
  > router.push({ name: 'user', params: { userId }}) // -> /user/123
  > router.push({ path: `/user/${userId}` }) // -> /user/123
  > // 这里的 params 不生效
  > router.push({ path: '/user', params: { userId }}) // -> /user
  > ```

#### 嵌套视图

- 不同于多个route-view，在component里面嵌入`router-view`,由子路有去填充，对于一般后台系统模板 左右两列挺有用的，注意命名路由，对应嵌入视图

  > ```html
  >
  > <div id="app">
  >   <h1>Nested Named Views</h1>
  >   <router-view></router-view>
  > </div>
  >
  > <script>
  > 	const UserSettingsNav = {
  > 	template: `
  > <div class="us__nav">
  >   <router-link to="/settings/emails">emails</router-link>
  >   <br>
  >   <router-link to="/settings/profile">profile</router-link>
  > </div>
  > `
  > }
  > const UserSettings = {
  > 	template: `
  > <div class="us">
  >   <h2>User Settings</h2>
  >   <UserSettingsNav/>
  >   <router-view class ="us__content"/>
  >   <router-view name="helper" class="us__content us__content--helper"/>
  > </div>
  >   `,
  >   components: { UserSettingsNav }
  > }
  >
  > const UserEmailsSubscriptions = {
  > 	template: `
  > <div>
  > 	<h3>Email Subscriptions</h3>
  > </div>
  >   `
  > }
  >
  > const UserProfile = {
  > 	template: `
  > <div>
  > 	<h3>Edit your profile</h3>
  > </div>
  >   `
  > }
  >
  > const UserProfilePreview = {
  > 	template: `
  > <div>
  > 	<h3>Preview of your profile</h3>
  > </div>
  >   `
  > }
  >
  > const router = new VueRouter({
  >   mode: 'history',
  >   routes: [
  >     { path: '/settings',
  >       // You could also have named views at tho top
  >       component: UserSettings,
  >       children: [{
  >       	path: 'emails',
  >         component: UserEmailsSubscriptions
  >       }, {
  >       	path: 'profile',
  >         components: {
  >         	default: UserProfile,
  >           helper: UserProfilePreview
  >         }
  >       }]
  >     }
  >   ]
  > })
  >
  > router.push('/settings/emails')
  >
  > new Vue({
  > 	router,
  >   el: '#app'
  > })
  > </script>
  >
  > ```

  ​

#### 重定向与重名

- 别名，值得注意一下，有时候，简单的路由可以配置详细的 别名



#### 导航守卫

- 完整的导航解析流程 

  [Vue-router]: https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E7%BB%84%E4%BB%B6%E5%86%85%E7%9A%84%E5%AE%88%E5%8D%AB	"Vue-router"

  > 1. 导航被触发。
  > 2. 在失活的组件里调用离开`beforeRouteLeave` 守卫。 
  > 3. 调用全局的 `beforeEach` 守卫。
  > 4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
  > 5. 在路由配置里调用 `beforeEnter`。
  > 6. 解析异步路由组件。
  > 7. 在被激活的组件里调用 `beforeRouteEnter`。
  > 8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
  > 9. 导航被确认。
  > 10. 调用全局的 `afterEach` 钩子。
  > 11. 触发 DOM 更新。
  > 12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数



#### 路由懒加载

- 异步加载组件

  > // 全局注册
  >
  > ```js
  > Vue.component(
  >   'async-webpack-example',
  >   // 这个 `import` 函数会返回一个 `Promise` 对象。
  >   () => import('./my-async-component')
  > )
  > ```

  > 局部注册
  >
  > ```js
  > new Vue({
  >   // ...
  >   components: {
  >     'my-component': () => import('./my-async-component')
  >   }
  > })
  > ```

- 按模块去把组件 分块打包

  > ```js
  > const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
  > const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
  > const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
  > ```

