## 背景
在早期的前端技术标准根本没有预料到前端行业会有今天的发展，在设计上存在很多缺陷，随着web 应用复杂性增加，网页已经从展示简单的文案和图像逐渐演变为功能复杂、交互密集的应用程序，这种变化推动了前端模块化的发展，以应对以下几个挑战：
1. 依赖管理混乱
2. 全局作用域污染
3. 代码膨胀

举个例子来说明下：
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Test</title>
  </head>
  <body></body>
  <script src="module-1.js"></script>
  <script src="module-2.js"></script>
</html>
```
module-1.js
```js
const add = (a, b) => a + b;
```
module-2.js
```js
const add = (b, a) => b + a;
```
毫无疑问，在浏览器渲染上面的html代码时，会有报错提示add已经被声明过了，这就是上面提到的全局作用域污染问题
> ==Uncaught SyntaxError: Identifier 'add' has already been declared (at module-2.js:1:1)==

来改动一下两个js文件的代码，module-2.js文件里面声明了一个函数用来实现乘法，module-1.js
里面需要使用，这时如果不调整js文件的引入顺序，那么也会有问题。
module-1.js
```js
const add = (a, b) => a + b;
multi(1,2)
```
module-2.js
```js
const add = (b, a) => b + a;
const multi = (b, a) => b * a;
```
因为module-1.js文件里的js先执行，此时multi函数还没有声明，直接使用的话就产生报错，这就是上面提到的依赖管理问题，当随着工程的日益增长，文件间的依赖关系变得难以追踪，引入的文件变得越来越多，如何配置准确的顺序来加载js资源变得尤为困难。
> ==Uncaught ReferenceError: multi is not defined at module-1.js:2:9==

另外由于在模块化之前，所有的代码都集中在几个js文件内，容易造成代码量快速膨胀的情况。
## JavaScript模块化
为了解决上面的问题，前端引入了模块化的概念，通过将各个功能点切分成独立的小部分，这种方式被称为模块化。
1. 模块可以声明对其它模块的依赖，使得能够正确加载依赖
2. 每个模块有自己的作用域，声明的变量只能在当前模块内访问到，如果需要在其它模块访问，需要在当前模块导入该变量。
3. 由于按照功能点切分成独立的小部分，原先所有代码都挤在几个文件造成代码膨胀的问题自然而然就解决了。

JavaScript模块化经历过早期AMD(require.js)和CMD(sea.js)的过渡，2015年官方推出了ES Module模块化标准。当js文件使用ES Module模块化标准时，可以使用import来引入依赖，

在当时由于大多数浏览器都不支持ES Module，所以在引入ES Module模块化标准后，也引发了新的问题：
1. ES Module这个模块化标准本身存在兼容性问题
2. 模块化切分过细后形成多个文件，在进入页面后需要从服务端下载多个CSS和JS资源，浏览器对同一域名资源请求的并发量也有限制，会影响页面的展示效果。

为了解决上面问题，webpack这样的打包工具应运而生。
## Webpack是什么？
引用Webpack官网的介绍
> ==**webpack** is a _static module bundler_ for modern JavaScript applications==

翻译下来就是webpack是服务于现代JavaScript应用的静态模块打包工具。
## Webpack解决了什么问题？
1. 首先它能够将ES6代码转换成ES5代码，代码中的import和export语句会被编译成其它语句，那么在引入js文件时就不需要带上`type = "module"`来表明这是一个ES Module模块文件，这样就解决了ES Module在浏览器上的兼容性问题。
2. 它能够将细分的模块再重新打包到一起，因为在生产环境也不需要模块化，解决了因模块过细导致频繁请求资源文件的问题。
3. 另外需要支持将不同种类的文件，把所有的文件都当成模块，其中所有的资源文件都通过代码来控制，那么代码和资源文件都有统一的管理方式。

前面两点不需要Webpack也能完成，使用gulp配合插件可以完成文件的编译、压缩以及文件合并。但是第三点无法支持。
## Webpack的打包结果是怎样的？
对上面两个js文件的代码加入模块的导出
module-1.js
```js
export const add = (a, b) => {
  return a + b;
};
```
module-2.js
```js
export const multi = (b, a) => b * a;
```
为了尽可能地观察清楚Webpack的打包结果，我们把配置文件中的mode配置项设置为none，这样Webpack在打包的时候就不会开启任何优化项。
对前面的代码来进行打包试试看，Webpack的打包结果究竟是怎样的？
```js
(() => {
  // webpackBootstrap
  "use strict";
  var __webpack_modules__ = [
    ,
    /* 0 */ /* 1 */
    (__unused_webpack_module, __webpack_exports__, __webpack_require__) => {
      __webpack_require__.r(__webpack_exports__);
      /* harmony export */ __webpack_require__.d(__webpack_exports__, {
        /* harmony export */ add: () => /* binding */ add,
        /* harmony export */
      });
      const add = (a, b) => {
        return a + b;
      };
    },
    /* 2 */
    (__unused_webpack_module, __webpack_exports__, __webpack_require__) => {
      __webpack_require__.r(__webpack_exports__);
      /* harmony export */ __webpack_require__.d(__webpack_exports__, {
        /* harmony export */ multiple: () => /* binding */ multiple,
        /* harmony export */
      });
      const multiple = (a, b) => {
        return a * b;
      };
    },
  ];
  // The module cache
  var __webpack_module_cache__ = {};

  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    var cachedModule = __webpack_module_cache__[moduleId];
    if (cachedModule !== undefined) {
      return cachedModule.exports;
    }
    // Create a new module (and put it into the cache)
    var module = (__webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {},
    });

    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;
  }

  /* webpack/runtime/define property getters */
  (() => {
    // define getter functions for harmony exports
    __webpack_require__.d = (exports, definition) => {
      for (var key in definition) {
        if (
          __webpack_require__.o(definition, key) &&
          !__webpack_require__.o(exports, key)
        ) {
          Object.defineProperty(exports, key, {
            enumerable: true,
            get: definition[key],
          });
        }
      }
    };
  })();

  /* webpack/runtime/hasOwnProperty shorthand */
  (() => {
    __webpack_require__.o = (obj, prop) =>
      Object.prototype.hasOwnProperty.call(obj, prop);
  })();

  /* webpack/runtime/make namespace object */
  (() => {
    // define __esModule on exports
    __webpack_require__.r = (exports) => {
      if (typeof Symbol !== "undefined" && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
      }
      Object.defineProperty(exports, "__esModule", { value: true });
    };
  })();

  var __webpack_exports__ = {};
  // This entry need to be wrapped in an IIFE because it need to be isolated against other modules in the chunk.
  (() => {
    __webpack_require__.r(__webpack_exports__);
    /* harmony import */ var _module_1__WEBPACK_IMPORTED_MODULE_0__ =
      __webpack_require__(1);
    /* harmony import */ var _module_2__WEBPACK_IMPORTED_MODULE_1__ =
      __webpack_require__(2);

    console.log(
      (0, _module_2__WEBPACK_IMPORTED_MODULE_1__.multiple)(
        (0, _module_1__WEBPACK_IMPORTED_MODULE_0__.add)(1, 2),
        3
      )
    );
  })();
})();

```
最外层是一个立即执行函数，使用\_\_webpack_module\_\_数组来存储模块，因为有module-1.js和module-2.js这两个文件，在webpack的世界里，一个文件就是一个模块，所以\_\_webpack_module\_\_数组的长度为2。打包过后的代码通过使用\_\_webpack_module\_\_数组索引来获取对应模块，每个模块其实都是一个立即执行函数，每个立即执行都创建了自己的作用域，外层作用域是无法直接获取到内部的变量，这也解释了为何模块外部无法访问模块内部变量，只有在使用export的方式导出后，外部才能访问到。

> ==上方的打包代码导出的对象是module.exports，这个和CommonJS里面的module.exports不是一回事，只是名称一样==

使用export导出的函数，上面的打包代码，都以属性的方式挂载在立即执行函数module.exports对象上。
