---
date: 2025-05-17
tags:
  - webpack
  - loader
---

## Loader 是什么？

Loader 本质上是一个函数，充当非 js 模块的转译器，把模块转换成 webpack 认识的模样。

### Loader 分类

Loader 可以通过执行阶段、执行顺序、数据格式的方式区分 loader 类型。

#### 顺序

在 webpack 配置文件里面可以通过 enforce 属性来配置执行顺序。
```js
module.exports = { module: { rules: [ { enforce: 'pre', 
// pre-loader 
test: /\.js$/, 
use: ['eslint-loader'] 
}, { test: /\.js$/, use: ['babel-loader'] // 默认顺序 }, { enforce: 'post', // post-loader test: /\.js$/, use: ['terser-loader'] } ] } };
```


## Loader 有什么作用？

- 将其它模块转换成 webpack 认识的 js 模块。
- 对模块进行预处理，例如 babel 将高版本 js 转换成低版本 js。

## 如何使用

在 webpack 配置文件里的 module 属性上配置规则

```js
module.exports = {
  module: {
   rules: [
     {
       test: /\.js$/,
       use: {
         loader: "babel-loader",
         options: {
           cacheDirectory: true,
         },
       },
       include: path.resolve(__dirname, "src"),
     },
     {
       test: /\.css$/,
       use: [
         // "style-loader",
         "./src/loader/style-loader.js",
         {
           loader: "css-loader",
           options: {
             sourceMap: false,
             // modules: true,
           },
         },
       ],
     },
     {
       test: /\.(png|jpg)$/,
       type: "asset/resource",
     },
     {
       test: /\.svg$/,
       type: "asset/source",
     },
   ],
 },
}
```

## 原理


