[TOC]

# Webpack 

## 编辑es6

- `babel-preset-env`: 通过自动根据您的目标浏览器或运行时环境确定所需的Babel插件和填充，从而将`ES2015 +`编译为ES5的Babel预设
- `babel-polyfill`: 全局垫片，可在入口entry处引用，一般在文件头`import`，这种方式为应用准备，开发包含es6语法的项目（全局）时应该使用；
- `babel-plugin-transform-runtime`:局部垫片，为开发框架组件准备，提取es6/7方法打包独立文件


## 打包公共模块代码

- `commonchunkplugin`:  单entry是木有作用的，单页面实现懒加载，对此不会生效

- `more entry`

  - `cacheGroups`: 设置`chunks`缓存
    - `initial`: 针对初始化 `entry files`
    - `all`: 针对整个项目一般用`test`匹配对应模块（外置框架模块）
    - `async`: 针对异步模块

- `single entry`

  - 代码分割 懒加载 

    - `require.ensure`: 动态加载模块
    - `require.include`: 在父模块中预置子模块的公共模块

    > 比如`subA and subB` 中都有 `moduleC`，那么在`parentModule` 中，提前`require.include("moduleC")`；那么，`moduleC`的代码就会从两个子模块中抽离出来

    - **`Webpack 3 import(path).then(fn)`**

    > ```js
    > import(/* webpackChunkName: 'subA' */'./subPageA').then((subA) => {
    > console.log(subA);
    > });
    > ```

## 打包样式文件

- `style-loader`

  > 将`css`模块以标签引入进来

  - ​

- `css-loader`

  > `import css`模块，以往除了`css3`原生语法`@import`可以引入样式文件，无法通过其他方式引入`css`模块，
  >
  > 那么这插件便是是我们开发更方便，像`js`一样引入`stylesheet module`


- `module rules`:越放在后面的就越先被执行


- 错误注意

> ```js
> // DeprecationWarning: Tapable.plugin is deprecated. Use new API on `.hooks` instead
> // 不是错误，只是警告，一般在一些依赖更新后弃用了一些方法后，给的提示而已
> // 在module 后面加上@next
>
> $ npm install extract-text-webpack-plugin@next --save-dev 
> ```

- `extract-text-webpack-plugin`

  - allChunks

  > 从所有额外的 chunk(additional chunk) 提取（默认情况下，它仅从初始chunk(initial chunk) 中提取）
  >
  > 从所有额外的 chunk(additional chunk) 提取（默认情况下，它仅从初始chunk(initial chunk) 中提取）
  > 当使用 `CommonsChunkPlugin` 并且在公共 chunk 中有提取的 chunk（来自`ExtractTextPlugin.extract`）时，`allChunks` **必须设置为 `true