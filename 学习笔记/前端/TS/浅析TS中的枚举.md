## 引文

枚举是TypeScript引入的新类型，在JavaScript中是没有枚举的。TypeScript官网对枚举是这么定义的

Enums allow a developer to define a set of named constants. Using enums can make it easier to document intent, or create a set of distinct cases. TypeScript provides both numeric and string-based enums.

翻译过来的大概意思就是：开发人员可以利用枚举来定义具名常量集合，使用枚举能够让人更快地记录代码意图或者是创建清晰的案例集合。TypeScript支持数字和字符串的枚举类型。

## 数字类型枚举

如果枚举变量的所有成员没有进行初始化，那么第一位成员的值为0，其它成员依次递加。

```ts
enum E {
  X, // 0
	Y, // 1
  Z  // 2
}
```

如果枚举变量中有成员进行了初始化，那么在它前面的成员的值按照从0开始的方式依次递加，在它之后的成员的值是在当前成员的数值基础上递加。

```ts
enum E {
  X, // 0
	Y = 3, // 3
  Z  // 4
}
```

这里有个需要注意的地方：没有初始化的成员需要放在首位或者是放置在使用常量或常量表达式后面，例如下面使用函数来初始化的方式就会引发错误。

```ts
const getValue = () => 3

enum E {
  X, // 0
	Y = getValue(), // 3
  Z  // 4
}
```

报错信息如下：

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1679320223585-60744d9a-6807-4404-84e3-91ae7edf56e6.png)

如果一定需要通过函数来进行初始化，那么使用函数初始化后的第一位成员需要使用常量或常量表达式来进行初始化，且在此之后的成员无需再进行初始化。

```ts
const getValue = () => 3

enum E {
  X, // 0
	Y = getValue(), // 3
  Z = 4, // 4
  Q  // 5
}
```

## 字符串类型枚举

相比数字类型的枚举变量，字符串类型的枚举变量中每个成员必须使用常量进行初始化

```ts
enum RequestType {
  Get = "GET",
  Post = "POST",
  Head = "HEAD",
  Delete = "DELETE"
}
```

## 混合类型枚举

```ts
enum Test {
  X = 1,
  Y = "1"
}
```

这样的声明在语法层面没有什么问题，但是不建议这么做，想不到要这么做的场景。

## 常量枚举

在enum关键字前面加上const，就是常量枚举了。

```ts
const enum RequestType {
  Get = "GET",
  Post = "POST",
  Head = "HEAD",
  Delete = "DELETE"
} 
```

## 普通枚举和常量枚举的区别

它们两之间的区别主要体现在编译结果上，常量枚举在编译之后，关于常量枚举的定义会被抹除掉，不会产生任何代码；而普通枚举变量在被编译过后产生会产生一段JavaScript代码来模拟实现枚举。

### 普通枚举(enum)

```ts
enum RequestType {
  Get = "GET",
  Post = "POST",
  Head = "HEAD",
  Delete = "DELETE"
} 
```

编译结果：

```js
"use strict";
var RequestType;
(function (RequestType) {
  RequestType["Get"] = "GET";
  RequestType["Post"] = "POST";
  RequestType["Head"] = "HEAD";
  RequestType["Delete"] = "DELETE";
})(RequestType || (RequestType = {}));
```

### 常量枚举(const enum)

```ts
const enum RequestType {
  Get = "GET",
  Post = "POST",
  Head = "HEAD",
  Delete = "DELETE"
} 

const requestType = RequestType.Get;
```

编译结果：

```ts
const requestType = "GET";
```

#### 陷阱

在我们使用TypeScript来进行编译的时候，能够根据整个类型系统来进行代码的转译，但是像Babel这样的工具每次只能处理一个文件的转译，假设有这么一种情况，你在一个文件中定义了常量枚举，在其它文件中引用了这个常量枚举中的成员，因为Babel无法理解整个项目的类型系统，它是无法对常量枚举中的这个成员进行值的替换的，那么在编译过程中就会发生错误。

所以TypeScript官网推荐我们尽量避免使用常量枚举，可以通过在tsconfig中设置isolatedModules配合linter来检查。

## 文章引用

1. [https://stackoverflow.com/questions/56854964/why-is-const-enum-allowed-with-isolatedmodules](https://stackoverflow.com/questions/56854964/why-is-const-enum-allowed-with-isolatedmodules)
2. [https://www.typescriptlang.org/tsconfig#isolatedModules](https://www.typescriptlang.org/tsconfig#isolatedModules)
3. [https://www.typescriptlang.org/docs/handbook/enums.html](https://www.typescriptlang.org/docs/handbook/enums.html)