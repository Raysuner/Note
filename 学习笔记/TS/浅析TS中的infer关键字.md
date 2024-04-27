## 引文

在刚接触TypeScript的时候，使用最多的就是type和interface这两个关键字，用来声明类型，其实这样也基本满足日常需求。但是如果需要设计一些高级类型的话，那么仅仅用原来所掌握的TypeScript知识是无法满足需求的。

设计高级类型的话涉及到类型编程的知识点，而类型编程中有两个关键字非常重要，分别是extends和infer。

extends用来约束入参的类型以及进行条件判断，infer用来声明局部的类型变量。

## extends

### 条件判断

我们举个简单的例子来说明

```ts
type isOne<T extends number> = T extends 1 ? true : false
```

我们单独看`T extends 1 ? true : false`这部分，这里和`JavaScript`中的三元表达式并无二致，但是有些同学不清楚其中extends表示的是什么含义。

T extends 1 ? true : false表示的含义其实就是传入的T类型是否能够赋值给字面量1这个类型，如果可以的话，就返回true，否则返回false

### 约束参数类型

继续使用上面的例子，我们单独看`T extends number`，这里extends的意思就是限制传入的T类型必须是number类型，否则报错。

为什么需要有这个限制？

因为我们是要判断类型T是否能够赋值给字面量类型1，但是如果传入的参数都不是number类型，那么就没必要做后续的条件判断了。

### 约束infer推导的局部变量类型

```ts
type GetFirst<T extends string[]> = T extends [infer FirstChar, ...infer Rest]
  ? `${FirstChar}`
  : never;

type Res = GetFirst<['1', '2', '3']>;
```

来看看报错信息，我们传入的参数类型限制为sting类型的数组，但是infer推导出来的类型实际上是unknown类型，所以导致类型不匹配。

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1678285474500-ed3c797d-4eb8-482e-8cae-54d7e06fbb5f.png)

解决办法有三种：

- 对FirstChar使用extends先做过滤

```ts
type GetFirst<T extends string[]> = T extends [infer FirstChar, ...infer Rest]
  ? FirstChar extends string
    ? `${FirstChar}`
    : never
  : never;
```

- FirstChar和string进行交叉运算

```ts
type GetFirst<T extends string[]> = T extends [infer FirstChar, ...infer Rest]
  ? `${FirstChar & string}`
  : never;
```

- 使用infer extends做类型转换

```ts
type GetFirst<T extends string[]> = T extends [
  infer FirstChar extends string,
  ...infer Rest
]
  ? `${FirstChar}`
  : never;
```

`infer extends`是在ts 4.7版本支持，低于这个版本无法使用。

### 类型转换

```ts
type StrToNum<T extends string> = T extends `${infer Num extends number}` ? Num : T

type Res = StrToNum<"1">

// ts 4.7时，返回的结果是type Res = number
// ts 4.8及以上， 返回的结果是type Res = 1
```

## infer

还是拿例子来进行讲解，下面的例子是要提取Promise包裹的类型。

```ts
type PromiseValue<T extends Promise<unknown>> = T extends Promise<infer Value> ? Value : never

type Res = PromiseValue<Promise<string>> // string
```

画个简单的图来描述下如何提取变量类型。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/10382191/1678282252368-36eeaaf0-6ef7-411b-939b-cc97df8d7caa.jpeg)

注意：infer声明的局部变量，只能在条件语句为true的分支里面使用。

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1678284355792-2bcb930e-405d-4136-9d86-150a088ef96a.png)

直接报错表示找不到通过infer声明的局部变量。

## 组合使用

### ReturnType

内置工具类型RetureType用于获取函数的返回值类型。

```ts
type MyReturnType<T extends (...args: any) => any> 
    = T extends (...args: any) => infer R 
        ? R 
        : any;

type Res = MyReturnType<() => string> // type Res = string
```

### Parameters

内置工具类型Parameters用于获取函数的参数类型。

```ts
type MyParameters<T extends (...args: any) => any> 
    = T extends (...args: infer P) => any 
        ? P 
        : never;

type Res = MyParameters<(a: string, b: number) => void>  // type Res = [a: string, b: number]
```