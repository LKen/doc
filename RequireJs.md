[TOC]

## RequireJS

- > ```text
  > 相对路径的规则:
  > 1.如果\<script\>标签引入require.js时没有指定data-main属性，则以引入该js的html文件所在的路径为根路径。
  > 2.如果有指定data-main属性，也就是有指定入口文件，则以入口文件所在的路径为根路径。在本例子中也就是main.js所在的script文件夹就是根路径，这也是为什么配置jQuery的时候需要返回上层目录再进入lib目录才能找到jQuery文件。
  > 3.如果再require.config()中有配置baseUrl，则以baseUrl的路径为根路径。
  >
  > 以上三条优先级逐级提升，如果有重叠，后面的根路径覆盖前面的根路径。
  > ```


- > \<script src="js/require.js" **defer async="true"** >\</script>
  >
  > **async**属性表明这个文件需要异步加载，避免网页失去响应。IE不支持这个属性，只支持**defer**，所以把defer也写上。

- > \<script src="js/require.js" **data-main="js/main"**>\</script>
  >
  > **data-main**属性的作用是，指定网页程序的主模块。在上例中，就是js目录下面的main.js，这个文件会第一个被require.js加载。由于require.js默认的文件后缀名是js，所以可以把main.js简写成main

- > 对外暴露的变量其实就三个，**requirejs**，**require**，**define**

- `packages` : 将对面的资源位置，对应自定义的简称变量

- `baseUrl`  : 基本路录，改变基目录

- `shim`       :  专门用来配置不兼容的模块，每个模块要定义

  - `exports`值（输出的变量名）: 表明这个模块外部调用时的名称
  - `deps` 数组 : 表明该模块的依赖性


## 关于RequireJS，如何引入文件Template

> 引入插件 `requirejs-text/text.js`
>
> 

