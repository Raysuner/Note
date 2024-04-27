## Redux是什么
Redux是一个全局状态管理库，能够帮助开发在编写应用时更加方便的在各个组件间共享状态值。

Redux并不一定需要选用React来作为UI库，它可以和任意UI库来配合使用，甚至不需要任何UI库，直接在html中也是可行的。

这篇文章会从概念出发，讲解Redux中各个概念的作用，并实现相关功能。

## Redux的一些概念
### State
前面说到Redux是一个全局状态管理库，那么就一定有一个变量来保存存储到全局的信息，一般情况下我们把这个状态变量称为State。
### reducer

```jsx
import { useState, createContext, useContext } from "react";

const appContext = createContext(null);

const Input = () => {
  const { state, setState } = useContext(appContext);
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={state.user.name}
        onChange={(e) => {
          state.user.name = e.target.value;
          setState(state);
        }}
      />
    </div>
  );
}

const Observer = () => {
  const { state } = useContext(appContext);
  return (
    <div className="observer" style={{ padding: "20px 0" }}>
      你的名字是: {state.user.name}
    </div>
  );
}

export default function App() {
  const [state, setState] = useState({
    user: {
      name: "",
      age: "",
    },
  });

  return (
    <appContext.Provider value={{ state, setState }}>
      <Input />
      <Observer />
    </appContext.Provider>
  );
}
```
上面是一段看起来很蠢的代码示例，按照编写者的想法来看，拿到输入结果后赋值给name属性，然后调用setState来更新state，看起来很完美。很不巧，在React中更新过程中新值与旧值会进行浅比较，如果相等的话React是不会进行更新的。
下面看看使用reducer后是如何避免这个问题的。
```jsx
function reducer(state, { type, payload }) {
  switch (type) {
    case "update":
      return {
        ...state,
        user: {
          ...state.user,
          name: payload.value,
        },
      };
    default:
      return state;
  }
}

const Input = () => {
  const { state, setState } = useContext(appContext);
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={state.user.name}
        onChange={(e) => {
          // 改动点在这
          setState(
            reducer(state, {
              type: "update",
              payload: { value: e.target.value },
            })
          );
        }}
      />
    </div>
  );
};
```
使用reducer后，编写时只需要传递改变的属性值即可，构造新的state的事情就完全交给了reducer。
这就是reducer的作用，**用来规范获取新的State变量的手段**。
### dispatch
上面使用setState来更新state，可以发现每次都需要调用reducer来构造新的state，我们可以稍微精简下
```jsx
const Input = () => {
  const { state, setState } = useContext(appContext);
  const dispatch = (action) => {
    setState(reducer(state, action));
  };
  
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={state.user.name}
        onChange={(e) => {
        // 得到一定程度的简化
          dispatch({
            type: "update",
            payload: { value: e.target.value },
          });
        }}
      />
    </div>
  );
}
```
这就是dispatch的作用，**用来规范setState的流程**。
### connect
回想一下，我们日常在使用redux的时候，dispatch并不是由我们自己声明的，而是redux内部声明，通过props的方式传递下来的。

在计算机领域有一句话很通用，计算机领域的任何问题都可以通过增加一个间接的中间层来解决问题，这里的问题也可以通过增加一层中间层来解决问题。
```jsx
const connect = (Component) => {
  const Wrapper = (props) => {
    const { state, setState } = useContext(appContext);
    const dispatch = (action) => {
      setState(reducer(state, action));
    };
    return <Component {...props} state={state} dispatch={dispatch} />;
  };
  return Wrapper;
};

const Input = connect(({ state, dispatch }) => {
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={state.user.name}
        onChange={(e) => {
          dispatch({
            type: "update",
            payload: { value: e.target.value },
          });
        }}
      />
    </div>
  );
});
```
上面增加了connect高阶组件，用来充当中间层，在这个中间层定义dispatch函数，然后利用props传递给组件，组件后续就能正常消费了。
**connect用来给组件增加state和dispatch两个props，使组件与全局状态连接起来**，connect这个函数名起的还是比较符合具有实际意义的。
### mapStateToProps
如果全局状态里面存储的对象层级较深，每次在消费的时候都需要`a.b.c.d`这样来写，这样显得太过冗长了。
```jsx
const Input = connect(state => ({user: state.user})) => (({ state, dispatch }) => {
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={user.name}
        onChange={(e) => {
          dispatch({
            type: "update",
            payload: { value: e.target.value },
          });
        }}
      />
    </div>
  );
});
```
如果其它组件也需要展示user里面的字段信息，那么完全可以抽取函数进行复用
```jsx
const extractUser = (state) => ({ user: state.user });

const Input = connect(extractUser)(({ user, dispatch }) => {
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={user.name}
        onChange={(e) => {
          dispatch({
            type: "update",
            payload: { value: e.target.value },
          });
        }}
      />
    </div>
  );
});

const Observer = connect(extractUser)(({ user }) => {
  return (
    <div className="observer" style={{ padding: "20px 0" }}>
      你的名字是: {user.name}
    </div>
  );
});
```
还可以进行再一步的提取
```jsx
const extractUser = (state) => ({ user: state.user });
const connectUser = connect(extractUser);

const Input = connectUser(({ user, dispatch }) => {
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={user.name}
        onChange={(e) => {
          dispatch({
            type: "update",
            payload: { value: e.target.value },
          });
        }}
      />
    </div>
  );
});

const Observer = connectUser(({ user }) => {
  return (
    <div className="observer" style={{ padding: "20px 0" }}>
      你的名字是: {user.name}
    </div>
  );
});
```
将connect函数前面部分的结果先存储下来，后续也可以进行复用，这也是为什么connect会什么是将参数分开放置在两个函数里，而不是把所有参数放置在同一个函数里面。

改造connect函数支持mapStateToProps
```jsx
const connect = (mapStateToProps) => (Component) => {
  const Wrapper = (props) => {
    const { state, setState } = useContext(appContext);
    const newState =
      typeof mapStateToProps === "function" ? mapStateToProps(state) : { state };
    const dispatch = (action) => {
      setState(reducer(state, action));
    };
    return <Component {...props} {...newState} dispatch={dispatch} />;
  };
  return Wrapper;
};
```
这就是mapStateToProps的作用，**用于提前提取全局状态的字段信息**。
### mapDispatchToProps
```jsx
const createFetchUser = (dispatch) => {
  return {
    fetchUser: (payload) => dispatch({ type: "update", payload }),
  };
};

const Input = connect(extractUser, createFetchUser)(({ user, fetchUser }) => {
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={user.name}
        onChange={(e) => {
          fetchUser({ value: e.target.value });
        }}
      />
    </div>
  );
});
```
和mapStateToProps的作用和用途一致，只不过作用的对象是dispatch函数。
对mapStateToProps进行提取以进行复用。
```jsx
const createFetchUser = (dispatch) => {
  return {
    fetchUser: (payload) => dispatch({ type: "update", payload }),
  };
}

const Input = connect(
  extractUser,
  createFetchUser
)(({ user, fetchUser }) => {
  return (
    <div
      className="input-wrapper"
      style={{ padding: "20px 0", borderBottom: "1px solid #222" }}
    >
      <label htmlFor="name">名字: </label>
      <input
        id="name"
        value={user.name}
        onChange={(e) => {
          fetchUser({ value: e.target.value });
        }}
      />
    </div>
  );
});
```
mapDispatch函数就是用来**提前构造更改State的函数**。

现在redux的一些常用概念已经基本
