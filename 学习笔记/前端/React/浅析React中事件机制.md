## 引文

React 事件机制是 React 中的核心概念之一，通过合成事件和事件委托实现了跨浏览器的事件机制。

在 react 中绑定事件通过如下方式

```jsx
<div onClickCapture={handleClickCapture} onClick={handleClick} />
```

使用 onClickCapture 处理捕获阶段，使用 onClick 处理冒泡阶段。

## 设计动机

我们知道浏览器自身拥有事件机制，但是为什么 react 需要实现自身的事件系统呢？

总共有以下三点原因：

- 受限于市场上浏览器众多，且对于事件的支持有差异，react 想要抹平这种差异。
- 使用事件委托的方式，减少绑定事件的行为次数。
- 支持事件的优先级调度

## 系统结构

react 事件机制

