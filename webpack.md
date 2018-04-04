[TOC]

# Webpack 

## 编辑es6

- `babel-preset-env`: 通过自动根据您的目标浏览器或运行时环境确定所需的Babel插件和填充，从而将`ES2015 +`编译为ES5的Babel预设
- `babel-polyfill`: 全局垫片，可在入口entry处引用，一般在文件头`import`，这种方式为应用准备，开发包含es6语法的项目（全局）时应该使用；
- `babel-plugin-transform-runtime`:局部垫片，为开发框架组件准备，提取es6/7方法打包独立文件


## 打包公共模块代码

- `commonchunkplugin`:  单entry是木有作用的，单页面实现懒加载，对此不会生效