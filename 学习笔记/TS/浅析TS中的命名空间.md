## 引言

在TypeScript 1.5之前只支持命名空间的方式来组织代码，避免全局命名空间污染和命名冲突问题，在当时这是一种非常好的实践方式，虽然现在更推荐使用ESModule的方式来进行模块化地管理，但是使用namespace来构造命名空间的方式依旧被使用，还是建议大家学习下。

我这里会把当时学习和使用namespace时的一些疑惑列举出来，看看自己有没有不清楚的点

![](https://cdn.nlark.com/yuque/0/2023/jpeg/10382191/1680415655505-8ee5a6ba-3073-4944-8c3a-44ecee034c61.jpeg)

## namespace的实现原理

直接通过代码来看实现原理

```ts
namespace Qing {
  interface Person {
    name: string;
    age: number;
  }

  let name = 'qing'
  let age = 18

  function createPerson(name: string, age: number): Person {
    return {name, age}
  }
}
```

编译成js后

```js
"use strict";
var Qing;
(function (Qing) {
    let name = 'qing';
    let age = 18;
    function createPerson(name, age) {
        return { name, age };
    }
})(Qing || (Qing = {}));
```

简单地解释下，就是声明了一个全局变量，然后利用函数立即执行表达式构建了一个局部的作用域，使得外部无法访问到命名空间中的变量和函数。

## export

仔细观察上面的代码其实可以发现，Qing这个命名空间本身都访问不到自己内部的属性了，这就有点绷不住了。原本的意图是要限制外部的访问权限，结果连自己都访问不了，这个时候就需要使用export来进行内容的导出了。

### 作用

我们在使用esmodule来进行导出的时候经常用到export字段，在namespace中也是一样，用来导出命名空间中的属性。

```ts
namespace Qing {
  interface Person {
    name: string;
    age: number;
  }

  let name = 'qing'
  export let age = 18

  export function createPerson(name: string, age: number): Person {
    return {name, age}
  }
}
```

编译结果

```js
"use strict";
var Qing;
(function (Qing) {
    let name = 'qing';
    Qing.age = 18;
    function createPerson(name, age) {
        return { name, age };
    }
    Qing.createPerson = createPerson;
})(Qing || (Qing = {}));
```

我们可以看到之前声明age变量和createPerson方法被挂在到了Qing这个变量上去了，这样命名空间Qing就能访问到导出的属性了。

### 使用场景

其实官网推荐是将你想要导出的内容都使用export来进行导出，据我个人的看过的代码来说，有很多的代码没有对命名空间内部的属性进行导出，但是在外部却在使用。

例一

```ts
// global.d.ts
declare namespace Qing {
  interface Person {
    name: string;
    age: number;
  }
}

// index.ts
function createPerson(name: string, age: string): Qing.Person {
	return {name, age}
}
```

例二

```ts
// global.d.ts
export namespace Qing {
  interface Person {
    name: string;
    age: number;
  }
}

// index.ts
import { Qing } from 'global.d.ts'
function createPerson(name: string, age: string): Qing.Person {
	return {name, age}
}
```

虽然上面的两个例子都没有使用export来导出Person这个类型，但是代码并没有报错，我来解释下原因：

因为在d.ts文件中声明命名空间时，这个命名空间声明的内容会自动默认全局可见，而无需特别使用export进行导出

但是某些场景下，你在不主动export的时候，代码会进行报错。

```ts
// utils.ts
export namespace Util {
  function getLength(str: string): number {
    return str.length;
  }
}

// index.ts
import { Util } from "./utils";

const str = "hello, world";
const length = Util.getLength(str);
console.log(length);
```

报错信息：

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1680435435418-3921f96d-0d8a-4797-925f-d140ed4285a8.png)

出现这种情况的时候，那么就需要使用export将命名空间内部定义过的属性暴露出去。

```ts
// utils.ts
export namespace Util {
  export function getLength(str: string): number {
    return str.length;
  }
}
```

为getLength这个函数前面加上export即可解决。

前面有说到在d.ts文件中不加export也能访问到，那么我这里把utils.ts这个文件改成utils.d.ts文件，是不是就不需要为getLength函数前面添加export？

很可惜在d.ts文件中是不支持函数定义的，也就是带有具体实现的函数。

结论：

**对于ts文件，文件中的命名空间中的内容如果想要被外部访问到，必须要在前面加上export。**

**对于d.ts文件，文件中的命名空间中的内容如果想要被外部访问到，不强制要求加上export。**