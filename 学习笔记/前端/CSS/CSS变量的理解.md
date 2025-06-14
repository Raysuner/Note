---
date: 2025-06-12
tags:
  - css变量
---


## 变量的定义
可以在css文件里面使用:root伪类选择器，它作用于选择文档的根元素。在HTML文档里，根元素通常是<html>元素。因此，:root选择器在html中等同于直接选择<html>元素。

1、 使用:root伪类选择器来声明全局变量

```css
// root.css
:root {
    --main-bg-color: #f0f0f0;
    --main-text-color: #333;
}

// index.css
body {
    background-color: var(--main-bg-color);
    color: var(--main-text-color);
}
```

2、在选择器里面定义css变量，该变量可以在所属选择器内局部使用

3、通过js在style上设置css变量
```js
document.body.style.setProperty("--bg-color", "purple");
```

## 变量的使用
通过var函数，第一个入参是变量名，第二个入参是默认值，如果变量不存在就是用默认值

变量如果是数值的话不能和单位直接连用

```css

.foo {
  --gap: 20;
  /* 无效 */
  margin-top: var(--gap)px;
}
```

必须使用calc函数

```css

.foo {
  --gap: 20;
  margin-top: calc(var(--gap) * 1px)
}
```



[https://www.ruanyifeng.com/blog/2017/05/css-variables.html](https://www.ruanyifeng.com/blog/2017/05/css-variables.html)

