# 引言

我们在一些面试题中经常会遇到一个问题，那就是**state的改变是同步还是异步的。**当初自己只是一味的记住事件触发导致state的改变是异步的，其它情况下都是同步的，并不理解其中的缘由。接下来就为大家梳理下其中的缘由，知其然也要知其所以然。

这篇文章我会分别会对类组件和函数组件中的state的使用和更新机制进行简单的介绍。

# 类组件中的state

## 基本用法

在书写类组件时，通过在类中声明`state`变量来维护状态，通过`setState`方法来更新存储的状态信息

```js
this.setState(updater[, callback])
```

`obj`这个参数可以传递两个类型：

- 对象类型：当传入的参数是对象类型时，那么会将这个对象合并到`state`里面去，更新state的状态。
- 函数类型：会将当前的state和props当做参数，返回值用于与原来的`state`进行合并，更新state的状态。

`callback`回调函数中可以获取到最新的`state`状态信息，可以作为依赖`state`变化执行一些副作用操作。

## state更新机制

```js
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      hello: "hello world",
    };
  }

  handleClick = () => {
    this.setState({ count: this.state.count + 1 }, () => {
      console.log("callback1", this.state.count);
    });
    console.log("log1", this.state.count);

    this.setState({ count: this.state.count + 1 }, () => {
      console.log("callback2", this.state.count);
    });
    console.log("log2", this.state.count);

    this.setState({ count: this.state.count + 1 }, () => {
      console.log("callback3", this.state.count);
    });
    console.log("log3", this.state.count);
  };

  render() {
    console.log("render this", this);
    return (
      <button onClick={this.handleClick}>
        {this.state.hello}: {this.state.count}
      </button>
    );
  }
}
```

打印结果：log1 0 ，log2 0，log3 0；cb1 1，cb2 1，cb3 1

按照原先的想法执行一次点击操作会执行3次加1的操作，但是从打印结果上来看丝毫没有生效，一起来看下执行流程。

![](https://cdn.nlark.com/yuque/0/2022/svg/10382191/1661664329017-24fa79ec-7209-4402-a49c-9e74f19d8e42.svg)

其实每次在执行setState的时候，state的状态信息还没有被更新过，每次都是使用的上次的state状态，所以3次打印的结果都是0。这种现象背后的原因是React会等待所有事件处理函数都处理函数调用的setState调用完成之后，再进行统一更新状态，避免重复渲染的问题。

  
**那么能不能更新state而不进行重新渲染呢？答案是不能，原因有以下两点，这个在React官网中有提到。**

- **这样会破坏掉props和state之间的一致性，造成一些难以debug的问题。**
- **一些新功能变得无法实现。**

**在React的issue里面有详细的解释：**[https://github.com/facebook/react/issues/11527](https://github.com/facebook/react/issues/11527)

那么在事件处理函数中为何使用`setTimeout`和`Promise`会打破这种批量更新导致的异步操作？

```js
handleClick = () => {
  setTimeout(() => {
    this.setState({ count: this.state.count + 1 }, () => {
      console.log("callback1", this.state.count);
    });
    console.log("log1", this.state.count);

    this.setState({ count: this.state.count + 1 }, () => {
      console.log("callback2", this.state.count);
    });
    console.log("log2", this.state.count);

    this.setState({ count: this.state.count + 1 }, () => {
      console.log("callback3", this.state.count);
    });
    console.log("log3", this.state.count);
  });
};
```

在上面的代码出对`handleClick`内的代码使用`setTimeout`来进行包裹，现在来看看打印结果  
打印结果：callbakc1 1， log1 1，callbakc2 2， log2 2， callback 3，log3 3。

那么现在程序执行的流程图就变为下面的这种形式了。

![](https://cdn.nlark.com/yuque/0/2022/svg/10382191/1661672223134-6bc1f7fa-e22b-4267-b687-fa6269d24082.svg)

每次调用setState函数后会更新state，然后进行重新进行渲染，随后调用setState提供的回调函数，回调函数执行完成后，代码会继续执行后面的代码。

**注意： 如果React版本是v18的话，setTimeout和Promise也是使用批量更新的方式如果需要及时更新，有下面两个办法**

1. 对`React`做降级处理
2. 使用`ReactDOM.render(<MyApp />, document.getElementById("root"));`，使用这种方式相当于降级处理。

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1661678253960-295265e1-abdd-486e-9afa-99038319776a.png)

**那如何在上述的代码中开启批量更新呢？**

使用ReactDom提供的`unstable_batchedUpdates`开启批量更新。不要被前缀`unstable`所影响，如今这个Api已经内置React v18中去了，无需担心稳定性问题。

在Redux官方文档中，提供了batch函数中，其实就是`unstable_batchedUpdates`的别名。下面提供redux文档的部分内容：

React's unstable_batchedUpdates() API allows any React updates in an event loop tick to be batched together into a single render pass. React already uses this internally for its own event handler callbacks. This API is actually part of the renderer packages like ReactDOM and React Native, not the React core itself.

Since React-Redux needs to work in both ReactDOM and React Native environments, we've taken care of importing this API from the correct renderer at build time for our own use. We also now re-export this function publicly ourselves, renamed to batch(). You can use it to ensure that multiple actions dispatched outside of React only result in a single render update, like this:

```js
handleClick = () => {
  setTimeout(() => {
    ReactDom.unstable_batchedUpdates(() => {
      this.setState({ count: this.state.count + 1 }, () => {
        console.log("callback1", this.state.count);
      });
      console.log("log1", this.state.count);
  
      this.setState({ count: this.state.count + 1 }, () => {
        console.log("callback2", this.state.count);
      });
      console.log("log2", this.state.count);
  
      this.setState({ count: this.state.count + 1 }, () => {
        console.log("callback3", this.state.count);
      });
      console.log("log3", this.state.count);
    }
  });
};
```

# 函数组件中的state

在`React v16.8`中，`Hooks`的出现让函 数组件焕发新生，让当时的`React`开发者感到大为惊叹，从此函数组件成为编写`React`组件的首选方式。

## 基本用法

```js
const [state, dipatchState] = useState(initialData)
```

`state`：维护状态信息的变量

`dipatchState`：用于改变`state`的函数，直接对`state`进行修改是不会被允许的，接收的参数类型如下：

- 非函数：参数会直接替换原有的`state`，而不是进行合并。
- 函数：会接收`state`和`props`两个参数，这两个参数都是当前最新的状态，返回的结果会更新到`state`里面。

## 函数组件中state更新机制

```jsx
function Counter(props) {
  const [count, setCount] = useState(0);

  console.log("outerCount", count);

  const handleClick = () => {
    setCount(count + 1);
    console.log("count1: ", count);
    setCount((count) => {
      console.log('funcCount', count);
      return count + 1;
    });
    console.log("count2: ", count)
  };

  return <button onClick={handleClick}>count：{count}</button>;
}
```

打印结果：count1 0，count2 0，funcCount 1，outerCount 1。

前三次打印结果都是0，背后的原因很简单，所有改变的`state`只有在下一次函数组件被执行的时候才会更新，在当前函数的执行上下文里的count都是本次函数组件渲染时的初始值。

注意：通过给dispathState传递的函数参数能够拿到最新的state，但不会影响当前函数执行上下文中state的值

在类组件中，可以通过给setState传递回调函数来监测state的变化，那么在函数组件中如何来检测state的变化呢？

答案是可以通过向useEffect的依赖中加入state，当state变化后，useEffect里面的函数会被重新执行。

把上面的代码稍稍精简一下

```jsx
function Counter(props) {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return <button onClick={handleClick}>count：{count}</button>;
}
```

假设点击两次按钮，那么流程就是

![](https://cdn.nlark.com/yuque/0/2022/svg/10382191/1661695100279-9790bc31-4991-4ac5-82e0-f265e3a6eb30.svg)

每次执行函数组件都会形成一次快照，上图每个块状图就是一次快照，每次快照都有自己的`props`、`state`、事件处理函数等其它在函数组件里面声明的变量和函数。

```jsx
// 第一次渲染
function Counter(props) {
  count = 0

  const handleClick = () => {
    setCount(count + 1);
  };

  <button onClick={handleClick}>count：{count}</button>;
}

// 第二次渲染
function Counter(props) {
  count = 1

  const handleClick = () => {
    setCount(count + 1);
  };

  <button onClick={handleClick}>count：{count}</button>;
}

// 第三次渲染
function Counter(props) {
  count = 2

  const handleClick = () => {
    setCount(count + 1);
  };

  <button onClick={handleClick}>count：{count}</button>;
}
```

**如何理解每次渲染都有自己的事件处理函数？借助**[**Dan**](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/#%E6%AF%8F%E4%B8%80%E6%AC%A1%E6%B8%B2%E6%9F%93%E9%83%BD%E6%9C%89%E5%AE%83%E8%87%AA%E5%B7%B1%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86%E5%87%BD%E6%95%B0)**提供的例子来说明问题。**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

在3s的时间内，连续点击2次以上，发现在3s过后，依然能够正确的展示数字。每次渲染都产生了新的`handleAlertClick`事件处理函数，这个函数捕获了当次渲染的`count`。这里其实借助了闭包的特性，顺利捕获了正确的count。

**注意：使用useState提供的dipatch函数更新state，dispatch函数会浅比较前后两次state，如果比较相等，那么会造成视图不更新。**

```jsx
function App(){
    const [ state, dispatchState ] = useState({ name:'ztq' })
    
    const handleClick = ()=>{ // 点击按钮，视图没有更新。
        state.name = 'raysuner'
        dispatchState(state) // 直接改变 `state`，在内存中指向的地址相同。
    }
    
    return (
      <div>
         <span> { state.name }</span>
        <button onClick={ handleClick }>changeName</button>
      </div>
    )
}
```