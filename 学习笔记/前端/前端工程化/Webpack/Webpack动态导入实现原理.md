## 环境
webpack 5.92.1

## 代码
```javascript
// index.js
import { multiple } from "./module-2";

console.log(multiple(1, 2));

const loadModule1 = () => {
  import("./module-1").then(({ add }) => {
    console.log(add(1, 2));
  });
};

loadModule1();

// module-1.js
export const add = (a, b) => {
  return a + b;
};

// module-2.js
export const multiple = (a, b) => {
  return a * b;
};
```

经过webpack编译及打包后，有了打包文件bundle.js以及拆分出来的1.js，接下来直接查看打包后的代码如何实现动态导入的

首先来看下bundle.js文件里面的内容

![1721139832774-28c69675-9370-42af-a6b1-488be4357dff.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721139832774-28c69675-9370-42af-a6b1-488be4357dff.png?lB1xlwRKms)

我们这里可以看到有一个webpackJsonCallback的函数，上方还有注释，这个函数就是用于加载chunk的jsonp的回调函数的定义，至于是加载什么chunk，一步步地往下看。

**<font style="background-color:#C1E77E;">红框标注的地方非常重要，webpackJsonpCallback劫持了chunkLoadingGlobal.push函数，即劫持了self['webpackChunkwebpack_test'].push函数，同时将原有的chunkLoadingGlobal.push函数当做参数传递</font>**<font style="background-color:#C1E77E;">。</font>



开始动态加载模块代码

![1721142393749-637801c2-0fe8-495e-9525-517c7781e8ee.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721142393749-637801c2-0fe8-495e-9525-517c7781e8ee.png?jTz0nMQknS)

webpack加载模块借助模块索引，上面截图中1表示加载的第1个异步模块。

首先看下__webpack_require__.e干了什么

![1721304773649-2703d8f5-fa35-418e-afc9-60edb9440d90.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721304773649-2703d8f5-fa35-418e-afc9-60edb9440d90.png?E1rx3cnLJ7)

发现是遍历__webpack_require__.f这个对象的key，然后返回一个promise，其中promises是传递给__webpack_require__.f.j，然后在这个函数里面会有promise推入到promises这个数组里。

![1721219356873-0bcd9944-cc8f-43eb-886a-b613f69480b6.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721219356873-0bcd9944-cc8f-43eb-886a-b613f69480b6.png?x1wdBclRc9)

上面这一堆代码看起来很多，简化下来就是先检查chunk有没有被加载过，如果没加过的话就构造chunk的url，调用__webpack_require__.l函数来请求chunk。

![1721309139371-da8d2371-5a87-43fb-a4ff-fc1fd9bad898.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721309139371-da8d2371-5a87-43fb-a4ff-fc1fd9bad898.png?GhUK2tUt8k)

webpack打包过程会对需要动态加载的模块单独分块，利用jsonp方式来加载单独分块的js文件，即1.js。

![1721309179949-5e55baf8-c2e2-46e4-8841-4790f55946a6.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721309179949-5e55baf8-c2e2-46e4-8841-4790f55946a6.png?tzuurJra1j)

前面说过webpackJsonpCallback劫持了`self['webpackChunkwebpack_test'].push`函数，在动态加载完1.js后，执行1.js代码就调用`self['webpackChunkwebpack_test'].push`函数，实际上就是调用了webpackJsonpCallback函数。

![1721312456345-59f43127-6170-4e24-a32c-f48a97be8a71.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721312456345-59f43127-6170-4e24-a32c-f48a97be8a71.png?PKI4iNBplA)

来到webpackJsonpCallback函数

![1721311276753-57a37e42-3864-4ad0-9c8b-e8bad4f0ac10.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721311276753-57a37e42-3864-4ad0-9c8b-e8bad4f0ac10.png?WObtXVJ3dX)

大致做了两件事，一是把动态模块放入到模块数组__webpack_modules__里面，二是resolve在__webpack_require__.f.j函数里面生成的promise，让__webpack_require__.e返回的promise被resolve，后面的then函数才能往下执行。

最后回到开始加载模块的代码处

![1721142393749-637801c2-0fe8-495e-9525-517c7781e8ee.png](http:8.149.242.20:9000/storage/uploads/202505/25/1721142393749-637801c2-0fe8-495e-9525-517c7781e8ee.png?jTz0nMQknS)

因为在webpackJsonpCallback里面把动态模块放到了__webpack_modules__里面，`__webpack_require__.bind(__webpack_require__, 2)`能够直接获取到动态模块，然后返回该模块的导出，最终获取add函数并执行add(1, 2)，至此整个流程已经结束。

