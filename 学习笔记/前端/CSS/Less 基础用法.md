## 注释

方法一：通过双斜杠`//`的方式来进行注释，使用此方法来进行注释在编译成css后，注释在css文件中不会进行展示。

方法二：通过斜杠加型号`/** */`的方式来进行注释，注释会留在编译生成的`css`文件中。

## 变量

### 变量的基础使用

变量的定义是通过`@变量: 变量值`的形式完成的。

```less
@width: 100px; // 这里一定需要有分号结尾

.box {
	width: @width;
}

// 编译结果
.box {
  width: 100px;
}
```

### 变量的扩展使用

`less`变量不仅仅能代表平常的`css`属性值，有其它很多的场景下也能使用到`less`变量。

#### 选择器

```less
@selector: h2;

@{selector} {
  color: red;
}

// 编译结果
h2 {
  color: red;
}
```

#### URL路径

在`less`使用路径来引入文件时，需要在路径两边加上单引号或者双引号

```less
@images: '../images';  // 这里必须要有引号

.box {
  background-image: url('@{images}/img.jpg'); // 这里也必须要是使用引号，并且在路径中使用变量时需要大括号包裹变量
}

// 编译结果
.box {
  background-image: url('../images/img.jpg')
}
```

在使用`import`关键字引入其它`less`时，变量也能发挥作用

```less
// external.less
@color: red;

// index.less
@path: '../style/'
@import '@{path}/external.less'

.box {
  color: @color;
}

// 编译结果
.box {
  color: red;
}
```

#### 属性名

变量值是属性名或属性名一部分时，在引用的时候需要带上大括号。

```less
@color: color;

.box {
  @{color}: green;
	background-@{color}: red;
}

// 编译结果

.box {
  color: green;
	background-color: red;
}
```

## 混合方法

有写过`css`的同学应该知道，在一些情况下，部分类选择器有相同的`css`属性设置，如果如果想要在各个类选择器中复用这部分`css`属性，那么需要把公共的`css`属性单独拿出来，并且为每个类选择器设置这些公共`css`属性，然后再单独为每个类选择器设置各自的`css`属性。

```less
.box1,.box2 {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
}

.box1 {
  // box1的css属性设置
}

.box2 {
  // box2的css属性设置
}
```

在`less`中，可以把这些步骤进行简化，允许将一个类的属性作为另一个类的属性，这样可以存储多个属性的设置，在其它的类中得到复用。

### 基础使用方法

```less
.box {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
}

.box1 {
  .box();
  // box1的css属性设置
}

.box2 {
  .box;
  // box2的css属性设置
}

// 编译结果
.box {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
}

.box1 {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
  // box1的css属性设置
}
.box2 {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
  // box2的css属性设置
}
```

注意： 你可能会看到有些less文件里面的混合方式没有在类选择器后加上括号，例如.box，这种方式也是可行的，无需纠结使用哪种方式，自己在编写less时保持风格统一即可。

这里看到公共样式也被编译到`css`中去了，如果不想公共样式被编译到`css`中去，那么只需要在类名后面加上括号。

```less
.box() {            // 增加括号
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
}

.box1 {
  .box();
  // box1的css属性设置
}

.box2 {
  .box;
  // box2的css属性设置
}

// 编译结果

.box1 {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
  // box1的css属性设置
}
.box2 {
  color: #222;
  background-color: #ddd;
  border-radius: 8px;
  // box2的css属性设置
}
```

可以发现`.box`这个类没有在`css`中出现，这样就达到了目的。

### 混合参数

`less`中混合的用法和`js`中函数的用法比较相似，既可以通过传参的方式来传值，也可以给与参数默认值，下面给出示例代码

```less
@borderWidth: 50px;
@borderStyle: dotted;
@borderColor: green;

.box(@width: 10px, @style: solid, @color: red) {
  margin-top: 20px;
  width: 200px;
  height: 200px;
  background-color: lightskyblue;
  border: @width @style @color;
}

.box1 {
  .box(); // 没有传入参数，那么使用函数默认参数
}

.box2 {
  .box(@borderWidth, @borderStyle, @borderColor); 
}

.box3 {
  .box(@style: double);  // 为单个参数传参
}

// 编译结果

.box1 {
  margin-top: 20px;
  width: 50px;
  height: 50px;
  background-color: lightskyblue;
  border: 10px solid red;
}
.box2 {
  margin-top: 20px;
  width: 50px;
  height: 50px;
  background-color: lightskyblue;
  border: 50px dotted green;
}
.box3 {
  margin-top: 20px;
  width: 50px;
  height: 50px;
  background-color: lightskyblue;
  border: 10px double red;
}
```

### 匹配

可以利用关键字进行匹配，这个和函数重载的方式类似

```less
.triangle(@_, @width: 10px, @color: lightskyblue) { // @_代表占位符
  width: 0px;
  height: 0px;
  border: @width solid @color;
}

.triangle(top, @width: 10px, @color: lightskyblue) {
  border-color: transparent transparent @color transparent;
}

.triangle(right, @width: 10px, @color: lightskyblue) {
  border-color: transparent transparent transparent @color;
}

.triangle(bottom, @width: 10px, @color: lightskyblue) {
  border-color: @color transparent transparent transparent;
}

.triangle(left, @width: 10px, @color: lightskyblue) {
  border-color: transparent @color transparent transparent;
}

.triangle1 {
  .triangle(top);
}

.triangle2 {
  .triangle(right);
}

.triangle3 {
  .triangle(bottom);
}

.triangle4 {
  .triangle(left);
}

// 编译结果

.triangle1 {
  width: 0px;
  height: 0px;
  border: 10px solid lightskyblue;
  border-color: transparent transparent lightskyblue transparent;
}
.triangle2 {
  width: 0px;
  height: 0px;
  border: 10px solid lightskyblue;
  border-color: transparent transparent transparent lightskyblue;
}
.triangle3 {
  width: 0px;
  height: 0px;
  border: 10px solid lightskyblue;
  border-color: lightskyblue transparent transparent transparent;
}
.triangle4 {
  width: 0px;
  height: 0px;
  border: 10px solid lightskyblue;
  border-color: transparent lightskyblue transparent transparent;
}
```

### 条件判断

在`less`中没有`if else`这样的判断关键字，但有相应的关键来替代，那就是`when`。

```less
// when的基础用法
.border(@width) when(@width > 200px) {
  border-color: lightskyblue;
}

// not关键字相当于!，用于取反
.border(@width) when not (@width > 200px) {
  border-color: green;
}

// and相当于&&
.border(@width, @height) when (@width > 200px) and (@height < 200px) {
  border-color: red;
}

// 逗号运算符相当于||
.border(@width, @height) when (@width > 200px), (@height < 200px) {
  border-color: blue;
}
```

### 数量不定的参数

在`less`中混合方法可以像`js`中`rest`语法一样，接收不定长度的参数

```less
.boxShadow(@a, @rest...) {
  width: 275px;
  height: 70px;
  background-color: blue;
  box-shadow: @arguments; // @arguments接收了所有的参数
  text-shadow: @a @rest;
}

.box4 {
  margin-top: 20px;
  .boxShadow(12px, 12px, 0px, 0px, red);
}

// 编译结果

.box4 {
  margin-top: 20px;
  width: 275px;
  height: 70px;
  background-color: blue;
  box-shadow: 12px 12px 0 0 red;
  text-shadow: 12px 12px 0 0 red;
}
```

### 命名空间

命名空间对通用类名进行隔离，避免类名命名冲突的问题

```less
.container1 {
  .title {
    color: #222;
  }
}

.container2 {
  .title {
    color: #ddd;
  }
}

.box1 {
  .container1 > .title
}

.box2 {
  .container2 > .title
}

// 编译结果

.box1 {
  color: #222;
}

.box1 {
  color: #ddd;
}
```

### important关键字

`important`关键字用于增加css属性的优先级，用于覆盖其它的同名`css`属性，当在混合中使用`important`时，类中的所有`css`属性都会被追加上`important`标记。

```less
.base {
  color: red;
  background: green;
}

.box {
  .base() important;
}

// 编译结果

.box {
  color: red !important;
  background: green !important;
}
```

### 循环

在`less`中循环是通过表达式和模式匹配组合的类递归的方式来实现的，这里举个例子，使用`less`中的循环实现对文本行数限制的枚举。

```less
.line(@i) when (@i > 0) {
  .line-@{i} {
    -webkit-line-clamp: @i;
  }
  .line(@i - 1); // 类递归调用
}

.line(3);

// 编译结果
.line-3 {
  -webkit-line-clamp: 3;
}
.line-2 {
  -webkit-line-clamp: 2;
}
.line-1 {
  -webkit-line-clamp: 1;
}
```

### 合并

有些`css`属性需要增加空格或逗号来拼接属性值，在混合场景下，可以通过在属性名后追加后缀完成拼接

#### 逗号

```less
.base-shadow {
  width: 275px;
  height: 70px;
  background-color: lightgreen;
  box-shadow+: 0px 12px 0px 0px lightyellow;
}

.mixin-shadow {
  box-shadow+: 12px 0px 0px 0px lightcoral;
}

// 编译结果

.base-shadow {
  width: 275px;
  height: 70px;
  background-color: lightgreen;
  box-shadow: 0px 12px 0px 0px lightyellow;
}
.mixin-shadow {
  width: 275px;
  height: 70px;
  background-color: lightgreen;
  box-shadow: 0px 12px 0px 0px lightyellow, 12px 0px 0px 0px lightcoral;
}
```

#### 空格

```less
.base-comma {
  transform+_: scale(1.5);  // +_后缀增加空格
}

.mixin-comma {
  .base-comma();
  transform+_: translate(50%); //+_后缀增加空格
}

// 编译结果

.base-comma {
  transform: scale(1.5);
}
.mixin-comma {
  transform: scale(1.5) translate(50%);
}
```

## 转义

通过在转义字符前面加上`~`符号，编译器不会对~修饰的表达式进行编译。

```less
.box {
  color: ~'green';
}

// 编译结果
.box {
  color: green;
}
```

看看`antd`中在是如何使用`less`中转义，这里举一个`[antd modal](https://github.com/ant-design/ant-design/blob/4.x-stable/components/modal/style/modal.less)`组件的例子。

```less
@ant-prefix: ant;
@dialog-prefix-cls: ~'@{ant-prefix}-modal'

@{dialog-prefix-cls} {
  position: fixed;
}
  
// 编译结果
ant-modal {
  position: fixed;
}
```

这里其实就是属性值变量拼串的场景，项目前缀拼上具体的组件名充当组件类名前缀。

可能有同学会疑惑，干么非要使用转义，直接使用变量的基础用法不就行了吗？那就试试看会有什么样的结果。

```less
@ant-prefix: ant;
@dialog-prefix-cls: @{ant-prefix}-modal // error: variable value expected
```

那就换一种方式

```less
@ant-prefix: ant;
@dialog-prefix-cls: "@{ant-prefix}-modal"

// 编译结果
"ant-modal" {         // error: at-rule or selector expected
  z-index: inherit;
  position: fixed;
}
```

`css`对带双引号的选择器是不认识的，开始报错。

结论： 就是在使用变量拼串的时候需要使用转义来辅助使用。

## 继承

就和`oop`语言中的继承有着类似的含义，可以从其它类继承其属性。

```less
.base {
  color: red;
}

.box {
  &:extend(.base);
  background: green;
}

// 编译结果
.base,
.extend-base {
  color: red;
}

.extend-base {
  background: green;
}
```

## 参考

[https://www.w3cschool.cn/less/nested_directives_bubbling.html](https://www.w3cschool.cn/less/nested_directives_bubbling.html)