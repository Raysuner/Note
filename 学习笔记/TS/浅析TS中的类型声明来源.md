## TS内置的声明

在ts源码中，有一个lib文件夹，里面会有一系列的lib.xxx.d.ts这样的文件，这些d.ts文件都是为了js引擎和浏览器内置的api声明，因为这些api都是标准化的，一般情况下基本不会发生变动，所以直接被内置到ts包中去。

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1677416982597-ab6ba324-5ebd-449b-831c-04c2edd85cb4.png)

这些api都通过declare的方式来进行声明，并没有具体的定义，因为这些定义已经内置到ts源码中，所以我们只需要在tsconfig.json文件中配置根据实际需求进行引入。

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1680361474828-c24b0cda-c062-4846-ab80-d0b153075bfa.png)

这里有lib里每个配置项对应的含义：[https://www.typescriptlang.org/tsconfig#lib](https://www.typescriptlang.org/tsconfig#lib)

## 通过@type/xxx来引入

之前ts还未完全推广开来的时候，很多项目都是使用的js，例如react，为了在使用react的时候配合ts使用，也会用到@types/xxx这样的方式来引入类型声明。

答案是通过@types这样的方式去进行引入

![](https://cdn.nlark.com/yuque/0/2023/png/10382191/1680356923450-efcb9d5a-c81f-4ddb-86a7-b71433e2a734.png)

我这里安装lodash的类型声明，图中每个lodash中的函数都有对应的类型声明文件

@types这个文件夹会放置在node_modules文件夹下，ts会在加载lib配置项配置的声明文件中的类型声明后，再去加载@types目录下的d.ts文件中的类型声明。

**这里有两个tsconfig文件中配置项必须要介绍一下，一个是typesRoot，另外一个是types。**

1. 在没有配置typesRoot时，ts会默认加载@types目录下的类型声明，但是当配置了typesRoot后，ts会根据配置内容去加载对应文件中的类型声明，如果配置的typesRoot里面没有包含@types，那么编译器将不会去@types文件夹里面去寻找类型声明文件。
2. 在没有配置types时，ts会加载整个@types下所有d.ts文件中的类型声明，当配置了types后，只会加载指定的包。

## 使用自己编写的类型声明

如果自己在编写TypeScript代码，那么完全可以使用自己定义的类型声明，在当前文件定义和使用或者是从其它文件导入。

如果不想通过模块导入的方式来获取类型声明，那么还有什么方法加载声明？

其实可以什么都不做，在没有配置files、include这两个配置项时，编译器会检查包含tsconfig.json文件的目录以及其子目录，当然node_modules会被排除在外，同时也建议把node_modules加入到exclude配置项中去。

当配置files、include来指定编译文件时，编译器在加载两个配置项里面包含d.ts文件后，随后你就可以在整个项目任意位置使用这些类型声明。

注意：上面谈到的类型声明在不导入的情况下可以直接使用的前提条件是，它们被声明全局空间中。

**那么如何声明全局空间和模块内的类型声明？如何去判断类型声明是在全局空间中还是在模块内？  
**在TypeScript 1.5版本之前就只支持命名空间的方式来组织代码，这种方式将相关的代码都放置在命名空间中，能够避免全局命名空间污染和命名冲突，TypeScript最早支持的模块化方案就是以命名空间的方式。

然而随着TypeScript的广泛使用，出现了一些新的问题和新的需求。

越来越多的JavaScript库和框架使用了模块化机制，例如(AMD，Common.js、ESModule)，作为JavaScript语言超集，TypeScript也应该支持这些规范，更好地适应当前的生态系统，也能够方便开发者更好地组织和维护代码。