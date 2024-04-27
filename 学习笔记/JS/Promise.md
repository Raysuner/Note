![](https://cdn.nlark.com/yuque/0/2023/jpeg/10382191/1697345563466-b332857d-1c04-4afc-977d-1e5b04d3f103.jpeg)

## 为什么需要Promise？

JavaScript在执行异步操作时，我们并不知道什么时候完成，但是我们又需要在这个异步任务完成后执行一系列动作，传统的做法就是使用回调函数来实现，下面举个常见的例子。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
  <script>
    function loadImage(imgUrl, callback) {
      const img = document.createElement("img");
      img.onload = function () {
        callback(this);
      };
      img.src = imgUrl;
    }

    loadImage(
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/1716878f-79a2-4db1-af8c-b9c2039f0b3c_product_W572_H370.jpg",
      (img) => document.body.appendChild(img)
    );
  </script>
</html>
```

上面这个例子会在图片加载完成后将图片放置在body元素下，随后在页面上也会展示出来。

但是如果我们需要在加载完这张图片后再加载其它的图片呢，只能在回调函数里面再次调用loadImage

```js
loadImage(
  "https://travel.12306.cn/imgs/resources/uploadfiles/images/1716878f-79a2-4db1-af8c-b9c2039f0b3c_product_W572_H370.jpg",
  (img) => {
    document.body.appendChild(img);
    loadImage(
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/8b36f9a7-f780-4e71-b719-9300109a9ff2_product_W572_H370.jpg",
      (img) => {
        document.body.appendChild(img);
      }
    );
  }
);
```

继续增加一张图片呢？

```js
loadImage(
  "https://travel.12306.cn/imgs/resources/uploadfiles/images/1716878f-79a2-4db1-af8c-b9c2039f0b3c_product_W572_H370.jpg",
  (img) => {
    document.body.appendChild(img);
    loadImage(
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/8b36f9a7-f780-4e71-b719-9300109a9ff2_product_W572_H370.jpg",
      (img) => {
        document.body.appendChild(img);
        loadImage(
          "https://travel.12306.cn/imgs/resources/uploadfiles/images/6d77d0ea-53d0-4518-b7e9-e53795b4920c_product_W572_H370.jpg",
          (img) => {
            document.body.appendChild(img);
          }
        );
      }
    );
  }
);
```

如果按照上述的方式再增加图片，我们就需要在每层的回调函数里面调用loadImage，就形成了所谓的回调地狱。

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1697202947496-a4a5bcdc-4699-4270-8d3f-a86fa1d706db.png)

## Promise

### 定义

Promise是一种解决异步编程的方案，它比传统的异步解决方案更加直观和靠谱。

### 状态

Promise对象总共有三种状态

- pending：执行中，Promise创建后的初始状态。
- fulfilled：执行成功，异步操作成功取得预期结果后的状态。
- rejected：执行失败，异步操作失败未取得预期结果后的状态。

### 创建方法

```js
const promise = new Promise((resolve, reject) => {})
```

Promise构造函数接收一个函数，这个函数可以被称为执行器，这个函数接收两个函数作为参数，当执行器有了结果后，会调用两个函数之一。

- resolve：在函数执行成功时调用，并且把执行器获取到的结果当成实参传递给它，调用形式如resolve(获取到的结果)
- reject：函数执行失败时调用，并且把具体的失败原因传递给它，调用形式如reject(失败原因)

注意：resolve和reject两个回调函数在Promise类内部已经定义好函数体，如果想了解实现的可以在网上搜索Promise的源码实现。

```js
const promise = new Promise((resolve, reject) => {
  /* 做一些需要时间的事，之后调用可能会resolve 也可能会reject */
  setTimeout(() => {
    const random = Math.random()
    console.log(random)
    if (random > 0.5) {
      resolve('success')
    } else {
      reject('fail')
    }

  }, 500)
})

console.log(promise)
```

在浏览器控制执行上面这段代码

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1697261172260-9eb27aa0-7ad5-4359-952e-c87f5fd48da4.png)

可以看到刚开始promise的状态是pending状态，500ms后promise的状态转变为rejected。

### 状态转换

当执行器获取到结果后，并且调用resolve或者reject两个函数中的一个，整个promise对象的状态就会发生变化。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/10382191/1697346976230-41f10f98-e5e6-41bd-90b1-025b26a5705b.jpeg)

这个状态的转换过程是不可逆的，一旦发生转换，状态就不会再发生变化了。

### 实例方法

promise对象里面有两个函数用来消费执行器产生的结果，分别是then和catch，而finally则用来执行清理工作。

#### then

then这个函数接收两个函数作为参数，当执行器传递的结果状态是fulfilled，第一个函数参数会接收到执行器传递过来的结果当做参数，并且执行；当执行器传递的结果状态为rejected，那么作为第二个函数参数会收到执行器传递过来的结果当做参数，并且执行。

```js
const promise = new Promise((resolve, reject) => {
  /* 做一些需要时间的事，之后调用可能会resolve 也可能会reject */
  setTimeout(() => {
    const random = Math.random();
    if (random > 0.5) {
      resolve("success");
    } else {
      reject("fail");
    }
  }, 500);
});

console.log(promise);

promise.then(
  (res) => console.log("resolved: ", res),  // 生成的随机数大于0.5，则会执行这个函数
  (err) => console.error("rejected: ", err)  // 生成的随机数小于0.5，则会执行这个函数
);
```

可以尝试多次执行上面这段代码，注意控制台打印信息，看是否符合上面的结论。

如果我们只对成功的情况感兴趣，那么我们可以只为then函数提供一个函数参数。

```js
const promise = new Promise((resolve) => setTimeout(() => resolve("done"), 1000))

promise.then(console.log) // 1秒后打印done
```

如果我们只对错误的情况感兴趣，那么我们可以为then的第一个参数提供null，在第二个参数提供具体的函数

```js
const promise = new Promise((resolve, reject) =>
  setTimeout(() => reject("fail"), 1000)
);

promise.then(null, console.log); // 1秒后打印fail
```

#### catch

catch这个函数接收一个函数作为参数，当执行器传递的结果状态为rejected，函数才会被调用。

```js
const promise = new Promise((reject) => setTimeout(() => reject("fail"), 500)).catch(
  console.log
)

promise.catch(console.log)
```

可能有同学会发现，传递给catch的参数好像和传递给then的第二个参数长得一模一样，两种方式有什么差异吗？

答案是没有，`then(null, errorHandler)`和`catch(errorHandler)`这两种用法都能达到一样的效果，都能消费执行器执行失败时传递的原因。

#### finally

常规的try-catch语句有finally语句，在promise中也有finally，它接收一个函数作为参数，无论执行器得到的结果状态是fulfilled还是rejected，这个函数参数是一定会被执行的。

finally的目的是用来执行清理动作的，例如请求已经完成，停止显示loading图标。

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const random = Math.random();
    if (random > 0.5) {
      resolve("success");
    } else {
      reject("fail");
    }
  }, 500);
});

promise
  .finally((res) => {
    console.log("======res======", res); // 打印undefined
    console.log("task is done, do something");
  })
  .then(console.log, console.error); // 打印success或者fail
```

通过打印结果可以确定两点

- 传递给finally的函数也不会接收到执行器处理后的结果。
- finally函数不参与对执行器产生结果的消费，将执行器产生的结果传递给后续的程序去进行消费。

手动实现下finally函数，对上面说到的这两个点就会非常清晰

```js
Promise.prototype._finally = function (callback) {
  return this.then(
    (res) => {
      callback();
      return res;
    },
    (err) => {
      callback();
      throw err;
    }
  );
};
```

### 链式调用

前面介绍的then、catch以及finally函数在调用后都会返回promise对象，进而可以再次调用then、catch以及finally，这样就可以进行链式调用了。

```js
const promise = new Promise((resolve) => {
  setTimeout(() => resolve(1), 1000);
});

promise
  .then((res) => {
    console.log(res); // 1
    return res * 2;
  })
  .then((res) => {
    console.log(res); // 2
    return res * 2;
  })
  .then((res) => {
    console.log(res); // 4
    return res * 2;
  })
  .then((res) => {
    console.log(res); // 8
  });
```

我们用链式调用的方式来优化先前加载图片的代码。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
  <script>
    function loadImage(imgUrl) {
      return new Promise((resolve) => {
        const img = document.createElement("img");
        img.onload = function () {
          resolve(this);
        };
        img.src = imgUrl;
      });
    }

    loadImage(
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/1716878f-79a2-4db1-af8c-b9c2039f0b3c_product_W572_H370.jpg"
    )
      .then((img) => {
        document.body.appendChild(img);
        return loadImage(
          "https://travel.12306.cn/imgs/resources/uploadfiles/images/8b36f9a7-f780-4e71-b719-9300109a9ff2_product_W572_H370.jpg"
        );
      })
      .then((img) => {
        document.body.appendChild(img);
        return loadImage(
          "https://travel.12306.cn/imgs/resources/uploadfiles/images/6d77d0ea-53d0-4518-b7e9-e53795b4920c_product_W572_H370.jpg"
        );
      })
      .then((img) => {
        document.body.appendChild(img);
      });
  </script>
</html>
```

注意：刚刚接触promise的同学不要犯下面这种错误，下面这种代码也是回调地狱的例子。

```js
    loadImage(
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/1716878f-79a2-4db1-af8c-b9c2039f0b3c_product_W572_H370.jpg"
    ).then((img) => {
      document.body.appendChild(img);
      loadImage(
        "https://travel.12306.cn/imgs/resources/uploadfiles/images/8b36f9a7-f780-4e71-b719-9300109a9ff2_product_W572_H370.jpg"
      ).then((img) => {
        document.body.appendChild(img);
        loadImage(
          "https://travel.12306.cn/imgs/resources/uploadfiles/images/6d77d0ea-53d0-4518-b7e9-e53795b4920c_product_W572_H370.jpg"
        ).then((img) => {
          document.body.appendChild(img);
        });
      });
    });
```

### 静态方法

#### Promise.resolve

用来生成状态为`fulfilled`的promise对象，使用方式如下

```js
const promise = Promise.resolve(1) // 生成值为1的promise对象
```

代码实现如下

```js
Promise.resolve2 = function (value) {
  return new Promise((resolve) => {
    resolve(value);
  });
};
```

#### Promise.reject

用来生成状态为`rejected`的promise对象，使用方式如下

```js
const promise = Promise.reject('fail')) // 错误原因为fail的promise对象
```

代码实现如下

```js
Promise.reject2 = function(value) {
  return new Promise((_, reject) => {
    reject(value)
  })
}
```

#### Promise.race

接收一个可迭代的对象，并将最先执行完成的promise对象返回。

```js
 const promise = Promise.race([
  new Promise((resolve, reject) => setTimeout(() => reject(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(2), 1000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
])

promise.then(console.log) // 打印2
```

代码实现

```js
Promise.race2 = function (promises) {
  return new Promise((resolve, reject) => {
    if (!promises[Symbol.iterator]) {
      reject(new Error(`${typeof promises} ${promises} is not iterable`));
    }
    for (const promise of promises) {
      Promise.resolve(promise).then(resolve, reject);
    }
  });
};
```

#### Promise.all

假设我们希望并行执行多个promise对象，并等待所有的promise都执行成功。

接收一个可迭代对象(通常是promise数组)，当迭代对象里面每个值都被resolve时，会返回一个新的promise，并将结果数组进行返回。当迭代对象里面有任意一个值被reject时，直接返回新的promise，其状态为rejected。

```js
 const promise = Promise.all([
  new Promise((resolve, reject) => setTimeout(() => reject(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(2), 1000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
])

promise.then(console.log, co sole.error) // console.error打印1
```

这里需要注意一个点，结果数组的顺序和源promise的顺序是一致的，即使前面的promise耗费时间最长，其结果也会放置在结果数组第一个。

```js
 const promise = Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(2), 1000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
])

promise.then(console.log) // 打印结果[1, 2, 3]
```

我们针对图片加载的例子使用Promise.all来实现。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body></body>
  <script>
    function loadImage(imgUrl) {
      return new Promise((resolve) => {
        const img = document.createElement("img");
        img.onload = function () {
          resolve(this);
        };
        img.src = imgUrl;
      });
    }

    const imgUrlList = [
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/1716878f-79a2-4db1-af8c-b9c2039f0b3c_product_W572_H370.jpg",
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/8b36f9a7-f780-4e71-b719-9300109a9ff2_product_W572_H370.jpg",
      "https://travel.12306.cn/imgs/resources/uploadfiles/images/6d77d0ea-53d0-4518-b7e9-e53795b4920c_product_W572_H370.jpg",
    ];
    const promiseList = imgUrlList.map((item) => loadImage(item));

    const promise = Promise.all(promiseList).then((imglist) => {
      imglist.forEach((item) => document.body.appendChild(item));
    });
  </script>
</html>
```

代码实现

```js
Promise.all2 = function (promises) {
  return new Promise((resolve, reject) => {
    if (!promises[Symbol.iterator]) {
      reject(new Error(`${typeof promises} ${promises} is not iterable`));
    }
    const len = promises.length;
    const result = new Array(len);
    let count = 0;
    if (!len) {
      resolve(result);
      return;
    }
    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i]).then(
        (res) => {
          count++;
          result[i] = res;  // 保证结果数组的放置顺序
          if (count === len) {
            resolve(result);
          }
        },
        (err) => {
          reject(err);
        }
      );
    }
  });
};
```

#### Promise.allSettled

前面提到的`Promise.all`遇到任意一个`promise reject`，那么`Promise.all`会直接返回一个`rejected`的promise对象。而`Promise.allSetled`只需要等待迭代对象内所有的值都完成了状态的转变，无论迭代对象里面的值是被resolve还是reject，那么就会返回一个状态为`fulfilled`的promise对象，并以包含对象数组的形式返回结果。

```js
const promise = Promise.allSettled([
  new Promise((resolve, reject) => setTimeout(() => reject(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(2), 1000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000)),
]);

promise.then(console.log, console.error); 

// 打印结果

// [
//   { status: 'rejected', reason: 1 },
//   { status: 'fulfilled', value: 2 },
//   { status: 'fulfilled', value: 3 }
// ]
```

代码实现

```js
Promise.allSettled2 = function (promises) {
  const resolveHandler = (res) => ({ status: "fulfilled", value: res });
  const rejectHandler = (err) => ({ status: "rejected", reason: err });
  return Promise.all(
    promises.map((item) => item.then(resolveHandler, rejectHandler))
  );
};
```

## 使用场景

大多数异步任务场景都可以使用promise，例如网络请求、文件操作、数据库操作等。当然不是所有的异步任务场景都适合使用promise，例如在事件驱动的编程模型中，使用时间监听器和触发器来处理异步操作更加自然和直观。

在JavaScript中，async和await提供基于promise更高级的异步编程方式，其使用方式看起来就像同步操作一样，更加直观。在使用promise的同时，可以配合async和await体验更好的异步编程。

## 一个小问题

我们前面在讲catch的时候说到了，catch(errorHandler)其实就是then(null, errorHandler)的简写，那么下面两种写法会有区别吗？

```js
// 写法1
promise.then(resolveHandler, rejectHandler)

// 写法2
promise.then(resolveHandler).catch(rejectHandler)
```

答案是不一样，假如在resolveHandler里面抛出错误，写法1最终会获得一个rejected的promise，而写法二由于后续有catch方法，所以即使f1里面有抛出异常，也能得到处理。