# 12 【react高级指引（上）】

## 1.setState 扩展

### 1.1 对象式 setState

首先在我们以前的认知中，`setState` 是用来更新状态的，我们一般给它传递一个对象，就像这样

```js
this.setState({
    count: count + 1
})
```

这样每次更新都会让 `count` 的值加 1。这也是我们最常做的东西

这里我们做一个案例，点我加 1，一个按钮一个值，我要在控制台输出每次的 `count` 的值

![image-20221027095114944](https://i0.hdslb.com/bfs/album/b1ad03d5936d35f609b52f853a640e207e6a0048.png)

那我们需要在控制台输出，要如何实现呢？

我们会考虑在 `setState` 更新之后 `log` 一下

```js
add = () => {
    const { count } = this.state
    this.setState({
        count: count + 1
    })
    console.log(this.state.count);
}
```

因此可能会写出这样的代码，看起来很合理，在调用完 `setState` 之后，输出 `count`

![image-20221027095134650](https://i0.hdslb.com/bfs/album/a4746c99b4345fa4e08eec081fcc0d9fb7a7553b.png)

我们发现显示的 `count` 和我们控制台输出的 `count` 值是不一样的

这是因为，我们调用的 `setState` 是同步事件，但是它的作用是让 React 去更新数据，而 React 不会立即的去更新数据，这是一个异步的任务，因此我们输出的 `count` 值会是状态更新之前的数据。“React **状态更新是异步的**”

那我们要如何实现同步呢？

其实在 `setState` 调用的第二个参数，我们可以接收一个函数，这个函数会在状态更新完毕并且界面更新之后调用，我们可以试试

> setState(stateChange, [callback])------对象式的setState
>         1.stateChange为状态改变对象(该对象可以体现出状态的更改)
>         2.callback是可选的回调函数, 它在状态更新完毕、界面也更新后(render调用后)才被调用

```js
add = () => {
    const { count } = this.state
    this.setState(
      {
        count: count + 1,
      },
      () => {
        document.title = `当前值是${this.state.count}`
      },
    )
}
```

我们将 `setState` 填上第二个参数，输出更新后的 `count` 值

![image-20221027173513180](https://i0.hdslb.com/bfs/album/1c7c4bf62e9cbb2e581f9e5ff2552901cde64074.png)

这样我们就能成功的获取到最新的数据了，如果有这个需求我们可以在第二个参数输出噢~

### 1.2 函数式 setState

，函数式的 `setState` 也是接收两个参数

第一个参数是 `updater` ，它是一个能够返回 `stateChange` 对象的函数

第二个参数是一个回调函数，用于在状态更新完毕，界面也更新之后调用

与对象式 `setState` 不同的是，我们传递的第一个参数 `updater` 可以接收到2个参数 `state` 和 `props`

我们尝试一下

> setState(updater, [callback])------函数式的setState
>             1.updater为返回stateChange对象的函数。
>             2.updater可以接收到state和props。
>             4.callback是可选的回调函数, 它在状态更新、界面也更新后(render调用后)才被调用。

```js
add = () => {
    this.setState(
      (state, props) => ({ count: state.count + 1 }),
      () => {
        document.title = `当前值是${this.state.count}`
      },
    )
}
```

![image-20221027173515460](https://i0.hdslb.com/bfs/album/1c7c4bf62e9cbb2e581f9e5ff2552901cde64074.png)

我们也成功的实现了

我们在第一个参数中传入了一个函数，这个函数可以接收到 `state` ，我们通过更新 `state` 中的 `count` 值，来驱动页面的更新

利用函数式 `setState` 的优势还是很不错的，可以直接获得 `state` 和 `props`

> 可以理解为对象式的 `setState` 是函数式 `setState` 的语法糖

### 1.3 总结

```css
(1). setState(stateChange, [callback])------对象式的setState
        1.stateChange为状态改变对象(该对象可以体现出状态的更改)
        2.callback是可选的回调函数, 它在状态更新完毕、界面也更新后(render调用后)才被调用

(2). setState(updater, [callback])------函数式的setState
        1.updater为返回stateChange对象的函数。
        2.updater可以接收到state和props。
        4.callback是可选的回调函数, 它在状态更新、界面也更新后(render调用后)才被调用。
总结:
    1.对象式的setState是函数式的setState的简写方式(语法糖)
    2.使用原则：
            (1).如果新状态不依赖于原状态 ===> 使用对象方式
            (2).如果新状态依赖于原状态 ===> 使用函数方式
            (3).如果需要在setState()执行后获取最新的状态数据, 
                要在第二个callback函数中读取
```


## 2.Context

在React中组件间的数据通信是通过props进行的，父组件给子组件设置props，子组件给后代组件设置props，props在组件间自上向下（父传子）的逐层传递数据。但并不是所有的数据都适合这种传递方式，有些数据需要在多个组件中共同使用，如果还通过props一层一层传递，麻烦自不必多说。

Context为我们提供了一种在不同组件间共享数据的方式，它不再拘泥于props刻板的逐层传递，而是在外层组件中统一设置，设置后内层所有的组件都可以访问到Context中所存储的数据。换句话说，Context类似于JS中的全局作用域，可以将一些公共数据设置到一个同一个Context中，使得所有的组件都可以访问到这些数据。

### 2.1 何时使用 Context

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据，例如当前认证的用户、主题或首选语言。举个例子，在下面的代码中，我们通过一个 “theme” 属性手动调整一个按钮组件的样式：

```js
class A extends React.Component {
  render() {
    return <B theme="dark" />;
  }
}

function B(props) {
  // B 组件接受一个额外的“theme”属性，然后传递给 C 组件。
  // 如果应用中每一个单独的按钮都需要知道 theme 的值，这会是件很麻烦的事，
  // 因为必须将这个值层层传递所有组件。
  return (
    <div>
      <C theme={props.theme} />
    </div>
  );
}

class C extends React.Component {
  render() {
    return h4>我从A组件接收到的主题模式:{this.props.theme}</h4>
  }
}
```

使用 context, 我们可以避免通过中间元素传递 props。

### 2.2 类式组件

当我们想要给子类的子类传递数据时，前面我们讲过了 redux 的做法，这里介绍的 Context 我觉得也类似于 Redux

```js
// React.createContext
const MyContext = React.createContext(defaultValue);
```

创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

**只有**当组件所处的树中没有匹配到 Provider 时，其 `defaultValue` 参数才会生效。此默认值有助于在不使用 Provider 包装组件的情况下对组件进行测试。注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效。

首先我们需要引入一个 `ThemeContext ` 组件，我们需要引用`ThemeContext ` 下的 `Provider`

```js
const ThemeContext  = React.createContext();
const { Provider } = ThemeContext ;
```

`Provider`译为生产者，和Consumer消费者对应。Provider会设置在外层组件中，通过`value`属性来指定Context的值。这个Context值在所有的Provider子组件中都可以访问。Context的搜索流程和JS中函数作用域类似，当我们获取Context时，React会在它的外层查找最近的Provider，然后返回它的Context值。如果没有找到Provider，则会返回Context模块中设置的默认值。

```js
<Provider value={{ theme }}>
    <B />
</Provider>
/* 
每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 value 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 value 值发生变化时，它内部的所有消费组件都会重新渲染。从 Provider 到其内部 consumer 组件（包括 .contextType 和 useContext）的传播不受制于 shouldComponentUpdate 函数，因此当 consumer 组件在其祖先组件跳过更新的情况下也能更新。
*/
```

但是我们需要在使用数据的组件中引入 `ThemeContext `

```js
static contextType = ThemeContext ;
```

在使用时，直接从 `this.context` 上取值即可

```js
const {theme} = this.context
```

**完整版**

```js
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext  = React.createContext('light')

export default class A extends Component {

	state = {theme:'dark'}

	render() {
		const {theme} = this.state
        // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    	// 无论多深，任何组件都能读取这个值。
    	// 在这个例子中，我们将 “dark” 作为当前的值传递下去。
		return (
				<ThemeContext.Provider value={theme}>
					<B/>
				</ThemeContext.Provider>
		)
	}
}

// 中间的组件再也不必指明往下传递 theme 了。
class B extends Component {
	render() {
		return (
				<>
            		<h3>我是B组件</h3>
				   <C/>
            	 </>
		)
	}
}

class C extends Component {
	//声明接收context
    // 指定 contextType 读取当前的 theme context。
    // React 会往上找到最近的 theme Provider，然后使用它的值。
    // 在这个例子中，当前的 theme 值为 “dark”。
	static contextType = ThemeContext 
    
	render() {
		const {theme} = this.context
		return (
				<>
            		<h3>我是C组件</h3>
            		<h4>我从A组件接收到的主题模式:{theme}</h4>
            	 </>
		)
	}
} 
```

> 挂载在 class 上的 `contextType` 属性可以赋值为由 [`React.createContext()`](https://zh-hans.reactjs.org/docs/context.html#reactcreatecontext) 创建的 Context 对象。此属性可以让你使用 `this.context` 来获取最近 Context 上的值。你可以在任何生命周期中访问到它，包括 render 函数中。

### 2.3 函数组件

函数组件和类式组件只有一点点小差别

Context对象中有一个属性叫做Consumer，直译过来为消费者，如果你了解生产消费者模式这里就比较好理解了，如果没接触过，你可以将Consumer理解为数据的获取工具。你可以将它理解为一个特殊的组件，所以你需要这样使用它

```jsx
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext  = React.createContext('light')
 
export default class A extends Component {

	state = {theme:'dark'}

	render() {
		const {theme} = this.state
        // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    	// 无论多深，任何组件都能读取这个值。
    	// 在这个例子中，我们将 “dark” 作为当前的值传递下去。
		return (
				<ThemeContext.Provider value={theme}>
					<B/>
				</ThemeContext.Provider>
		)
	}
}

// 中间的组件再也不必指明往下传递 theme 了。
class B extends Component {
	render() {
		return (
				<>
            		<h3>我是B组件</h3>
				   <C/>
            	 </>
		)
	}
}


function C(){
	return (
        <div>
			<h3>我是C组件</h3>
			<h4>我从A组件接收到的用户名:
			<ThemeContext.Consumer>
				{ctx => {
        			return (<span>ctx</span>)
        		}}
			</ThemeContext.Consumer>
			</h4>
		</div>
	)
}
```

Consumer的标签体必须是一个函数，这个函数会在组件渲染时调用并且将Context中存储的数据作为参数传递进函数，该函数的返回值将会作为组件被最终渲染到页面中。这里我们将参数命名为了ctx，在回调函数中我们就可以通过ctx.xxx访问到Context中的数据。如果需要访问多个Context可以使用多个Consumer嵌套即可。

### 2.4 hook-useContext

通过Consumer使用Context实在是不够优雅，所以React还为我们提供了一个钩子函数`useContext()`，我们只需要将Context对象作为参数传递给钩子函数，它就会直接给我们返回Context对象中存储的数据。

因为我们平时的组件不会写的一个文件中，所以`React.createContext`要单独写在一个文件中

`store/theme-context.js`

```js
import React from "react";

const ThemeContext  = React.createContext('light')

export default ThemeContext;
```

```jsx
import React, {useContext} from 'react';
import ThemeContext from '../store/theme-context';

function C(){
    
    const ctx = useContext(TestContext);

	return (
        <div>
			<h3>我是C组件</h3>
			<h4>我从A组件接收到的用户名:
			<span>{ctx}</span>
			</h4>
		</div>
	)
}
```

## 3.错误边界

### 3.1 基本使用

当不可控因素导致数据不正常时，我们不能直接将报错页面呈现在用户的面前，由于我们没有办法给每一个组件、每一个文件添加判断，来确保正常运行，这样很不现实，因此我们要用到**错误边界**技术

错误边界是一种 `React` 组件，这种组件**可以捕获发生在其子组件树任何位置的** **JavaScript** **错误，并打印这些错误，同时展示降级** **UI**，而并不会渲染那些发生崩溃的子组件树。**错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误**

**错误边界就是让这块组件报错的影响降到最小，不要影响到其他组件或者全局的正常运行**

> 例如 A 组件报错了，我们可以在 A 组件内添加一小段的提示，并把错误控制在 A 组件内，不影响其他组件

- 我们要对容易出错的组件的父组件做手脚，而不是组件本身

我们在父组件中通过 `getDerivedStateFromError` 来配置**子组件**出错时的处理函数

> ###  **编写生命周期函数 getDerivedStateFromError**
>
> 1. 静态函数
> 2. 运行时间点：渲染子组件的过程中，发生错误之后，在更新页面之前
> 3. **注意：只有子组件发生错误，才会运行该函数**
> 4. 该函数返回一个对象，React会将该对象的属性覆盖掉当前组件的state（必须返回 **`null`** 或者**状态对象**（State Obect））
> 5. 参数：错误对象
> 6. 通常，该函数用于改变状态

```js
state={
    hasError:false,
}

static getDerivedStateFromError(error) {
    console.log(error);
    return { hasError: error }
}
```

我们可以将 `hasError` 配置到状态当中，当 `hasError` 状态改变成 `error` 时，表明有错误发生，我们需要在组件中通过判断 `hasError` 值，来指定是否显示子组件

```js
{this.state.hasError ? <h2>Child出错啦</h2> : <Child />}
```

但是我们会发现这个效果过了几秒之后自动又出现报错页面了，那是因为**开发环境还是会报错**，**生产环境不会报错** 直接显示 要显示的文字，白话一些就是这个适用于生产环境，为了生产环境不报错。
开发中我们可以将`Child出错啦`这种错误提示换成一个错误组件。

### 3.2 综合案例

按照React官方的约定，一个类组件定义了**static getDerivedStateFromError()** 或**componentDidCatch()** 这两个生命周期函数中的任意一个（或两个），即可被称作ErrorBoundary组件，实现错误边界的功能。

其中，getDerivedStateFromError方法被约定为渲染备用UI，componentDidCatch方法被约定为捕获打印错误信息。

> ###  **编写生命周期函数 componentDidCatch**
>
> 1. 实例方法
> 2. 运行时间点：渲染子组件的过程中，发生错误，更新页面之后，由于其运行时间点比较靠后，因此不太会在该函数中改变状态
> 3. 通常，该函数用于记录错误消息

```jsx
export class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            hasError: false,
            Error: null,
            ErrorInfo: null
        };
    }
    //控制渲染降级UI
    static getDerivedStateFromError(error,info) {
        return {hasError: error};
    }
    //捕获抛出异常
    componentDidCatch(error, errorInfo) {
         // 1、错误信息（error）
        // 2、错误堆栈（errorInfo)
        //传递异常信息
        this.setState((preState) => 
            ({hasError: preState.hasError, Error: error, ErrorInfo: errorInfo})
        );
                //可以将异常信息抛出给日志系统等等
                //do something....
    }
    render() {
        //如果捕获到异常，渲染降级UI
        if (this.state.hasError) {
            return <div>
                <h1>{`Error:${this.state.Error?.message}`}</h1>
                    {this.state.ErrorInfo?.componentStack}
            </div>;
        }
        return this.props.children;
    }
}
```

> 虽然函数式组件无法定义 `Error Boundary`，但 `Error Boundary` 可以捕获函数式组件的异常错误

实现ErrorBoundary组件后，我们只需要将其当作常规组件使用，将其需要捕获的组件放入其中即可。

使用方式如下：

```js
//main.js
import ReactDOM from 'react-dom/client';
import {ErrorBoundary} from './ErrorBoundary.jsx';
ReactDOM.createRoot(document.getElementById('root')).render(
    <ErrorBoundary>
        <App/>
    </ErrorBoundary>
);
```

```js
//app.js
import React from 'react';
function App() {
    const [count, setCount] = useState(0);
    if (count>0){
        throw new Error('count>0!');
    }
    return (
        <div>
            <button onClick={() => setCount((count) => count + 1)}>
                count is {count}
            </button>
        </div>
    );
}
export default App;
```

点击按钮后即可展示抛出异常时，应该渲染的降级UI：

![image-20221027094444543](https://i0.hdslb.com/bfs/album/3269915e4f64c035ae9e3ce91a8bc1af11881fbd.png)

### 3.3 让子组件不影响父组件正常显示案例

假设B组件（子组件）的出错：users不是一个数组，却是一个字符串。此时，会触发调用`getDerivedStateFromError`，并返回状态数据`{hasError:error}`。A组件（父组件）将根据hasError值判断是渲染备用的错误页面还是B组件。

```jsx
import React, { Component } from 'react'

export default class A extends Component {
  state = { hasError: '' }
  static getDerivedStateFromError(error) {
    return {
      hasError: error,
    }
  }

  componentDidCatch(error, info) {
    console.log('error:', error)
    console.log('info:', info)
    console.log('用于统计错误信息并反馈给后台,将通知开发人员进行bug修复')
  }

  render() {
    const { hasError } = this.state
    return (
      <div className="a">
        <div>我是组件A</div>
        {hasError ? '当前网络不稳定,请稍候再试!' : <B />}
      </div>
    )
  }
}

class B extends Component {
  state = {
    users: '',
  }
  render() {
    const { users } = this.state
    return (
      <div className="b">
        <div>我是组件B</div>
        {users.map(userObj => (
          <li key={userObj.id}>
            {userObj.name}，{userObj.age}
          </li>
        ))}
      </div>
    )
  }
}
```

<img src="https://i0.hdslb.com/bfs/album/b79f78aeb57461debfea87d644ea5c8e4baec625.png" alt="image-20221027190233518"  />

### 3.4 使用错误边界需要注意什么

没有什么技术栈或者技术思维是银弹，错误边界看起来用得很爽，但是需要注意以下几点：

- 错误边界实际上是用来捕获render阶段时抛出的异常，而React事件处理器中的错误并不会渲染过程中被触发，所以**错误边界捕获不到事件处理器中的错误**。
- React官方推荐使用try/catch来自行处理事件处理器中的异常。
- 错误边界无法捕获异步代码中的错误（例如 `setTimeout`或 `requestAnimationFrame`回调函数），这两个函数中的代码通常不在当前任务队列内执行。
- 目前错误边界只能在类组件中实现，也只能捕获**其子组件树**的错误信息。错误边界无法捕获自身的错误，如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，类似于JavaScript中的cantch的工作机制。
- 错误边界无法在服务端渲染中生效，因为根本的渲染方法已经`ReactDOM.createRoot().render()`修改为了`ReactDOM.hydrateRoot()`， 而上面也提到了，错误边界捕获的是render阶段时抛出的异常。

**总结：仅处理渲染子组件期间的同步错误**

## 4.路由组件的lazyLoad

懒加载在 React 中用的最多的就是路由组件了，页面刷新时，所有的页面都会重新加载，这并不是我们想要的，我们想要实现点击哪个路由链接再加载即可，这样避免了不必要的加载

![image-20221027095740307](https://i0.hdslb.com/bfs/album/ab202fe40dbd0d4437efa829614f5f366d1da52b.png)

我们可以发现，我们页面一加载时，所有的路由组件都会被加载

如果我们有 100 个路由组件，但是用户只点击了几个，这就会有很大的消耗，因此我们需要做懒加载处理，**我们点击哪个时，才去加载哪一个**

首先我们需要从 `react` 库中暴露一个 `lazy` 函数

> `React.lazy()` 允许你定义一个动态加载的组件。这有助于缩减 bundle 的体积，并延迟加载在初次渲染时未用到的组件。

```js
import React, { Component ,lazy} from 'react';
```

然后我们需要更改引入组件的方式

```js
// 这个组件是动态加载的
const Home = lazy(() => import('./Home'))
const About = lazy(() => import('./About'))
```

采用 `lazy` 函数包裹

![image-20221027095800440](https://i0.hdslb.com/bfs/album/956ae433c7a0f17a76ea7e5665e8f2e94ab3b925.png)

我们会遇到这样的错误，提示我们用一个标签包裹

这里是因为，当我们网速慢的时候，路由组件就会有可能加载不出来，页面就会白屏，它需要我们来指定一个路由组件加载的东西，相对于 loading

> `React.Suspense` 可以指定加载指示器（loading indicator），以防其组件树中的某些子组件尚未具备渲染条件。在未来，我们计划让 `Suspense` 处理更多的场景，如数据获取等。你可以在 [我们的路线图](https://zh-hans.reactjs.org/blog/2018/11/27/react-16-roadmap.html) 了解这一点。

```jsx
<Suspense fallback={<h1>loading</h1>}>
     <Routes>
        <Route path="/home" component={Home}></Route>
        <Route path="/about" component={About}></Route>
	</Routes>
</Suspense>
```

初次登录页面的时候

![image-20221027100147592](https://i0.hdslb.com/bfs/album/97dbf644b51f4a7e39ea6b28f99dc498ec08ecb1.png)

注意噢，这些文件都不是路由组件，当我们点击了对应组件之后才会加载

![68747470733a2f2f6c6a63696d672e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f696d672f72656163742d657874656e73696f6e2d6c617a796c6f61642d332e676966](https://i0.hdslb.com/bfs/album/b03697901855aaabe2ca5aab01a20dfcbead7cd5.gif)

从上图我们可以看出，每次点击时，才会去请求 `chunk` 文件

那我们更改写的 `fallback` 有什么用呢？它会在页面还没有加载出来的时候显示

> 注意：因为 loading 是作为一个兜底的存在，因此 loading 是 必须提前引入的，不能懒加载

## 5.Fragment

我们编写组件的时候每次都需要采用一个 `div` 标签包裹，才能让它正常的编译，但是这样会引发什么问题呢？我们打开控制台看看它的层级

![image-20221027100328758](https://i0.hdslb.com/bfs/album/2971587bd1dbd9d07893218a1c5a86381692b3bf.png)

它包裹了几层无意义的 div 标签，我们可以采用 `Fragment` 来解决这个问题

React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点。

首先，我们需要从 react 中暴露出 `Fragment` ，将我们所写的内容采用 `Fragment` 标签进行包裹，当它解析到 `Fragment` 标签的时候，就会把它去掉

这样我们的内容就直接挂在了 `root` 标签下

```jsx
render() {
  return (
    <React.Fragment 可选 key={xxx.id}>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

在React中为我们提供了一种更加便捷的方式，直接使用`<></>`代替Fragment更加简单：

> 同时采用空标签，也能实现，但是它不能接收任何值，而 `Fragment` 能够接收 1 个值`key`

```js
render() {
  return (
        <>
          <ChildA />
          <ChildB />
          <ChildC />
        </>
  );
}
```

## 6.使用 PropTypes 进行类型检查

**已在 `02 【面向组件编程】`中 `3.props `进行说明**


# 13【react高级指引（下）】

## 1.组件优化

### 1.1 shouldComponentUpdate 优化

在我们之前一直写的代码中，我们一直使用的`Component` 是有问题存在的

1. 只要执行 `setState` ，即使不改变状态数据，组件也会调用 `render`
2. 当前组件状态更新，也会引起子组件 `render`

而我们想要的是只有组件的 `state` 或者 `props` 数据发生改变的时候，再调用 `render`

我们可以采用重写 `shouldComponentUpdate` 的方法，但是这个方法不能根治这个问题，当状态很多时，我们没有办法增加判断

看个案例来了解下原理：

如果你的组件只有当 `props.color` 或者 `state.count` 的值改变才需要更新时，你可以使用 `shouldComponentUpdate` 来进行检查：

````jsx
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
````

在这段代码中，`shouldComponentUpdate` 仅检查了 `props.color` 或 `state.count` 是否改变。如果这些值没有改变，那么这个组件不会更新。如果你的组件更复杂一些，你可以使用类似“浅比较”的模式来检查 `props` 和 `state` 中所有的字段，以此来决定是否组件需要更新。React 已经提供了一位好帮手来帮你实现这种常见的模式 - 你只要继承 `React.PureComponent` 就行了。

### 1.2 PureComponent 优化

这段代码可以改成以下这种更简洁的形式：

```jsx
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
```

大部分情况下，你可以使用 `React.PureComponent` 来代替手写 `shouldComponentUpdate`。但它只进行浅比较，所以当 props 或者 state 某种程度是可变的话，浅比较会有遗漏，那你就不能使用它了。当数据结构很复杂时，情况会变得麻烦。

> `PureComponent` 会对比当前对象和下一个状态的 `prop` 和 `state` ，而这个比较属于浅比较，比较基本数据类型是否相同，而对于引用数据类型，**比较的是它的引用地址是否相同，这个比较与内容无关**

```js
state = {stus:['小张','小李','小王']}

addStu = ()=>{
    /* const {stus} = this.state
    stus.unshift('小刘')
    this.setState({stus}) */

    const {stus} = this.state
    this.setState({stus:['小刘',...stus]})
}
```

注释掉的那部分，我们是用`unshift`方法为`stus`数组添加了一项，它本身的地址是不变的，这样的话会被当做没有产生变化(因为引用数据类型比较的是地址)，所以我们平时都是采用合并数组的方式去更新数组。

### 1.3 案例

```jsx
import React, { PureComponent } from 'react'
import "./index.css";

export default class A extends PureComponent {
  state = {
    username:"张三"
  }

  handleClick = () => {
    this.setState({})
  }

  render() {
    console.log("A:enter render()")
    const {username} = this.state;
    const {handleClick} = this;

    return (
      <div className="a">
        <div>我是组件A</div>
        <span>我的username是{username}</span>&nbsp;&nbsp;
        <button onClick={handleClick}>执行setState且不改变状态数据</button>
        <B/>
      </div>
    )
  }
}

class B extends PureComponent{
  render(){
    console.log("B:enter render()")
    return (
      <div className="b">
        <div>我是组件B</div>
      </div>
    )
  }
}

```

点击按钮后不会有任何变化，render函数也没有调用

![image-20221027191454468](https://i0.hdslb.com/bfs/album/fb16728a87c04da136da2a965d4980bd70580234.png)

修改代码

```js
handleClick = () => {
    this.setState({
      username: '李四',
    })
}
```

点击按钮后只有`A`组件的`render`函数会调用

![image-20221027192124322](https://i0.hdslb.com/bfs/album/f0022ed007d420efc7b284799314e5f2bfa944db.png)

修改代码

```js
handleClick = () => {
    const { state } = this
    state.username = '李四'
    this.setState(state)
}
```

![image-20221027192253591](https://i0.hdslb.com/bfs/album/8cb94c97d7c461cc870ad0e5cb3a1e370d33f95c.png)

点击后不会有任何变化，`render`函数没有调用，这个时候其实是`shouldComponentUpdate`返回的`false`。

## 2.Render Props

**如何向组件内部动态传入带内容的结构(标签)?**

```css
Vue中: 
	使用slot技术, 也就是通过组件标签体传入结构  <AA><BB/></AA>
React中:
	使用children props: 通过组件标签体传入结构
	使用render props: 通过组件标签属性传入结构, 一般用render函数属性
```

**children props**

```jsx
render() {
    return (
            <A>
              <B>xxxx</B>
            </A>
    )
}


问题: 如果B组件需要A组件内的数据, ==> 做不到 
```

术语 [“render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术

采用 render props 技术，我们可以像组件内部动态传入带有内容的结构

> 当我们在一个组件标签中填写内容时，这个内容会被定义为 children props，我们可以通过 `this.props.children` 来获取

例如：

```html
<A>hello</A>
```

这个 hello 我们就可以通过 children 来获取

而我们所说的 render props 就是在组件标签中传入一个 render 方法(名字可以自己定义，这个名字更语义化)，又因为属于 props ，因而被叫做了 render props

```jsx
<A render={(name) => <B name={name} />} />
A组件: {this.props.render(内部state数据)}
B组件: 读取A组件传入的数据显示 {this.props.data} 
```

你可以把 `render` 看作是 `props`，只是它有特殊作用，当然它也可以用其他名字来命名

在上面的代码中，我们需要在 A 组件中预留出 B 组件渲染的位置 在需要的位置上加上`{this.props.render(name)}`

那我们在 B 组件中，如何接收 A 组件传递的 `name` 值呢？通过 `this.props.name` 的方式

```jsx
export default class Parent extends Component {
	render() {
		return (
			<div className="parent">
				<h3>我是Parent组件</h3>
				<A render={ name => (<B name={name}/>) }/>
			</div>
		)
	}
}

class A extends Component {
	state = {name:'tom'}
	render() {
		console.log(this.props);
		const {name} = this.state
		return (
			<div className="a">
				<h3>我是A组件</h3>
				{this.props.render(name)}
			</div>
		)
	}
}

class B extends Component {
	render() {
		console.log('B--render');
		return (
			<div className="b">
				<h3>我是B组件,{this.props.name}</h3>
			</div>
		)
	}
}

```

## 3.Portal

Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。

> 李立超老师的博客
>
> 这篇博客对于Portal的引出我觉得写的很好
>
> [portal – 李立超 | lilichao.com](https://www.lilichao.com/index.php/2022/03/22/portal/)

### 3.1 问题的引出

在React中，父组件引入子组件后，子组件会直接在父组件内部渲染。换句话说，React元素中的子组件，在DOM中，也会是其父组件对应DOM的后代元素。

但是，在有些场景下如果将子组件直接渲染为父组件的后代，在网页显示时会出现一些问题。比如，需要在React中添加一个会盖住其他元素的Backdrop组件，Backdrop显示后，页面中所有的元素都会被遮盖。很显然这里需要用到定位，但是如果将遮罩层直接在当前组件中渲染的话，遮罩层会成为当前组件的后代元素。如果此时，当前元素后边的兄弟元素中有开启定位的情况出现，且层级不低于当前元素时，便会出现盖住遮罩层的情况。

```jsx
const Backdrop = () => {
    
  return <div
           style={
                  {
                    position:'fixed',
                    top:0,
                    bottom:0,
                    left:0,
                    right:0,
                    background:'rgba(0,0,0,.3)',
                    zIndex:9999
                  }
                }
           >
  </div>
};

const Box = props => {
    
  return <div
          style={
              {
                width:100,
                height:100,
                background:props.bgColor
              }
            }
           >
             {props.children}
           </div>
};

const App = () => {
    
  return (
  		<div>
            <Box bgColor='yellowgreen'>
            <Backdrop/>
            </Box>
            <Box bgColor='orange' />
  		</div>;
  )
};
```

上例代码中，App组件中引入了两个Box组件，一个绿色，一个橙色。绿色组件中引入了Backdrop组件，Backdrop组件是一个遮罩层，可以在覆盖住整个网页。

现在三个组件的关系是，绿色Box是橙色Box的兄弟元素，Backdrop是绿色Box的子元素。如果Box组件没有开启定位，遮罩层可以正常显示覆盖整个页面。

![image-20221029232123087](https://i0.hdslb.com/bfs/album/24f29c3777968ed6c976c7144938653880f7b9fc.png)

Backdrop能够盖住页面

但是如果为Box开启定位，并设置层级会出现什么情况呢？

```jsx
const Box = props => {
    
  return <div
           style={
                  {
                    width:100,
                    height:100,
                    background:props.bgColor,
                    position:'relative',
                    zIndex:1
                  }
                }
            >
             {props.children}
           </div>
};
```

现在修改Box组件，开启相对定位，并设置了z-index为1，结果页面变成了这个样子：

![image-20221029232209420](https://i0.hdslb.com/bfs/album/949222a13cb1e7048d3e5982e8631431b3922ab2.png)

和上图对比，显然橙色的box没有被盖住，这是为什么呢？首先我们来看看他们的结构：

```jsx
<App>
    <绿色Box>
            <遮罩/>
    </绿色Box>
    <橙色Box/>
</App>
```

绿色Box和橙色Box都开启了定位，且z-index相同都为1，但是由于橙色在后边，所以实际层级是高于绿色的。由于绿色是遮罩层的父元素，所以即使遮罩的层级是9999也依然盖不住橙色。

问题出在了哪？遮罩层的作用，是用来盖住其他元素的，它本就不该作为Box的子元素出现，作为子元素了，就难免会出现类似问题。所以我们需要在Box中使用遮罩，但是又不能使他成为Box的子元素。怎么办呢？React为我们提供了一个“传送门”可以将元素传送到指定的位置上。

通过ReactDOM中的createPortal()方法，可以在渲染元素时将元素渲染到网页中的指定位置。这个方法就和他的名字一样，给React元素开启了一个传送门，让它可以去到它应该去的地方。

### 3.2 Portal的用法

1. 在index.html中添加一个新的元素
2. 在组件中中通过ReactDOM.createPortal()将元素渲染到新建的元素中

在index.html中添加新元素：

```html
<div id="backdrop"></div>
```

修改Backdrop组件：

```jsx
const backdropDOM = document.getElementById('backdrop');

const Backdrop = () => {
  return ReactDOM.createPortal(
  		<div
           style={
                  {
                    position:'fixed',
                    top:0,
                    bottom:0,
                    left:0,
                    right:0,
                    zIndex:9999,
                    background:'rgba(0,0,0,.3)'
                  }
                }
           >
 		 </div>,
      backdropDOM
  );
};
```

如此一来，我们虽然是在Box中引入了Backdrop，但是由于在Backdrop中开启了“传送门”，Backdrop就会直接渲染到网页中id为backdrop的div中，这样一来上边的问题就解决了

### 3.3 通过 Portal 进行事件冒泡

尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 *React 树*， 且与 *DOM 树* 中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。

这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 *React 树*的祖先，即便这些元素并不是 *DOM 树* 中的祖先。假设存在如下 HTML 结构：

```html
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

在 `#app-root` 里的 `Parent` 组件能够捕获到未被捕获的从兄弟节点 `#modal-root` 冒泡上来的事件。

```jsx
// 在 DOM 中有两个容器是兄弟级 （siblings）
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // 在 Modal 的所有子元素被挂载后，
    // 这个 portal 元素会被嵌入到 DOM 树中，
    // 这意味着子元素将被挂载到一个分离的 DOM 节点中。
    // 如果要求子组件在挂载时可以立刻接入 DOM 树，
    // 例如衡量一个 DOM 节点，
    // 或者在后代节点中使用 ‘autoFocus’，
    // 则需添加 state 到 Modal 中，
    // 仅当 Modal 被插入 DOM 树中才能渲染子元素。
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el
    );
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // 当子元素里的按钮被点击时，
    // 这个将会被触发更新父元素的 state，
    // 即使这个按钮在 DOM 中不是直接关联的后代
    this.setState(state => ({
      clicks: state.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // 这个按钮的点击事件会冒泡到父元素
  // 因为这里没有定义 'onClick' 属性
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}

const root = ReactDOM.createRoot(appRoot);
root.render(<Parent />);
```

![image-20221029233009114](https://i0.hdslb.com/bfs/album/05f650b10fafb40e1a7a4d1ac26072afa414afa2.png)

点击click后,可以发现数字从0变成1了

![image-20221029233124852](https://i0.hdslb.com/bfs/album/a64a35bc5b691c67fc361986e4770c75fd9bf4df.png)

子组件`Child`的点击事件能冒泡到父组件`Parent `,触发父元素的点击事件

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/jGBWpE)

在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 `<Modal />` 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件。


# 14【react-Hook （上）】

*Hook* 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

## 1.准备

### 1.1  什么是 Hook 

Hooks 译为钩子，Hooks 就是在函数组件内，负责钩进外部功能的函数。

React 提供了一些常用钩子，React 也支持自定义钩子，这些钩子都是用于为函数引入外部功能。

当我们在组件中，需要引入外部功能时，就可以使用 React 提供的钩子，或者自定义钩子。

### 1.2 动机

**在组件之间复用状态逻辑很难**
React 没有提供将可复用性行为“附加”到组件的途径（例如，把组件连接到 store）。
你可以使用 Hook 从组件中提取状态逻辑，使得这些逻辑可以单独测试并复用。Hook 使你在无需修改组件结构的情况下复用状态逻辑。 这使得在组件间或社区内共享 Hook 变得更便捷。
**复杂组件变得难以理解**
我们经常维护一些组件，组件起初很简单，但是逐渐会被状态逻辑和副作用充斥。每个生命周期常常包含一些不相关的逻辑。
Hook 将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据），而并非强制按照生命周期划分。你还可以使用 reducer 来管理组件的内部状态，使其更加可预测。

**难以理解的 class**
你必须去理解 JavaScript 中 this 的工作方式，这与其他语言存在巨大差异。还不能忘记绑定事件处理器。
class 不能很好的压缩，并且会使热重载出现不稳定的情况。因此，我们想提供一个使代码更易于优化的 API。
为了解决这些问题，Hook 使你在非 class 的情况下可以使用更多的 React 特性。 从概念上讲，React 组件一直更像是函数。而 Hook 则拥抱了函数，同时也没有牺牲 React 的精神原则。Hook 提供了问题的解决方案，无需学习复杂的函数式或响应式编程技术。

### 1.3 Hook API

- [基础 Hook](https://zh-hans.reactjs.org/docs/hooks-reference.html#basic-hooks)
  - [`useState`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate)
  - [`useEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useeffect)
  - [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)
- [额外的 Hook](https://zh-hans.reactjs.org/docs/hooks-reference.html#additional-hooks)
  - [`useReducer`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer)
  - [`useCallback`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback)
  - [`useMemo`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)
  - [`useRef`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useref)
  - [`useImperativeHandle`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle)
  - [`useLayoutEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect)
  - [`useDebugValue`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedebugvalue)
  - [`useDeferredValue`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedeferredvalue)
  - [`useTransition`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usetransition)
  - [`useId`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useid)
- [Library Hooks](https://zh-hans.reactjs.org/docs/hooks-reference.html#library-hooks)
  - [`useSyncExternalStore`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usesyncexternalstore)
  - [`useInsertionEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useinsertioneffect)

### 1.4 什么时候会用 Hook

如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其转化为 class。现在你可以在现有的函数组件中使用 Hook。

**注意：**

在组件中有些特殊的规则，规定什么地方能使用 Hook，什么地方不能使用。我们将在 [Hook 规则](https://zh-hans.reactjs.org/docs/hooks-rules.html)中学习它们。

## 2.使用 State Hook

### 2.1 声明 State 变量

首先我们需要明确一点，函数式组件**没有**自己的 `this`

在 class 中，我们通过在构造函数中设置 `this.state` 为 `{ count: 0 }` 来初始化 `count` state 为 `0`：

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
```

在函数组件中，我们没有 `this`，所以我们不能分配或读取 `this.state`。我们直接在组件中调用 `useState` Hook：

```js
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 “count” 的 state 变量
  const [count, setCount] = useState(0);
  console.log(count, setCount)
}
```

![image-20221027201435122](https://i0.hdslb.com/bfs/album/3a8c62e2e92fc6adc7ae1e70cb0f39c18de3bb76.png)

**`调用 useState` 方法的时候做了什么?** 它定义一个 “state 变量”。我们的变量叫 `count`， 但是我们可以叫他任何名字，比如 `banana`。这是一种在函数调用时保存变量的方式 —— `useState` 是一种新方法，它与 class 里面的 `this.state` 提供的功能完全相同。一般来说，在函数退出后变量就会”消失”，而 state 中的变量会被 React 保留。

**`useState` 需要哪些参数？** `useState()` 方法里面唯一的参数就是初始 state。不同于 class 的是，我们可以按照需要使用数字或字符串对其进行赋值，而不一定是对象。在示例中，只需使用数字来记录用户点击次数，所以我们传了 `0` 作为变量的初始 state。（如果我们想要在 state 中存储两个不同的变量，只需调用 `useState()` 两次即可。）

**`useState` 方法的返回值是什么？** 返回值为：当前 state 以及更新 state 的函数。这就是我们写 `const [count, setCount] = useState()` 的原因。这与 class 里面 `this.state.count` 和 `this.setState` 类似，唯一区别就是你需要成对的获取它们。如果你不熟悉我们使用的语法，我们会在[本章节的底部](https://zh-hans.reactjs.org/docs/hooks-state.html#tip-what-do-square-brackets-mean)介绍它。

> 简单说
>
> 它让函数式组件能够维护自己的 `state` ，它**接收一个参数**，作为**初始化** `state` 的值，赋值给 `count`，因此 `useState` 的初始值只有**第一次有效**，它所映射出的两个变量 `count` 和 `setCount` 我们可以理解为 `setState` 来使用
>
> **useState 能够返回一个数组，第一个元素是 state ，第二个是更新 state 的函数**

既然我们知道了 `useState` 的作用，我们的示例应该更容易理解了：

我们声明了一个叫 `count` 的 state 变量，然后把它设为 `0`。React 会在重复渲染时记住它当前的值，并且提供最新的值给我们的函数。我们可以通过调用 `setCount` 来更新当前的 `count`。

> 注意
>
> 你可能想知道：为什么叫 `useState` 而不叫 `createState`?
>
> “Create” 可能不是很准确，因为 state 只在组件首次渲染的时候被创建。在下一次重新渲染时，`useState` 返回给我们当前的 state。否则它就不是 “state”了！这也是 Hook 的名字*总是*以 `use` 开头的一个原因。我们将在后面的 [Hook 规则](https://zh-hans.reactjs.org/docs/hooks-rules.html)中了解原因。

### 2.2 读取 State

当我们想在 class 中显示当前的 count，我们读取 `this.state.count`：

```html
  <p>You clicked {this.state.count} times</p>
```

在函数中，我们可以直接用 `count`:

```html
<p>You clicked {count} times</p>
```

### 2.3 更新 State

在 class 中，我们需要调用 `this.setState()` 来更新 `count` 值：

```jsx
  <button onClick={() => this.setState({ count: this.state.count + 1 })}>
    Click me
  </button>
```

在函数中，我们已经有了 `setCount` 和 `count` 变量，所以我们不需要 `this`:

```jsx
<button onClick={() => setCount(count + 1)}>
    Click me
 </button>
```

### 2.4 使用多个 state 变量

将 state 变量声明为一对 `[something, setSomething]` 也很方便，因为如果我们想使用多个 state 变量，它允许我们给不同的 state 变量取不同的名称：

```js
function ExampleWithManyStates() {
  // 声明多个 state 变量
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: '学习 Hook' }]);
```

在以上组件中，我们有局部变量 `age`，`fruit` 和 `todos`，并且我们可以单独更新它们：

```js
  function handleOrangeClick() {
    // 和 this.setState({ fruit: 'orange' }) 类似
    setFruit('orange');
  }
```

你**不必**使用多个 state 变量。State 变量可以很好地存储对象和数组，因此，你仍然可以将相关数据分为一组。然而，不像 class 中的 `this.setState`，更新 state 变量总是*替换*它而不是合并它。

### 2.5 总结

现在让我们来**仔细回顾一下学到的知识**，看下我们是否真正理解了。

```js
 1:  import React, { useState } from 'react';
 2:
 3:  function Example() {
 4:    const [count, setCount] = useState(0);
 5:
 6:    return (
 7:      <div>
 8:        <p>You clicked {count} times</p>
 9:        <button onClick={() => setCount(count + 1)}>
10:         Click me
11:        </button>
12:      </div>
13:    );
14:  }
```

- **第一行:** 引入 React 中的 `useState` Hook。它让我们在函数组件中存储内部 state。
- **第四行:** 在 `Example` 组件内部，我们通过调用 `useState` Hook 声明了一个新的 state 变量。它返回一对值给到我们命名的变量上。我们把变量命名为 `count`，因为它存储的是点击次数。我们通过传 `0` 作为 `useState` 唯一的参数来将其初始化为 `0`。第二个返回的值本身就是一个函数。它让我们可以更新 `count` 的值，所以我们叫它 `setCount`。
- **第九行:** 当用户点击按钮后，我们传递一个新的值给 `setCount`。React 会重新渲染 `Example` 组件，并把最新的 `count` 传给它。

乍一看这似乎有点太多了。不要急于求成！如果你有不理解的地方，请再次查看以上代码并从头到尾阅读。我们保证一旦你试着”忘记” class 里面 state 是如何工作的，并用新的眼光看这段代码，就容易理解了。

## 3.使用 Effect Hook

### 3.1 副作用

React组件有部分逻辑都可以直接编写到组件的函数体中的，像是对数组调用filter、map等方法，像是判断某个组件是否显示等。但是有一部分逻辑如果直接写在函数体中，会影响到组件的渲染，这部分会产生“副作用”的代码，是一定不能直接写在函数体中。

例如，如果直接将修改state的逻辑编写到了组件之中，就会导致组件不断的循环渲染，直至调用次数过多内存溢出。

### 3.2 React.StrictMode

编写React组件时，我们要极力的避免组件中出现那些会产生“副作用”的代码。同时，如果你的React使用了严格模式，也就是在React中使用了`React.StrictMode`标签，那么React会非常“智能”的去检查你的组件中是否写有副作用的代码，当然这个智能是加了引号的。

React并不能自动替你发现副作用，但是它会想办法让它显现出来，从而让你发现它。那么它是怎么让你发现副作用的呢？React的严格模式，在处于开发模式下，会主动的重复调用一些函数，以使副作用显现。所以在处于开发模式且开启了React严格模式时，这些函数会被调用两次：

类组件的的 `constructor`, `render`, 和 `shouldComponentUpdate` 方法
类组件的静态方法 `getDerivedStateFromProps`
函数组件的函数体
参数为函数的`setState`
参数为函数的`useState`, `useMemo`, or `useReducer`

重复的调用会使副作用更容易凸显出来，你可以尝试着在函数组件的函数体中调用一个`console.log`你会发现它会执行两次，如果你的浏览器中安装了React Developer Tools，第二次调用会显示为灰色。

如果你无法通过浏览器正常安装[React Developer Tools](https://my-wp.oss-cn-beijing.aliyuncs.com/wp-content/uploads/2022/05/20220512111133423.zip)可以通过点击这里下载。

### 3.3 Effect 基本使用

在类式组件中，提供了一些声明周期钩子给我们使用，我们可以在组件的特殊时期执行特定的事情，例如 `componentDidMount` ，能够在组件挂载完成后执行一些东西

在函数式组件中也可以实现，它采用的是 `Effect Hook` ，它的语法更加的简单，同时融合了 `componentDidUpdata` 生命周期，极大的方便了我们的开发

`Effect Hook` 可以让你在函数组件中执行副作用操作，专门用来处理那些不能直接写在组件内部的代码。

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 类似于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用这个bom api更新网页标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

我们为计数器增加了一个小功能：将 document 的 title 设置为包含了点击次数的消息。

`useEffect()`中的回调函数会在组件每次渲染完毕之后执行，这也是它和写在函数体中代码的最大的不同，函数体中的代码会在组件渲染前执行，而`useEffect()`中的代码是在组件渲染后才执行，这就避免了代码的执行影响到组件渲染。

通过使用这个Hook，我设置了React组件在渲染后所要执行的操作。React会将我们传递的函数保存（我们称这个函数为effect），并且在DOM更新后执行调用它。React会确保effect每次运行时，DOM都已经更新完毕。

> 提示
>
> 如果你熟悉 React class 的生命周期函数，你可以把 `useEffect` Hook 看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。

在 React 组件中有两种常见副作用操作：需要清除的和不需要清除的。我们来更仔细地看一下他们之间的区别。

### 3.2 无需清除的 effect

有时候，我们只想**在 React 更新 DOM 之后运行一些额外的代码。**比如发送网络请求，手动变更 DOM，记录日志，这些都是常见的无需清除的操作。因为我们在执行完这些操作之后，就可以忽略他们了。让我们对比一下使用 class 和 Hook 都是怎么实现这些副作用的。

#### 3.2.1 使用 class 的示例

在 React 的 class 组件中，`render` 函数是不应该有任何副作用的。一般来说，在这里执行操作太早了，我们基本上都希望在 React 更新 DOM 之后才执行我们的操作。

这就是为什么在 React class 中，我们把副作用操作放到 `componentDidMount` 和 `componentDidUpdate` 函数中。回到示例中，这是一个 React 计数器的 class 组件。它在 React 对 DOM 进行操作之后，立即更新了 document 的 title 属性

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
    
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

注意，**在这个 class 中，我们需要在两个生命周期函数中编写重复的代码。**

这是因为很多情况下，我们希望在组件加载和更新时执行同样的操作。从概念上说，我们希望它在每次渲染之后执行 —— 但 React 的 class 组件没有提供这样的方法。即使我们提取出一个方法，我们还是要在两个地方调用它。

现在让我们来看看如何使用 `useEffect` 执行相同的操作。

#### 3.2.2 使用 Hook 的示例

我们在本章节开始时已经看到了这个示例，但让我们再仔细观察它：

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**`useEffect` 做了什么？** 通过使用这个 Hook，你可以告诉 React 组件需要在渲染后执行某些操作。React 会保存你传递的函数（我们将它称之为 “effect”），并且在执行 DOM 更新之后调用它。在这个 effect 中，我们设置了 document 的 title 属性，不过我们也可以执行数据获取或调用其他命令式的 API。

**为什么在组件内部调用 `useEffect`？** 将 `useEffect` 放在组件内部让我们可以在 effect 中直接访问 `count` state 变量（或其他 props）。我们不需要特殊的 API 来读取它 —— 它已经保存在函数作用域中。Hook 使用了 JavaScript 的闭包机制，而不用在 JavaScript 已经提供了解决方案的情况下，还引入特定的 React API。

**`useEffect` 会在每次渲染后都执行吗？** 是的，默认情况下，它在第一次渲染之后*和*每次更新之后都会执行。（我们稍后会谈到[如何控制它](https://zh-hans.reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects)。）你可能会更容易接受 effect 发生在“渲染之后”这种概念，不用再去考虑“挂载”还是“更新”。React 保证了每次运行 effect 的同时，DOM 都已经更新完毕。

#### 3.2.3 详细说明

现在我们已经对 effect 有了大致了解，下面这些代码应该不难看懂了：

```js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
}
```

我们声明了 `count` state 变量，并告诉 React 我们需要使用 effect。紧接着传递函数给 `useEffect` Hook。此函数就是我们的 effect。然后使用 `document.title` 浏览器 API 设置 document 的 title。我们可以在 effect 中获取到最新的 `count` 值，因为他在函数的作用域内。当 React 渲染组件时，会保存已使用的 effect，并在更新完 DOM 后执行它。这个过程在每次渲染时都会发生，包括首次渲染。

经验丰富的 JavaScript 开发人员可能会注意到，传递给 `useEffect` 的函数在每次渲染中都会有所不同，这是刻意为之的。事实上这正是我们可以在 effect 中获取最新的 `count` 的值，而不用担心其过期的原因。每次我们重新渲染，都会生成*新的* effect，替换掉之前的。某种意义上讲，effect 更像是渲染结果的一部分 —— 每个 effect “属于”一次特定的渲染。我们将在[本章节后续部分](https://zh-hans.reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update)更清楚地了解这样做的意义。

> 提示
>
> 与 `componentDidMount` 或 `componentDidUpdate` 不同，使用 `useEffect` 调度的 effect 不会阻塞浏览器更新屏幕，这让你的应用看起来响应更快。大多数情况下，effect 不需要同步地执行。在个别情况下（例如测量布局），有单独的 [`useLayoutEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect) Hook 供你使用，其 API 与 `useEffect` 相同。

### 3.3 需要清除的 effect

之前，我们研究了如何使用不需要清除的副作用，还有一些副作用是需要清除的。例如**订阅外部数据源**。这种情况下，清除工作是非常重要的，可以防止引起内存泄露！现在让我们来比较一下如何用 Class 和 Hook 来实现。

#### 3.3.1 使用 Class 的示例

在 React class 中，你通常会在 `componentDidMount` 中设置订阅，并在 `componentWillUnmount` 中清除它。例如，假设我们有一个 `ChatAPI` 模块，它允许我们订阅好友的在线状态。以下是我们如何使用 class 订阅和显示该状态：

```js
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
  }

  componentDidMount() {
    ChatAPI.subscribe(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  componentWillUnmount() {
    ChatAPI.unsubscribe(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  handleStatusChange = (status) => {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

你会注意到 `componentDidMount` 和 `componentWillUnmount` 之间相互对应。使用生命周期函数迫使我们拆分这些逻辑代码，即使这两部分代码都作用于相同的副作用。

> 注意
>
> 眼尖的读者可能已经注意到了，这个示例还需要编写 `componentDidUpdate` 方法才能保证完全正确。我们先暂时忽略这一点，本章节中[后续部分](https://zh-hans.reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update)会介绍它。

#### 3.3.2 使用 Hook 的示例

如何使用 Hook 编写这个组件。

你可能认为需要单独的 effect 来执行清除操作。但由于添加和删除订阅的代码的紧密性，所以 `useEffect` 的设计是在同一个地方执行。如果你的 effect 返回一个函数，React 将会在下一次effect执行前调用它，我们可以在这个函数中清除掉前一次effect执行所带来的影响。

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribe(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribe(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

**为什么要在 effect 中返回一个函数？** 这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。

**React 何时清除 effect？** React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 React *会*在执行当前 effect 之前对上一个 effect 进行清除。

> 注意
>
> 并不是必须为 effect 中返回的函数命名。这里我们将其命名为 `cleanup` 是为了表明此函数的目的，但其实也可以返回一个箭头函数或者给起一个别的名字。

### 3.4 通过跳过 Effect 进行性能优化

在某些情况下，每次渲染后都执行清理或者执行 effect 可能会导致性能问题。在 class 组件中，我们可以通过在 `componentDidUpdate` 中添加对 `prevProps` 或 `prevState` 的比较逻辑解决：

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

这是很常见的需求，所以它被内置到了 `useEffect` 的 Hook API 中。如果某些特定值在两次重渲染之间没有发生变化，你可以通知 React **跳过**对 effect 的调用，只要传递数组作为 `useEffect` 的第二个可选参数即可：

```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```

上面这个示例中，我们传入 `[count]` 作为第二个参数。这个参数是什么作用呢？如果 `count` 的值是 `5`，而且我们的组件重渲染的时候 `count` 还是等于 `5`，React 将对前一次渲染的 `[5]` 和后一次渲染的 `[5]` 进行比较。因为数组中的所有元素都是相等的(`5 === 5`)，React 会跳过这个 effect，这就实现了性能的优化。

当渲染时，如果 `count` 的值更新成了 `6`，React 将会把前一次渲染时的数组 `[5]` 和这次渲染的数组 `[6]` 中的元素进行对比。这次因为 `5 !== 6`，React 就会再次调用 effect。如果数组中有多个元素，即使只有一个元素发生变化，React 也会执行 effect。

对于有清除操作的 effect 同样适用：

```js
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribe(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribe(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // 仅在 props.friend.id 发生变化时，重新订阅
```

未来版本，可能会在构建时自动添加第二个参数。

> 注意：
>
> 如果你要使用此优化方式，请确保数组中包含了**所有外部作用域中会随时间变化并且在 effect 中使用的变量**，否则你的代码会引用到先前渲染中的旧变量。参阅文档，了解更多关于[如何处理函数](https://zh-hans.reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)以及[数组频繁变化时的措施](https://zh-hans.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)内容。
>
> 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行。这并不属于特殊情况 —— 它依然遵循依赖数组的工作方式。
>
> 如果你传入了一个空数组（`[]`），effect 内部的 props 和 state 就会一直拥有其初始值。尽管传入 `[]` 作为第二个参数更接近大家更熟悉的 `componentDidMount` 和 `componentWillUnmount` 思维模式，但我们有[更好的](https://zh-hans.reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)[方式](https://zh-hans.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)来避免过于频繁的重复调用 effect。除此之外，请记得 React 会等待浏览器完成画面渲染之后才会延迟调用 `useEffect`，因此会使得额外操作很方便。
>
> 我们推荐启用 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) 中的 [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) 规则。此规则会在添加错误依赖时发出警告并给出修复建议。

## 4.useRef

```js
const refContainer = useRef(initialValue);
```

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内持续存在。

一个常见的用例便是命令式地访问子组件：

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

我们要获取元素的真实DOM对象，首先我们需要使用useRef()这个钩子函数获取一个对象，这个对象就是一个容器，React会自动将DOM对象传递到容器中。代码`const divRef = useRef()`就是通过钩子函数在创建这个对象，并将其存储到变量中。

创建对象后，还需要在被获取引用的元素上添加一个ref属性，该属性的值就是刚刚我们所声明的变量，像是这样`ref={divRef}`这句话的意思就是将对象的引用赋值给变量divRef。这两个步骤缺一不可，都处理完了，就可以通过divRef来访问原生DOM对象了。

useRef()返回的是一个普通的JS对象，JS对象中有一个current属性，它指向的便是原生的DOM对象。上例中，如果想访问div的原生DOM对象，只需通过`divRef.current`即可访问，它可以调用DOM对象的各种方法和属性，但还是要再次强调：慎用！

尽量减少在React中操作原生的DOM对象，如果实在非得操作也尽量是那些不会对数据产生影响的操作，像是设置焦点、读取信息等。

`useRef()`所返回的对象就是一个普通的JS对象，所以上例中即使我们不使用钩子函数，仅仅创建一个形如`{current:null}`的对象也是可以的。只是我们自己创建的对象组件每次渲染时都会重新创建一个新的对象，而通过`useRef()`创建的对象可以确保组件每次的重渲染获取到的都是相同的对象。

## 5.useReducer

### 5.1 基本使用

为了解决复杂`State`带来的不便，`React`为我们提供了一个新的使用`State`的方式。`Reducer`横空出世，reduce单词中文意味减少，而reducer我觉得可以翻译为“当你的state的过于复杂时，你就可以使用的可以对state进行整合的工具”。当然这是个玩笑话，个人认为`Reducer`可以翻译为“整合器”，它的作用就是将那些和同一个`state`相关的所有函数都整合到一起，方便在组件中进行调用。

当然工具都有其使用场景，`Reducer`也不例外，它只适用于那些比较复杂的`state`，对于简单的`state`使用`Reducer`只能是徒增烦恼。

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

它的返回值和`useState()`类似，第一个参数是`state`用来读取`state`的值，第二个参数同样是一个函数，不同于`setState()`这个函数我们可以称它是一个“派发器”，通过它可以向`reducer()`发送不同的指令，控制`reducer()`做不同的操作。

它的参数有三个，第三个我们暂且忽略，只看前两个。`reducer()`是一个函数，也是我们所谓的“整合器”。它的返回值会成为新的`state`值。当我们调用`dispatch()`时，`dispatch()`会将消息发送给`reducer()`，`reducer()`可以根据不同的消息对`state`进行不同的处理。`initialArg`就是`state`的初始值，和`useState()`参数一样。

以下是用 `reducer `写的的计数器示例：

```jsx
/*
*   参数：
*       reducer : 整合函数
*           对于我们当前state的所有操作都应该在该函数中定义
*           该函数的返回值，会成为state的新值
*           reducer在执行时，会收到两个参数：
*               state 当前最新的state
*               action 它需要一个对象
*                       在对象中会存储dispatch所发送的指令
*       initialArg : state的初始值，作用和useState()中的值是一样
*   返回值：
*       数组：
*           第一个参数，state 用来获取state的值
*           第二个参数，state 修改的派发器
*                   通过派发器可以发送操作state的命令
*                   具体的修改行为将会由另外一个函数(reducer)执行
* */

// 为了避免reducer会重复创建，通常reducer会定义到组件的外部
function countReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [count, countDispatch] = useReducer(countReducer, {count: 0});
    // 这里本来初始值是直接给0的，但是为了countReducer函数中的state写成对象形式
    
  return (
    <>
      Count: {count.count}
      <button onClick={() => countDispatch({type: 'decrement'})}>-</button>
      <button onClick={() => countDispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

### 5.2 state初始化的两种方式

**指定初始 state**

有两种不同初始化 `useReducer` state 的方式，你可以根据使用场景选择其中的一种。将初始 state 作为第二个参数传入 `useReducer` 是最简单的方法：

```js
const [state, dispatch] = useReducer(
reducer,
{count: 0}  );
```

**惰性初始化**

你可以选择惰性地创建初始 state。为此，需要将 `init` 函数作为 `useReducer` 的第三个参数传入，这样初始 state 将被设置为 `init(initialArg)`。

这么做可以将用于计算 state 的逻辑提取到 reducer 外部，这也为将来对重置 state 的 action 做处理提供了便利：

```jsx
export default function App() {
  return (
    <div>
      <Counter initialCount={0} />
    </div>
  )
}

function countInit(initialCount) {
  return {count: initialCount};
}

function countReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return countInit(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [count, countDispatch] = useReducer(countReducer, initialCount, countInit);
  return (
    <>
      Count: {count.count}
      <button
        onClick={() => countDispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => countDispatch({type: 'decrement'})}>-</button>
      <button onClick={() => countDispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

![image-20221030143937094](https://i0.hdslb.com/bfs/album/08a8b739d946a92c01ec286bfce8b81771a42e71.png)

### 5.3 跳过 dispatch

如果 Reducer Hook 的返回值与当前 state 相同，React 将跳过子组件的渲染及副作用的执行。（React 使用 [`Object.is` 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 来比较 state。）

> 这里的state如果是个对象，还是会渲染子组件，因为我们返回的是一个新对象，我想应该比较的是地址，如果直接将state返回，子组件是不会重新渲染的

需要注意的是，React 可能仍需要在跳过渲染前再次渲染该组件。不过由于 React 不会对组件树的“深层”节点进行不必要的渲染，所以大可不必担心。如果你在渲染期间执行了高开销的计算，则可以使用 `useMemo` 来进行优化。


# 15【react-Hook （下）】

## 1.React.memo

### 1.1 基本介绍

> 这是一个高阶组件，用来做性能优化的，这个本来应该是写在`React高级指引`中的，但是这个案例会和后面的`useCallback`联合起来，所以就写在这里了

*   React.memo() 是一个高阶组件，如果你的组件在相同 props 的情况下渲染相同的结果，那么你可以通过将其包装在 `React.memo` 中调用，以此通过记忆组件渲染结果的方式来提高组件的性能表现。
    *   它接收另一个组件作为参数，并且会返回一个包装过的新组件
    *   包装过的新组件就会具有缓存功能，这意味着在这种情况下，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果。
    *   包装过后，只有组件的props发生变化，才会触发组件的重新的渲染，否则总是返回缓存中结果。如果函数组件被 `React.memo` 包裹，且其实现中拥有 [`useState`](https://zh-hans.reactjs.org/docs/hooks-state.html)，[`useReducer`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer) 或 [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext) 的 Hook，当 state 或 context 发生变化时，它仍会重新渲染。

### 1.2 问题的引出

React组件会在两种情况下发生重新渲染。第一种，当组件自身的state发生变化时。第二种，当组件的父组件重新渲染时。第一种情况下的重新渲染无可厚非，state都变了，组件自然应该重新进行渲染。但是第二种情况似乎并不是总那么的必要。

`App.jsx`

```jsx
import React, { useState } from 'react'

export default function App() {
  console.log('App渲染')

  const [count, setCount] = useState(1)

  const clickHandler = () => {
    setCount(prevState => prevState + 1)
  }

  return (
    <div>
      <h2>App -- {count}</h2>
      <button onClick={clickHandler}>增加</button>
      <A />
    </div>
  )
}

function A() {
  console.log('A渲染')
  return <div>我是A组件</div>
}
```

在点击增加后，我们发现`App`和`A`都重新渲染了。

当APP组件重新渲染时，A组件也会重新渲染。A组件中没有state，甚至连props都没有设置。换言之，A组件无论如何渲染，每次渲染的结果都是相同的，虽然重渲染并不会应用到真实DOM上，但很显然这种渲染是完全没有必要的。

![image-20221030172720453](https://i0.hdslb.com/bfs/album/86fdfde2cc02ab0826730bc14eefc637c2696ecd.png)

为了减少像A组件这样组件的渲染，React为我们提供了一个方法`React.memo()`。该方法是一个高阶函数，可以用来根据组件的props对组件进行缓存，当一个组件的父组件发生重新渲染，而子组件的props没有发生变化时，它会直接将缓存中的组件渲染结果返回而不是再次触发子组件的重新渲染，这样一来就大大的降低了子组件重新渲染的次数。

### 1.3 使用React.memo

使用`React.memo`包裹`A组件`

> 这里只是为了演示方便，把所有组件写一个文件，就用这种方式包裹`A组件`,平时单文件组件的时候我们这样使用,`export default React.memo(A)`

```jsx
import React, { useState } from 'react'

export default function App() {
  console.log('App渲染')

  const [count, setCount] = useState(1)

  const clickHandler = () => {
    setCount(prevState => prevState + 1)
  }

  return (
    <div>
      <h2>App -- {count}</h2>
      <button onClick={clickHandler}>增加</button>
      <A />
    </div>
  )
}

const A = React.memo(() => {
  console.log('A渲染')
  return <div>我是A组件</div>
})
```

修改后的代码中，并没有直接使用A组件，而是在A组件外层套了一层函数`React.memo()`，这样一来，返回的A组件就增加了缓存功能，只有当A组件的props属性发生变化时，才会触发组件的重新渲染。memo只会根据props判断是否需要重新渲染，和state和context无关，state或context发生变化时，组件依然会正常的进行重新渲染

在点击增加后，我们发现只有`App`重新渲染了。

![image-20221030173239606](https://i0.hdslb.com/bfs/album/0a613f29500b63ae26e2db2870ef9c18f3e7815f.png)

这时我们改下代码

```jsx
export default function App() {
  console.log('App渲染')

  const [count, setCount] = useState(1)

  const clickHandler = () => {
    setCount(prevState => prevState + 1)
  }
	
  // 增加
  const test = count % 4 === 0

  return (
    <div>
      <h2>App -- {count}</h2>
      <button onClick={clickHandler}>增加</button>
  	  {/* 改动 */}
      <A test={test} />
    </div>
  )
}

const A = React.memo(props => {
  console.log('A渲染')
  return (
    <div>
      我是A组件
      {/* 增加 */}
      <p>{props.test && 'props.test 为 true'}</p>
    </div>
  )
})
```

这次加了个表达式的结果传给`A`组件，一开始是`false`，只有为`true`的时候，`A`组件才会重新渲染

这时界面是这样的

![image-20221030174105525](https://i0.hdslb.com/bfs/album/9ddba53bea0c611e1c3e2837761f2b60f2f29322.png)

点击3次后，表达式为`true`，A组件的`props`发生改变，所以重新渲染了。

![image-20221030173754653](https://i0.hdslb.com/bfs/album/0f5ff4c127b4d404f1207ccbf497a5cd271d4651.png)

### 1.4 使用注意

1. 此方法仅作为**[性能优化](https://zh-hans.reactjs.org/docs/optimizing-performance.html)**的方式而存在。但请不要依赖它来“阻止”渲染，因为这会产生 bug。
2. 与 class 组件中 [`shouldComponentUpdate()`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate) 方法不同的是，如果 props 相等，`areEqual` 会返回 `true`；如果 props 不相等，则返回 `false`。这与 `shouldComponentUpdate` 方法的返回值相反。

### 1.5 容易出错的情况

先回到这个案例的初始代码，在这之上进行修改

我们把`App组件`的`clickHandler`方法传递给`A组件`，让`A组件`也能够改变`App组件`的`state`

```jsx
import React, { useState } from 'react'

export default function App() {
  console.log('App渲染')

  const [count, setCount] = useState(1)

  const clickHandler = () => {
    setCount(prevState => prevState + 1)
  }

  return (
    <div>
      <h2>App -- {count}</h2>
      <button onClick={clickHandler}>增加</button>
      <A clickHandler={clickHandler} />
    </div>
  )
}

const A = React.memo(props => {
  console.log('A渲染')
  return (
    <div>
      我是A组件
      <button onClick={props.clickHandler}>A组件的增加</button>
    </div>
  )
})
```

点击`A组件的增加`,发现`A组件`也重新渲染了

![image-20221030175830062](https://i0.hdslb.com/bfs/album/3e8995b7352d72b47e772ea6c7fed94ef1531267.png)

这是因为`App组件`重新渲染的时候，`clickHandler`也重新创建了，这时传递给子组件的`clickHandler`和上一次不一样，所以`react.memo`失效了。

这个问题可以用`useCallback`解决。

## 2.useCallback

### 2.1 基本介绍

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

把内联回调函数及依赖项数组作为参数传入 `useCallback`，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 `shouldComponentUpdate`）的子组件时，它将非常有用。

`useCallback`和`useMemo`设计的初衷是用来做性能优化的。在`Class Component`中考虑以下的场景：

```javascript
class Foo extends Component {
  handleClick() {
    console.log('Click happened');
  }
  render() {
    return <Button onClick={() => this.handleClick()}>Click Me</Button>;
  }
}
```

传给 Button 的 onClick 方法每次都是重新创建的，这会导致每次 Foo render 的时候，Button 也跟着 render。优化方法有 2 种，箭头函数和 bind。下面以 bind 为例子：

```javascript
class Foo extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    console.log('Click happened');
  }
  render() {
    return <Button onClick={this.handleClick}>Click Me</Button>;
  }
}
```

同样的，`Function Component`也有这个问题：

```javascript
function Foo() {
  const [count, setCount] = useState(0);

  const handleClick() {
    console.log(`Click happened with dependency: ${count}`)
  }
  return <Button onClick={handleClick}>Click Me</Button>;
}
```

而 React 给出的方案是`useCallback` Hook。在依赖不变的情况下 (在我们的例子中是 count )，它会返回相同的引用，避免子组件进行无意义的重复渲染

### 2.2 解决1.5遗留的问题

```javascript
/*
*   useCallback()
*		这个hook会缓存方法的引用
*       参数：
*           1. 回调函数
*           2. 依赖数组
*               - 当依赖数组中的变量发生变化时，回调函数才会重新创建
*               - 如果不指定依赖数组，回调函数每次都会重新创建
*               - 一定要将回调函数中使用到的所有变量都设置到依赖数组中
*                   除了（setState）
* */
```

我们将`clickHandler`方法改造一下

```js
  const clickHandler = useCallback(() => {
    setCount(prevState => prevState + 1)
  }, [])
```

> 第二个参数一定要加，不然和平常写没有区别
>
> 依赖项`[]`的意思是只有第一次渲染时才会创建，之后都不会重新创建了

点击`A组件的增加`,发现只有`App组件`重新渲染了。因为`clickHandler`没有重新创建，传给子组件的没有变化，所以子组件这次没有重新渲染。

![image-20221030180349406](https://i0.hdslb.com/bfs/album/2e7d34de3518ecb98ac3433cae205a4683a069f5.png)

**完整代码**

```jsx
import React, { useState, useCallback } from 'react'

export default function App() {
  console.log('App渲染')

  const [count, setCount] = useState(1)

  const clickHandler = useCallback(() => {
    setCount(prevState => prevState + 1)
  }, [])

  return (
    <div>
      <h2>App -- {count}</h2>
      <button onClick={clickHandler}>增加</button>
      <A clickHandler={clickHandler} />
    </div>
  )
}

const A = React.memo(props => {
  console.log('A渲染')
  return (
    <div>
      我是A组件
      <button onClick={props.clickHandler}>A组件的增加</button>
    </div>
  )
})
```

### 2.3 第二个参数的使用

继续改造上面的代码

```jsx
import React, { useState, useCallback } from 'react'

export default function App() {
  console.log('App渲染')

  const [count, setCount] = useState(1)
  // 增加
  const [num, setNum] = useState(1)

  const clickHandler = useCallback(() => {
    setCount(prevState => prevState + num)
  // 增加
    setNum(prevState => prevState + 1)
  }, [])

  return (
    <div>
      <h2>App -- {count}</h2>
      <button onClick={clickHandler}>增加</button>
      <A clickHandler={clickHandler} />
    </div>
  )
}

const A = React.memo(props => {
  console.log('A渲染')
  return (
    <div>
      我是A组件
      <button onClick={props.clickHandler}>A组件的增加</button>
    </div>
  )
})
```

增加了一个`num`，让每一次`count`的增加比上次多1，现在这样写是有问题的。

![image-20221030181249832](https://i0.hdslb.com/bfs/album/2e04eb7727a7b875a3b771761c4c4d2ac9e30b04.png)

点击了两次增加后，预期值应该是4，但是显示的是3，是为什么呢？

因为`clickHandler`只在初次渲染的时候创建，当时`num`的值是1，这个函数一直没有重新创建，内部用的`num`一直是1

这时我们可以加一个依赖项

```js
const clickHandler = useCallback(() => {
    setCount(prevState => prevState + num)
    setNum(prevState => prevState + 1)
  }, [num])
```

这样`num`变化了，这个函数也会重新创建。

![image-20221030181534667](https://i0.hdslb.com/bfs/album/d1f9956dac9668ab3c7af6dc5f093f1155f39d6b.png)

点击了两次增加后，count变成了预期值4。

## 3.useMemo

useMemo和useCallback十分相似，useCallback用来缓存函数对象，useMemo用来缓存函数的执行结果。在组件中，会有一些函数具有十分的复杂的逻辑，执行速度比较慢。闭了避免这些执行速度慢的函数返回执行，可以通过useMemo来缓存它们的执行结果，像是这样：

```js
const result = useMemo(()=>{
    return 复杂逻辑函数();
},[依赖项])
```

useMemo中的函数会在依赖项发生变化时执行，注意！是执行，这点和useCallback不同，useCallback是创建。执行后返回执行结果，如果依赖项不发生变化，则一直会返回上次的结果，不会再执行函数。这样一来就避免复杂逻辑的重复执行。

### 3.1 问题的引出

`App.jsx`

````jsx
import React, { useMemo, useState } from 'react'

const App = () => {
  const [count, setCount] = useState(1)

  let a = 123
  let b = 456

  function sum(a, b) {
    console.log('sum执行了')
    return a + b
  }
    
  return (
    <div>
      <h1>App</h1>
      <p>sum的结果：{sum(a, b)}</p>
      <h3>{count}</h3>
      <button onClick={() => setCount(prevState => prevState + 1)}>点我</button>
    </div>
  )
}

export default App
````

这是一个计数器案例，但是多添加了一个函数展示结果，这种情况这个函数只需要在一开始调用一次就够了，但是`count`的改变会导致重新渲染模板，这样`sum`函数也会反复执行。

![image-20221107143139632](https://i0.hdslb.com/bfs/album/2e224dfe8aa7df39ae77c2d98bede3d590d7d1b8.png)

现在这个`sum`函数太简单了，体现不出性能上的问题，我们可以把sum中的逻辑改复杂一点。

```jsx
import React, { useMemo, useState } from 'react'

const App = () => {
  const [count, setCount] = useState(1)

  let a = 123
  let b = 456

  function sum(a, b) {
    console.log('sum执行了')
    const begin = +new Date()
    while (true) {
      if (Date.now() - begin > 3000) break
    }
    return a + b
  }
  return (
    <div>
      <h1>App</h1>
      <p>sum的结果：{sum(a, b)}</p>
      <h3>{count}</h3>
      <button onClick={() => setCount(prevState => prevState + 1)}>点我</button>
    </div>
  )
}

export default App
```

增加了一个功能，让这个函数起码3秒才能执行完。

![image-20221107143451560](https://i0.hdslb.com/bfs/album/67454e5c8256260a19f3cc84d2b1da78935ec5d3.png)

这个时候因为`sum`函数要3秒才能执行完，导致下面数字显示也变慢了3秒。

### 3.2 使用 useMemo 解决上面的问题

`App.jsx`

改写模板中的`sum`方法的调用

```jsx
<p>sum的结果：{useMemo(() => sum(a, b), [])}</p>
```

![image-20221107143946116](https://i0.hdslb.com/bfs/album/790ec03b82b253c6a80c823542d21b745adb8035.png)

第一次加载慢是不可避免的，但是这个钩子函数将`sum`函数的返回值缓存起来，这样我们模板重新渲染时就没有再去执行`sum`函数，而是直接使用上一次的返回值。

### 3.3 第二个参数的使用

继续改造上面的代码，把`Sum`单独抽离成一个组件

`Sum.jsx`

```jsx
import React from 'react'

export default function Sum(props) {
  console.log('Sum执行了')
  return <span>{props.a + props.b}</span>
}
```

`App.jsx`

添加了一个功能可以变换`a`的值

```jsx
import React, { useMemo, useState } from 'react'
import Sum from './Sum'

const App = () => {
  const [count, setCount] = useState(1)

  let a = 123
  let b = 456

  if (count % 2 === 0) a = a + 1

  const result = useMemo(() => <Sum a={a} b={b} />, [])

  return (
    <div>
      <h1>App</h1>
      <p>sum的结果：{result}</p>
      <h3>{count}</h3>
      <button onClick={() => setCount(prevState => prevState + 1)}>点我</button>
    </div>
  )
}

export default App
```

现在有一个问题，如果`Sum`组件接收的值变化了，网页上显示的还是原来的缓存值，这个时候就要利用第二个参数。

![image-20221107145159066](https://i0.hdslb.com/bfs/album/deecc395220dcff4c6c9acb8b71bbda6ca6a5ae7.png)

`App.jsx`

```jsx
const result = useMemo(() => <Sum a={a} b={b} />, [a])
```

> 这里的意思和以前是一样的，如果`a`的值变化了，将会重新计算。

![image-20221107145403725](https://i0.hdslb.com/bfs/album/931d55fbd9ffca98894a4b8178c8da9751231923.png)

## 4.React.forwardRef

> 这是一个高阶组件，用来做性能优化的，这个本来应该是写在`React高级指引`中的，但是这个案例会和后面的`useImperativeHandle`联合起来，所以就写在这里了

`React.forwardRef` 会创建一个React组件，这个组件能够将其接受的 [ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 属性转发到其组件树下的另一个组件中。这种技术并不常见，但在以下两种场景中特别有用：

- [转发 refs 到 DOM 组件](https://zh-hans.reactjs.org/docs/forwarding-refs.html#forwarding-refs-to-dom-components)
- [在高阶组件中转发 refs](https://zh-hans.reactjs.org/docs/forwarding-refs.html#forwarding-refs-in-higher-order-components)

`React.forwardRef` 接受渲染函数作为参数。React 将使用 `props` 和 `ref` 作为参数来调用此函数。此函数应返回 React 节点。

```jsx
import React, { useRef } from 'react'

const Child = React.forwardRef((props, ref) => {
  return (
    <>
      <h2>这是Child组件</h2>
      <input type="text" ref={ref} />
    </>
  )
})

export default function App() {
  const childRef = useRef(null)
  console.log(childRef)
  return (
    <div>
      <h2>这是App组件</h2>
      <Child ref={childRef} />
    </div>
  )
}
```

在上述的示例中，React 会将 `<Child ref={childRef}>` 元素的 `ref` 作为第二个参数传递给 `React.forwardRef` 函数中的渲染函数。该渲染函数会将 `ref` 传递给 `<input ref={ref}>` 元素。

因此，当 React 附加了 ref 属性之后，`ref.current` 将直接指向 `<input>` DOM 元素实例。

![image-20221107150428406](https://i0.hdslb.com/bfs/album/1cb30d9536293b08fe2097448858e58e3826a242.png)

我们改造`App`组件

```jsx
export default function App() {
  const childRef = useRef(null)

  childRef.current.value = 'App组件设置的'

  return (
    <div>
      <h2>这是App组件</h2>
      <Child ref={childRef} />
    </div>
  )
}
```

![image-20221107150535398](https://i0.hdslb.com/bfs/album/97b65dfb2775fb6d2b94e3e7e0bd1c57a0f45571.png)

我们可以直接在`App`组件操作`Child`组件的内容，但是这样并不好，我们希望`Child`组件的内容只由`Child`组件自己去操作，所以引出了`useImperativeHandle`

## 5.useImperativeHandle

```js
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` 可以让你在使用 `ref` 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。`useImperativeHandle` 应当与 [`forwardRef`](https://zh-hans.reactjs.org/docs/react-api.html#reactforwardref) 一起使用：

`App.jsx`

```jsx
import React, { useRef, useEffect, useImperativeHandle } from 'react'

const Child = React.forwardRef((props, ref) => {
  const inputRef = useRef(null)

  const changeInputValue = value => (inputRef.current.value = value)

  // useImperativeHandle 可以用来指定ref返回的值
  useImperativeHandle(ref, () => ({
    changeInputValue,
  }))

  return (
    <>
      <h2>这是Child组件</h2>
      <input type="text" ref={inputRef} />
    </>
  )
})

export default function App() {
  const childRef = useRef(null)

  useEffect(() => {
    console.log(childRef)
  }, [])

  return (
    <div>
      <h2>这是App组件</h2>
      <button onClick={() => childRef.current.changeInputValue('App组件修改的')}>点击改变</button>
      <Child ref={childRef} />
    </div>
  )
}
```

我们来看看`childRef`的输出是什么

![image-20221107151926890](https://i0.hdslb.com/bfs/album/bf501a8976223d92e9330a13aadba40a623e73b0.png)

可以发现我们把子组件的`changeInputValue`暴露出去了。

![image-20221107152122394](https://i0.hdslb.com/bfs/album/fb13b0f4d452eae98a26edeabc03076defb73f52.png)

点击按钮发现也是可以正常使用的。
