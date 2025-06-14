## 核心定义
基于 ES Module 的静态语法分析的代码消除技术去除未使用的代码

作用：优化打包体积，去除生产环境的死代码

## 背景与痛点
在引入 npm 包时，可能只需要引入其中某个方法，但是如果没有 Tree Shaking ，那么会把整个包打包到产物里，通过 Tree Shaking 可以仅仅只把需要使用的函数方法打包到最终产物里，减少打包体积，这就是按需引入的方式。

## 使用前提
1、使用 ES Module

2、开启生产模式

3、devtool 没有设置为 eval 系列

## 原理
分析依赖 => 标记模块是否被使用 => 未被使用的代码的 export 删除 => 使用 terser 工具删除死代码

## 注意
webpack 配置文件里面也有字段sideEffect，只不过这个是开启是否检测package.json里面sideEffect的值的，生产环境下默认为true。



官方文档：[https://webpack.js.org/guides/tree-shaking/](https://webpack.js.org/guides/tree-shaking/)  
  
[https://github.com/webpack/webpack/tree/main/examples/side-effects](https://github.com/webpack/webpack/tree/main/examples/side-effects)

