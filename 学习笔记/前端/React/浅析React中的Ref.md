## 引言

相信大部分同学对ref的认知还处于获取`DOM`节点和组件实例层面上，实际上除了这个功能，还有其它小技巧可以使用，这篇文章将详细地介绍`ref`的创建和使用，相关代码会以函数组件为主。

## 创建Ref

### 字符串类型Ref

```jsx
class App extends React.Component {
  render() {
    console.log("this", this);
    return (
      <>
        <div ref="dom">hello world</div>
        <Children ref="children" />
      </>
    );
  }
}

class Children extends React.Component {
  render() {
    return <div>china</div>;
  }
}
```

打印结果

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665491903495-77413556-830a-4768-9631-746ec77e921d.png)

无论是对于真实`DOM`，还是类组件，通过字符串创建的`ref`会绑定在`this.refs`对象上。

### 函数类型Ref

```jsx
class App extends React.Component {
  dom = null
  children = null
  render() {
    componentDidMount() {
      console.log(this.dom, this.childrenDom);
    }
    return (
      <>
        <div ref={(node) => this.dom = node}>hello world</div>
        <Children ref={(node) => this.childrenDom = node} />
      </>
    );
  }
}

class Children extends React.Component {
  render() {
    return <div>china</div>;
  }
}
```

打印结果

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665492064230-47a1871f-4e19-4f61-948e-ab080a3f04ec.png)

### 对象类型Ref

#### createRef创建Ref

创建对象类型的`Ref`对于类组件和函数组件的创建方式是不一样的，在类组件中创建`Ref`对象需要通过`createRef`函数来进行创建，这个函数的实现其实非常简单

```js
function createRef() {
	return {
	    current: null
	}
}
```

从上面的代码来看，就是创建并返回带有`current`属性的对象。

**注意：不要在函数组件里面使用createRef函数**

举个例子来看下为什么在函数组件里面不能使用createRef函数来创建`Ref`

```jsx
function Counter() {
  const ref1 = React.useRef(0);
  const ref2 = React.createRef();
  const [count, setCount] = React.useState(0);

  console.log(ref1, ref2);

  return (
    <>
      <div class="counter-area">
        <div class="count1">count1: {ref1.current}</div>
        <div class="count2">count2: {ref2.current}</div>
      </div>
      <button
        onClick={() => {
          ref1.current = (ref1.current || 0) + 1;
          ref2.current = (ref2.current || 0) + 1;
          setCount(count + 1);
        }}
        >
        点我++
      </button>
    </>
  );
}
```

一起看下打印结果

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665495067159-f62e0957-303f-43b8-b6ed-b029d4a889bc.png)

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665495006336-6d81502a-f260-4976-82f7-9e74f659a854.png)

发现`count2`的右侧一直没有打印，根据控制台里面打印的数据不难发现由`createRef`创建的`ref`的`current`属性一直都为`null`，所以`count2`右侧没有数据。

函数组件更新UI视图实际上会将整个函数重新执行一遍，那么`createRef`函数在组件每次渲染的时候也会重新调用，生成初始状态的`ref`，对应的`current`属性为`null`。

#### useRef创建Ref

`React`提供了内置的`useRef`来生成`ref`对象。

```jsx
function Counter() {
  const ref = React.useRef(null);

  React.useEffect(() => {
    console.log(ref.current);
  }, []);

  return <div ref={ref}>hello world</div>;
}
```

上面有说过在函数组件里面通过`createRef`创建`ref`会有问题，那通过`useRef`这个函数生成的`ref`会不会有上述的问题？

答案是不会，在类组件里面可以把`ref`存储到实例对象上去，但是函数组件并没有实例的说法，但是函数组件有对应的`Fiber`对象，只要组件没有销毁，那么`fiber`对象也不会销毁，将`useRef`产生的`ref`挂载到对应的`fiber`对象上，那么`ref`就不会在函数每次重新执行时被重置掉。

## Ref的高阶用法

### Ref转发

我们在通过Props的方式向组件传递信息时，某些特定的属性是会被`React`底层处理的，我们在组件里面是无法接受到的，例如`key`、`ref`

```jsx
function Counter(props) {
  const { key, ref } = props;
  console.log("props", props);
  return (
    <>
      <div class="key">key: {key}</div>
      <div class="ref">ref: {ref.current}</div>
    </>
  );
}

function App() {
  const ref = useRef(null);
  return <Counter key={"hello"} ref={ref} />;
}
```

控制台中打印信息如下，可以了解到通过`props`的方式传递`key`和`ref`给组件是无效的。

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665844838765-b6fd861d-1737-4d11-b2ab-593f745d8467.png)

那如何传递`ref`和`key`给子组件，既然`react`不允许，那么我们换个身份，暗度陈仓。

```jsx
function Counter(props) {
  const { pkey, pref } = props;
  console.log("props", props);
  return (
    <>
      <div class="key">key: {pkey}</div>
      <div class="ref">ref: {pref.current}</div>
    </>
  );
}

function App() {
  const ref = useRef(null);
  return <Counter pkey={"hello"} pref={ref} />;
}
```

打印信息如下

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665846175574-f16d7fa5-51d4-4f83-bf94-2b3448c9c841.png)

**通过别名的方式可以传递**`**ref**`**，那么为什么还需要**`**forwardRef**`**来传递**`**ref**`**？**

假设这样一种场景，你的组件中需要引用外部库提供的组件，例如`antd`、`fusion`这种组件库提供的组件，而且你需要传递`ref`给这个组件，因为引用的外部组件是不可能知道你会用什么样的别名属性来传递`ref`，所以这个时候只能使用`forwardRef`来进行传递`ref`，因为这个是`react`规定用来传递`ref`值的属性名。

#### 跨层级传递Ref

场景：需要把`ref`从`GrandFather`组件传递到`Son`组件，在`Son`组件里面展示`ref`存储的信息并获取`Son`组件里面的`dom`节点。

```jsx
const Son = (props) => {
  const { msg, grandFatherRef } = props;
  return (
    <>
      <div class="msg">msg: {msg}</div>
      <div class="ref" ref={grandFatherRef}>
        ref: {grandFatherRef.current}
      </div>
    </>
  );
};

const Father = forwardRef((props, ref) => {
  return <Son grandFatherRef={ref} {...props} />;
});

function GrandFather() {
  const ref = useRef("我是来自GrandFather的ref啦~~");
  useEffect(() => console.log(ref.current), []);
  return <Father ref={ref} msg={"我是来自GrandFather的消息啦~~"} />;
}
```

页面展示情况

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665915573353-f5ce133a-ea3d-4538-84d2-d3cddc902b72.png)

控制台打印结果

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665916854810-aa4876af-acb4-4d7a-9262-746e758bf894.png)

上面的代码就是通过别名配合`forwardRef`来转发`ref`，这种转发`ref`的方式，是非常常见的用法。通过别名的方式传递`ref`和通过`forwardRef`传递`ref`的方式其实没有太大的差别，最本质的区别就是通过别名的方式需要传递方和接收方人为地都约定好属性名，而通过`forwardRef`的方式是`react`里面约定了传递的`ref`属性名。

#### 合并转发Ref

我们从父组件传递下去的ref不仅仅可以用来展示获取某个`dom`节点，还可以在子组件里面更改`ref`的信息，获取子组件里面其它信息。

场景：我们需要获取子组件里面input和button这两个`dom`节点

```jsx
const Son = forwardRef((props, ref) => {
  console.log("props", props);
  const sonRef = useRef({});
  useEffect(() => {
    ref.current = {
      ...sonRef.current,
    };
    return () => ref.current = {}
  }, []);
  return (
    <>
      <button ref={(button) => Object.assign(sonRef.current, { button })}>
        点我
      </button>
      <input
        type="text"
        ref={(input) => Object.assign(sonRef.current, { input })}
        />
    </>
  );
});

function Father() {
  const ref = useRef("我是来自Father的ref啦~~");
  useEffect(() => console.log(ref.current), []);
  return <Son ref={ref} msg={"我是来自Father的消息啦~~"} />;
}
```

控制台打印结果

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665917848333-318d440a-a456-413a-a2bd-211a5601bd5e.png)

尽管我们传递下去的`ref`的`current`为字符串属性，但是我们通过在子组件里面修改`current`属性，进而获取到了子组件里面`button`和`input`的`dom`节点

上面的代码其实可以把`useEffect`更改成为`**useImperativeHandle**`，这是`react`提供的`hook`，用来配合`forwardRef`来进行使用，可以。

`**useImperativeHandle**`接收三个参数：

1. 通过`forwardRef`传递过来的`ref`
2. 处理函数，其返回值暴露给父组件的`ref`对象
3. 依赖项

```jsx
const Son = forwardRef((props, ref) => {
  console.log("props", props);
  const sonRef = useRef({});
  useImperativeHandle(
    ref,
    () => ({
      ...sonRef.current,
    }),
    []
  );
  return (
    <>
      <button ref={(button) => Object.assign(sonRef.current, { button })}>
        点我
      </button>
      <input
        type="text"
        ref={(input) => Object.assign(sonRef.current, { input })}
        />
    </>
  );
});

function Father() {
  const ref = useRef("我是来自Father的ref啦~~");
  useEffect(() => console.log(ref.current), []);
  return <Son ref={ref} msg={"我是来自Father的消息啦~~"} />;
}
```

#### 高阶组件转发Ref

在使用高阶组件包裹一个原始组件的时候，因为高阶组件会返回一个新的组件，如果不进行ref转发，从上层组件传递下来的ref会指向这个新的组件

```jsx
function HOC(Comp) {
  function Wrapper(props) {
    const { forwardedRef, ...restProps } = props;
    return <Comp ref={forwardedRef} {...restProps} />;
  }

  return forwardRef((props, ref) => (
    <Wrapper forwardedRef={ref} {...props} />
  ));
}

const Son = forwardRef((props, ref) => {
  return <div ref={ref}>hello world</div>;
});

const NewSon = HOC(Son);

function Father() {
  const ref = useRef("我是来自Father的ref啦~~");
  useEffect(() => console.log(ref.current), []);
  return <NewSon ref={ref} msg={"我是来自Father的消息啦~~"} />;
}
```

注意：第12行处增加`forwardRef`转发是因为函数组件无法直接接收`ref`属性

控制台打印结果

![](https://cdn.nlark.com/yuque/0/2022/png/10382191/1665924298889-43eb96fb-36e9-4ffe-b66e-b850fa1f64bd.png)

### Ref实现组件间的通信

子组件可以通过`props`获取和改变父组件里面的`state`，父组件也可以通过`ref`来获取和改变子组件里面的`state`，实现了父子组件之间的双向通信，这其实在一定程度上打破了react单向数据传递的原则。

```jsx
const Son = forwardRef((props, ref) => {
  const { toFather } = props;
  const [fatherMsg, setFatherMsg] = useState('');
  const [sonMsg, setSonMsg] = useState('');

  useImperativeHandle(
    ref,
    () => ({
      fatherSay: setFatherMsg
    }),
    []
  );

  return (
    <div className="son-box">
      <div className="father-say">父组件对我说: {fatherMsg}</div>
      <div className="son-say">
        我对父组件说:
        <input
          type="text"
          value={sonMsg}
          onChange={(e) => setSonMsg(e.target.value)}
        />
        <button onClick={() => toFather(sonMsg)}>to father</button>
      </div>
    </div>
  );
});

function Father() {
  const [fatherMsg, setFatherMsg] = useState('');
  const [sonMsg, setSonMsg] = useState('');
  const sonRef = useRef(null);

  return (
    <div className="father-box">
      <div className="son-say">子组件对我说: {sonMsg}</div>
      <div className="father-say">
        对子组件说:
        <input
          type="text"
          value={fatherMsg}
          onChange={(e) => setFatherMsg(e.target.value)}
        />
        <button onClick={() => sonRef.current.fatherSay(fatherMsg)}>
          to son
        </button>
      </div>
      <Son ref={sonRef} toFather={setSonMsg} />
    </div>
  );
}
```

## Ref和State的抉择

我们在平常的学习和工作中，在使用函数组件的时候，常常会使用`useState`来生成`state`，每次`state`改变都会引发整个函数组件重新渲染，但是有些`state`的改变不需要更新视图，那么我们可以考虑使用ref来存储，`ref`存储的值在发生变化后是不会引起整个组件的重新渲染的。