# 01 【react入门】

## 1.React简介

**react是什么？**

**React** 是一个用于构建用户界面的 JavaScript 库。

- 是一个将数据渲染为 HTML 视图的开源 JS 库
- 它遵循基于组件的方法，有助于构建可重用的 UI 组件
- 它用于开发复杂的交互式的 web 和移动 UI

> React 有什么特点？

1. 使用虚拟 DOM 而不是真正的 DOM
2. 它可以用服务器渲染
3. 它遵循单向数据流或数据绑定
4. 高效
5. 声明式编码，组件化编码

> React 的一些主要优点？

1. 它提高了应用的性能
2. 可以方便在客户端和服务器端使用
3. 由于使用 JSX，代码的可读性更好
4. 使用React，编写 UI 测试用例变得非常容易

**为什么学？**

1.原生JS操作DOM繁琐，效率低

2.使用JS直接操作DOM,浏览器会进行大量的重绘重排

3.原生JS没有组件化编码方案，代码复用低

> 在学习之前最好看一下关于npm的知识：下面是我在网上看见的一个写的还不错的npm的文章
>
> [npm](https://blog.csdn.net/qq_25502269/article/details/79346545)

## 2.React 基础案例

首先需要引入几个 react 包

- React 核心库、操作 DOM 的 react 扩展库、将 jsx 转为 js 的 babel 库

【先引入react.development.js，后引入react-dom.development.js】

> `react.development.js`
>
> - react 是react核心库，只要使用react就必须要引入
> - 下载地址：https://unpkg.com/react@18.0.0/umd/react.development.js
>
> `react-dom.development.js`
>
> - react-dom 是react的dom包，使用react开发web应用时必须引入
> - 下载地址：https://unpkg.com/react-dom@18.0.0/umd/react-dom.development.js
>
> `babel.min.js `
>
> - 由于JSX最终需要转换为JS代码执行，所以浏览器并不能正常识别JSX，所以当我们在浏览器中直接使用JSX时，还必须引入babel来完成对代码的编译。
>
> - babel下载地址：https://unpkg.com/babel-standalone@6/babel.min.js

![image-20221022171647360](https://i0.hdslb.com/bfs/album/514c5df0f5f8e7242ca17e1c939b4822b716315f.png)

```
react.development.js
react-dom.development.js
babel.min.js 
```

2.创建一个容器

3.创建虚拟DOM，渲染到容器中

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>hello_react</title>
  </head>
  <body>
    <!-- 准备好一个“容器” -->
    <div id="test"></div>

    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>

    <script type="text/babel">
      /* 此处一定要写babel */
      //1.创建虚拟DOM
      const VDOM = <h1>Hello</h1> /* 此处一定不要写引号，因为不是字符串 */
      //2.渲染虚拟DOM到页面
	const root = ReactDOM.createRoot(document.querySelector('#test'));
      root.render(VDOM);
    </script>
  </body>
</html>
```

> 后面很多地方没有用`createRoot`这种方式是因为一开始学的课程是2020年的，这是现在新的创建方式。
>
> 这里我就只把第一个案例改成新方式了

这样，就会在页面中的这个div容器上添加这个h1.

![image-20221022171539523](https://i0.hdslb.com/bfs/album/7c5713f248cc28bcb531d2eda325e56789df4286.png)

> *   React.createElement()
>     - `React.createElement(type, [props], [...children])`
>     - 用来创建React元素
>     - React元素无法修改
> *   ReactDOM.createRoot()
>     - `createRoot(container[, options])`
>     - 用来创建React的根容器，容器用来放置React元素
> *   ReactDOM.render()
>     *   `root.render(element)`
>     *   用来将React元素渲染到根元素中
>     *   根元素中所有的内容都会被删除，被React元素所替换
>     *   当重复调用render()时，React会将两次的渲染结果进行比较，
>     *   它会确保只修改那些发生变化的元素，对DOM做最少的修改

## 3.jsx 语法

JSX 是 JavaScript 的语法扩展，JSX 使得我们可以以类似于 HTML 的形式去使用 JS。JSX便是React中声明式编程的体现方式。声明式编程，简单理解就是以结果为导向的编程。使用JSX将我们所期望的网页结构编写出来，然后React再根据JSX自动生成JS代码。所以我们所编写的JSX代码，最终都会转换为以调用`React.createElement()`创建元素的代码。

1. 定义虚拟DOM，JSX不是字符串，不要加引号
2. 标签中混入JS表达式的时候使用`{}`

```js
id = {myId.toUpperCase()}
```

3. 样式的类名指定不能使用class，使用`className`

4. 内敛样式要使用`{{}}`包裹

```js
style={{color:'skyblue',fontSize:'24px'}}
```

5. 不能有多个根标签，只能有一个根标签

6.  JSX的标签必须正确结束（自结束标签必须写/）

7. JSX中html标签应该小写，React组件应该大写开头。如果小写字母开头，就将标签转化为 html 同名元素，如果 html 中无该标签对应的元素，就报错；如果是大写字母开头，react 就去渲染对应的组件，如果没有就报错

8. 如果表达式是空值、布尔值、undefined，将不会显示

> 关于JS表达式和JS语句：
>
> - JS表达式：返回一个值，可以放在任何一个需要值的地方 a a+b demo(a) arr.map() function text(){} 
>
> - JS语句：if(){} for(){} while(){} swith(){} 不会返回一个值

**其它**

1. 注释

写在花括号里

```js
ReactDOM.render(
    <div>
    <h1>小丞</h1>
    {/*注释...*/}
     </div>,
    document.getElementById('example')
);
```

2. `class`需要使用`className`代替
3. `style`中必须使用对象设置` style={{background:'red'}}`

```js
<style>
	.title{
		background-color: orange;
		width: 200px;
	}
</style>

<!-- 准备好一个“容器” -->
<div id="test"></div>

<script type="text/babel" >
	const myId = 'aTgUiGu'
	const myData = 'HeLlo,rEaCt'

	//1.创建虚拟DOM
	const VDOM = (
		<div>
			<h2 className="title" id={myId.toLowerCase()}>
				<span style={{color:'white',fontSize:'29px'}}>{myData.toLowerCase()}</span>
			</h2>
			<h2 className="title" id={myId.toLowerCase()}>
				<span style={{color:'white',fontSize:'29px'}}>{myData.toUpperCase()}</span>
			</h2>
			<input type="text"/>
		</div>
	)
	//2.渲染虚拟DOM到页面
	ReactDOM.render(VDOM,document.getElementById('test'))
</script>
```

![image-20221022204158589](https://i0.hdslb.com/bfs/album/9d4bafde75cb82f79b17a91491c46eb8576b784a.png)

4. 数组

JSX 允许在模板中插入数组，数组自动展开全部成员

> {} 只能用来放js表达式，而不能放语句（if for）
> 在语句中是可以去操作JSX

```js
var arr = [
  <h1>小丞</h1>,
  <h2>同学</h2>,
];
ReactDOM.render(
  <div>{arr}</div>,
  document.getElementById('example')
);
```

**tip: JSX 小练习**

根据动态数据生成 `li`

```js
const data = ['A','B','C']
const VDOM = (
    <div>
        <ul>
            {
                data.map((item,index)=>{
                    return <li key={index}>{item}</li>
                })
            }
        </ul>
    </div>
)
ReactDOM.render(VDOM,document.querySelector('.test'))
```

<img src="https://i0.hdslb.com/bfs/album/09241923178d7fdca14d087e6f1a9627dc3b7081.png" alt="image-20221022204645014"  />



## 4.两种创建虚拟DOM的方式

**使用JSX创建虚拟DOM**

```js
//1.创建虚拟DOM
	const VDOM = (  /* 此处一定不要写引号，因为不是字符串 */
    	<h1 id="title">
			<span>Hello,React</span>
		</h1>
	)
//2.渲染虚拟DOM到页面
	ReactDOM.render(VDOM,document.querySelector('.test'))
```

这个在上面的案例中已经演示过了 ，下面看看另外一种创建虚拟DOM的方式

**2.使用JS创建虚拟DOM**

````js
/*
* React.createElement()
*   - 用来创建一个React元素
*   - 参数：
*        1.元素的名称（html标签必须小写）
*        2.标签中的属性
*           - class属性需要使用className来设置
*           - 在设置事件时，属性名需要修改为驼峰命名法
*       3.元素的内容（子元素）
*   - 注意点：
*       React元素最终会通过虚拟DOM转换为真实的DOM元素
*       React元素一旦创建就无法修改，只能通过新创建的元素进行替换
* */
````

```js
//1.创建虚拟DOM,创建嵌套格式的dom
const VDOM=React.createElement('h1',{id:'title'},React.createElement('span',{},'hello,React'))
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.querySelector('.test'))
```

使用JS和JSX都可以创建虚拟DOM，但是可以看出JS创建虚拟DOM比较繁琐，尤其是标签如果很多的情况下，所以还是比较推荐使用JSX来创建。

## 5.两种DOM的区别

````js
    <!-- 准备好一个“容器” -->
    <div id="test"></div>

    <script type="text/babel">
      /* 此处一定要写babel */
      //1.创建虚拟DOM
      const VDOM = <h1>Hello,React</h1> /* 此处一定不要写引号，因为不是字符串 */
      //2.渲染虚拟DOM到页面
      ReactDOM.render(VDOM, document.getElementById('test'))
      const TDOM = document.querySelector('#test')
      console.log('虚拟DOM', VDOM)
      console.dir('真实DOM')
      console.dir(TDOM)
      //   debugger
      console.log(typeof VDOM)
      console.log(VDOM instanceof Object)
````

![image-20221022194600803](https://i0.hdslb.com/bfs/album/3c9c35333c0883a1057bd4c82be8bfbf9b69f04b.png)

 **关于虚拟DOM：**

​          1. 本质是Object类型的对象（一般对象）

​          2. 虚拟DOM比较“轻”，真实DOM比较“重”，因为虚拟DOM是React内部在用，无需真实DOM上那么多的属性。

​          3. 虚拟DOM最终会被React转化为真实DOM，呈现在页面上。

# 02 【面向组件编程】

## 1.组件的使用

当应用是以多组件的方式实现，这个应用就是一个组件化的应用

只有两种方式的组件

- 函数组件
- 类式组件

> **注意：**
>
> 1. 组件名必须是首字母大写（React 会将以小写字母开头的组件视为原生 DOM 标签。例如，< div />`代表 HTML 的 div 标签，而`< Weclome /> 则代表一个组件，并且需在作用域内使用 `Welcome`）
> 2. 虚拟DOM元素只能有一个根元素
> 3. 虚拟DOM元素必须有结束标签 `< />`

**渲染类组件标签的基本流程**

1. React 内部会创建组件实例对象
2. 调用`render()`得到虚拟 DOM ,并解析为真实 DOM
3. 插入到指定的页面元素内部

### 1.1 函数式组件

定义组件最简单的方式就是编写 JavaScript 函数：

```js
 //1.创建函数式组件
      function MyComponent(props) {
        console.log(this) //此处的this是undefined，因为babel编译后开启了严格模式
        return <h2>我是用函数定义的组件(适用于【简单组件】的定义)</h2>
      }

 //2.渲染组件到页面
      ReactDOM.render(<MyComponent />, document.getElementById('test'))
```

该函数是一个有效的 React 组件，因为它接收唯一带有数据的 “props”（代表属性）对象与并返回一个 React 元素。这类组件被称为“函数组件”，因为它本质上就是 JavaScript 函数。

让我们来回顾一下这个例子中发生了什么：

1. React解析组件标签，找到了MyComponent组件。

2. 发现组件是使用函数定义的，随后调用该函数，将返回的虚拟DOM转为真实DOM，随后呈现在页面中。

**注意：** **组件名称必须以大写字母开头。**

React 会将以小写字母开头的组件视为原生 DOM 标签。例如，`<div />` 代表 HTML 的 div 标签，而 `<Welcome />` 则代表一个组件，并且需在作用域内使用 `Welcome`。

你可以在[深入 JSX](https://zh-hans.reactjs.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized) 中了解更多关于此规范的原因。

### 1.2 类式组件

> **将函数组件转换成 class 组件**
>
> 通过以下五步将 `Clock` 的函数组件转成 class 组件：
>
> 1. 创建一个同名的 [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)，并且继承于 `React.Component`。
> 2. 添加一个空的 `render()` 方法。
> 3. 将函数体移动到 `render()` 方法之中。
> 4. 在 `render()` 方法中使用 `this.props` 替换 `props`。
> 5. 删除剩余的空函数声明。

❓React.component is what

```js
class MyComponent extends React.Component {
        render() {
          console.log('render中的this:', this)
          return <h2>我是用类定义的组件(适用于【复杂组件】的定义)</h2>
        }
      }

ReactDOM.render(<MyComponent />, document.getElementById('test'))
```

每次组件更新时 `render` 方法都会被调用，但只要在相同的 DOM 节点中渲染 `<MyComponent/>` ，就仅有一个 `MyComponent` 组件的 class 实例被创建使用。这就使得我们可以使用如 state 或生命周期方法等很多其他特性。

**执行过程：**

1. React解析组件标签，找到相应的组件

2. 发现组件是类定义的，随后new出来的类的实例，并通过该实例调用到原型上的render方法

3. 将render返回的虚拟DOM转化为真实的DOM,随后呈现在页面中

### 1.3 组合组件

组件可以在其输出中引用其他组件。这就可以让我们用同一组件来抽象出任意层次的细节。按钮，表单，对话框，甚至整个屏幕的内容：在 React 应用程序中，这些通常都会以组件的形式表示。

例如，我们可以创建一个可以多次渲染 `Welcome` 组件的 `App` 组件：

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />      
      <Welcome name="Cahal" />      
      <Welcome name="Edite" />    
     </div>
  );
}
```

![image-20221023135154884](https://i0.hdslb.com/bfs/album/61b62685a57a9ebd6162b9d13448aa8d6a74be99.png)

通常来说，每个新的 React 应用程序的顶层组件都是 `App` 组件。但是，如果你将 React 集成到现有的应用程序中，你可能需要使用像 `Button` 这样的小组件，并自下而上地将这类组件逐步应用到视图层的每一处。

### 1.4 提取组件

将组件拆分为更小的组件。

例如，参考如下 `Comment` 组件：

```js
function formatDate(date) {
	return date.toLocaleDateString()
}

function Comment(props) {
    return (
      <div className="Comment">
        <div className="UserInfo">
          <img className="Avatar" src={props.author.avatarUrl} alt={props.author.name} />
          <div className="UserInfo-name">{props.author.name}</div>
        </div>
        <div className="Comment-text">{props.text}</div>
        <div className="Comment-date">{formatDate(props.date)}</div>
      </div>
    )
}

const comment = {
    date: new Date(),
    text: 'I hope you enjoy learning React!',
    author: {
      name: 'Hello Kitty',
      avatarUrl: 'http://placekitten.com/g/64/64',
    },
}

ReactDOM.render(
<Comment date={comment.date} text={comment.text} author={comment.author} />,
document.getElementById('test'),
)
```

**[在 CodePen 上试试](https://codepen.io/gaearon/pen/VKQwEo?editors=1010)**

该组件用于描述一个社交媒体网站上的评论功能，它接收 `author`（对象），`text` （字符串）以及 `date`（日期）作为 props。

![image-20221023135735919](https://i0.hdslb.com/bfs/album/09cf8c18c6f756985e0fc0e4436b075fe02b8027.png)

该组件由于嵌套的关系，变得难以维护，且很难复用它的各个部分。因此，让我们从中提取一些组件出来。

首先，我们将提取 `Avatar` 组件：

```js
function Avatar(props) {
  return (
    <img className="Avatar"      src={props.user.avatarUrl}      alt={props.user.name}    />  );
}
```

`Avatar` 不需知道它在 `Comment` 组件内部是如何渲染的。因此，我们给它的 props 起了一个更通用的名字：`user`，而不是 `author`。

我们建议从组件自身的角度命名 props，而不是依赖于调用组件的上下文命名。

我们现在针对 `Comment` 做些微小调整：

```js
  function Avatar(props) {
    return <img className="Avatar" src={props.user.avatarUrl} alt={props.user.name} />
  }

  function Comment(props) {
    return (
      <div className="Comment">
        <div className="UserInfo">
          <Avatar user={props.author} />
          <div className="UserInfo-name">{props.author.name}</div>
        </div>
        <div className="Comment-text">{props.text}</div>
        <div className="Comment-date">{formatDate(props.date)}</div>
      </div>
    )
  }
```

接下来，我们将提取 `UserInfo` 组件，该组件在用户名旁渲染 `Avatar` 组件：

```js
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```

进一步简化 `Comment` 组件：

```js
  function Avatar(props) {
    return <img className="Avatar" src={props.user.avatarUrl} alt={props.user.name} />
  }

  function UserInfo(props) {
    return (
      <div className="UserInfo">
        <Avatar user={props.user} />
        <div className="UserInfo-name">{props.user.name}</div>
      </div>
    )
  }

  function Comment(props) {
    return (
      <div className="Comment">
        <UserInfo user={props.author} />
        <div className="Comment-text">{props.text}</div>
        <div className="Comment-date">{formatDate(props.date)}</div>
      </div>
    )
  }
```

**[在 CodePen 上试试](https://codepen.io/gaearon/pen/rrJNJY?editors=1010)**

最初看上去，提取组件可能是一件繁重的工作，但是，在大型应用中，构建可复用组件库是完全值得的。根据经验来看，如果 UI 中有一部分被多次使用（`Button`，`Panel`，`Avatar`），或者组件本身就足够复杂（`App`，`FeedStory`，`Comment`），那么它就是一个可提取出独立组件的候选项。

## 组件实例的三大属性 state props refs

## 2.state

### 2.1 基本使用

> 我们都说React是一个状态机，体现是什么地方呢，就是体现在state上，通过与用户的交互，实现不同的状态，然后去渲染UI,这样就让用户的数据和界面保持一致了。state是组件的私有属性。
>
> 在React中，更新组件的state，结果就会重新渲染用户界面(不需要操作DOM),一句话就是说，用户的界面会随着状态的改变而改变。
>
> state是组件对象最重要的属性，值是对象（可以包含多个key-value的组合）
>
> 简单的说就是组件的状态，也就是该组件所存储的数据

**案例**：

需求：页面显示【今天天气很炎热】，鼠标点击文字的时候，页面更改为【今天天气很凉爽】

核心代码如下：

````js
<body>
    <!-- 准备好容器 -->
    <div id="test">
        
    </div>
</body>

<!--这里使用了js来创建虚拟DOM-->
<script type="text/babel">
        //1.创建组件
        class St extends React.Component{
            constructor(props){
                super(props);
                //先给state赋值
                this.state = {isHot:true,win:"ss"};
                //找到原型的dem，根据dem函数创建了一个dem1的函数，并且将实例对象的this赋值过去
                this.dem1 = this.dem.bind(this);
            }
            //render会调用1+n次【1就是初始化的时候调用的，n就是每一次修改state的时候调用的】
            render(){ //这个This也是实例对象
                //如果加dem()，就是将函数的回调值放入这个地方
                //this.dem这里面加入this，并不是调用，只不过是找到了dem这个函数，在调用的时候相当于直接调用，并不是实例对象的调用
                return <h1 onClick = {this.dem1}>今天天气很{this.state.isHot?"炎热":"凉爽"}</h1>    
            }
            //通过state的实例调用dem的时候，this就是实例对象
            dem(){
                const state =  this.state.isHot;
                 //状态中的属性不能直接进行更改，需要借助API
                // this.state.isHot = !isHot; 错误
                //必须使用setState对其进行修改，并且这是一个合并
                this.setState({isHot:!state});
            }
        }
        // 2.渲染，如果有多个渲染同一个容器，后面的会将前面的覆盖掉
        ReactDOM.render(<St/>,document.getElementById("test"));
</script>
````

在**类式组件**的函数中，直接修改`state`值

```js
this.state.isHot = false
```

> 页面的渲染靠的是`render`函数

这时候会发现页面内容不会改变，原因是 React 中不建议 `state`不允许直接修改，而是通过类的原型对象上的方法 `setState()`

**注意：**

1. 组件的构造函数，必须要传递一个props参数

2. 特别关注this【重点】，类中所有的方法局部都开启了严格模式，如果直接进行调用，this就是undefined

3. 想要改变state,需要使用setState进行修改，如果只是修改state的部分属性，则不会影响其他的属性，这个只是合并并不是覆盖。

**在优化过程中遇到的问题**

1. 组件中的 render 方法中的 this 为组件实例对象
2. 组件自定义方法中由于开启了严格模式，this 指向`undefined`如何解决
   1. 通过 bind 改变 this 指向
   2. 推荐采用箭头函数，箭头函数的 `this` 指向
3. state 数据不能直接修改或者更新

### 2.2 setState()

this.setState()，该方法接收两种参数：对象或函数。

```js
this.setState(partialState, [callback]);
```

- `partialState`: 需要更新的状态的部分对象
- `callback`: 更新完状态后的回调函数

**有两种写法:**

1. 对象：即想要修改的state

   ```js
   this.setState({
       isHot: false
   })
   ```

2. 函数：接收两个函数，第一个函数接受两个参数，第一个是当前state，第二个是当前props，该函数返回一个对象，和直接传递对象参数是一样的，就是要修改的state；第二个函数参数是state改变后触发的回调

```js
this.setState(state => ({count: state.count+1});
```

- 在执行 `setState`操作后，React 会自动调用一次 `render()`
- `render()` 的执行次数是 1+n (1 为初始化时的自动调用，n 为状态更新的次数)

### 2.3 简化版本

1. state的赋值可以不再构造函数中进行 （指state要更新的值的传值，上面代码onclick为一个函数，下面例子onclick可以自由传值）

2. 使用了箭头函数，将this进行了改变

```js
<body>
    <!-- 准备好容器 -->
    <div id="test">
        
    </div>
</body>

<script type="text/babel">
        class St extends React.Component{
            //可以直接对其进行赋值
            state = {isHot:true};
            render(){ //这个This也是实例对象
                return <h1 onClick = {this.dem}>今天天气很{this.state.isHot?"炎热":"凉爽"}</h1>    
                //或者使用{()=>this.dem()也是可以的}
            }
            //箭头函数 [自定义方法--->要用赋值语句的形式+箭头函数]
            dem = () =>{
                console.log(this);
                const state =  this.state.isHot;
                this.setState({isHot:!state});
            }
        }
        ReactDOM.render(<St />,document.getElementById("test"));       
</script>
```

如果想要在调用方法的时候传递参数，有两个方法：

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

上述两种方式是等价的，分别通过[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)和 [`Function.prototype.bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) 来实现。

在这两种情况下，React 的事件对象 `e` 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递，而通过 `bind` 的方式，事件对象以及更多的参数将会被隐式的进行传递。

### 2.4 State 的更新可能是异步的

**React控制之外的事件中调用setState是同步更新的。比如原生js绑定的事件，setTimeout/setInterval等**。

> 18版本中测试setTimeout回调函数中也是异步更新的

**大部分开发中用到的都是React封装的事件，比如onChange、onClick、onTouchMove等，这些事件处理程序中的setState都是异步处理的。**

```js
//1.创建组件
class St extends React.Component{
    //可以直接对其进行赋值
    state = {isHot:10};
    render(){ //这个This也是实例对象
        return <h1 onClick = {this.dem}>点击事件</h1> 
    }
//箭头函数 [自定义方法--->要用赋值语句的形式+箭头函数]
    dem = () =>{
        //修改isHot
        this.setState({ isHot: this.state.isHot + 1})
        console.log(this.state.isHot);
    }
}
```

上面的案例中预期setState使得isHot变成了11，输出也应该是11。然而在控制台打印的却是10，也就是并没有对其进行更新。这是因为异步的进行了处理，在输出的时候还没有对其进行处理。

```js
document.getElementById("test").addEventListener("click",()=>{
        this.setState({isHot: this.state.isHot + 1});
        console.log(this.state.isHot);
    })
}
```

但是通过这个原生JS的，可以发现，控制台打印的就是11，也就是已经对其进行了处理。也就是进行了同步的更新。

**React怎么调用同步或者异步的呢？**

在 React 的 setState 函数实现中，会根据一个变量 isBatchingUpdates 判断是直接更新 this.state 还是放到队列中延时更新，而 isBatchingUpdates 默认是 false，表示 setState 会同步更新 this.state；但是，有一个函数 batchedUpdates，该函数会把 isBatchingUpdates 修改为 true，而当 React 在调用事件处理函数之前就会先调用这个 batchedUpdates将isBatchingUpdates修改为true，这样由 React 控制的事件处理过程 setState 不会同步更新 this.state。

**如果是同步更新，每一个setState对调用一个render，并且如果多次调用setState会以最后调用的为准，前面的将会作废；如果是异步更新，多个setSate会统一调用一次render**

```js
dem = () =>{
    this.setState({
        isHot:  1,
        cont:444
    })
    this.setState({
    	isHot: this.state.isHot + 1

    })
    this.setState({
        isHot:  888,
        cont:888
    })
}
```

上面的最后会输出：isHot是888，cont是888

```js
 dem = ()=> {  
    this.setState({
        isHot: this.state.isHot + 1,

    })
    this.setState({
        isHot: this.state.isHot + 1,

    })
    this.setState({
        isHot: this.state.isHot + 888
    })
}
```

初始isHot为10，最后isHot输出为898，也就是前面两个都没有执行。

**注意！！这是异步更新才有的，如果同步更新，每一次都会调用render，这样每一次更新都会 **

### 2.5 异步更新解决方案

出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。

因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

例如，此代码可能会无法更新计数器：

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数：

```js
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

上面使用了[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，不过使用普通的函数也同样可以：

```js
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### 2.6 数据是向下流动的

不管是父组件或是子组件都无法知道某个组件是有状态的还是无状态的，并且它们也并不关心它是函数组件还是 class 组件。

这就是为什么称 state 为局部的或是封装的的原因。除了拥有并设置了它的组件，其他组件都无法访问。

组件可以选择把它的 state 作为 props 向下传递到它的子组件中：

```js
<FormattedDate date={this.state.date} />
```

`FormattedDate` 组件会在其 props 中接收参数 `date`，但是组件本身无法知道它是来自于 `Clock` 的 state，或是 `Clock` 的 props，还是手动输入的：

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/zKRqNB?editors=0010)

这通常会被叫做“自上而下”或是“单向”的数据流。任何的 state 总是所属于特定的组件，而且从该 state 派生的任何数据或 UI 只能影响树中“低于”它们的组件。

如果你把一个以组件构成的树想象成一个 props 的数据瀑布的话，那么每一个组件的 state 就像是在任意一点上给瀑布增加额外的水源，但是它只能向下流动。

为了证明每个组件都是真正独立的，我们可以创建一个渲染三个 `Clock` 的 `App` 组件：

```js
function App() {
  return (
    <div>
      <Clock />      
      <Clock />      
      <Clock />    
     </div>
  );
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/vXdGmd?editors=0010)

每个 `Clock` 组件都会单独设置它自己的计时器并且更新它。

在 React 应用中，组件是有状态组件还是无状态组件属于组件实现的细节，它可能会随着时间的推移而改变。你可以在有状态的组件中使用无状态的组件，反之亦然。

## 3.props

### 3.1 基本使用

与`state`不同，`state`是组件自身的状态，而`props`则是外部传入的数据

基本使用：

```js
<body>
    <div id = "div">

    </div>

</body>
<script type="text/babel">
    class Person extends React.Component{
        render(){
          const { name, age, sex } = this.props
            return (
                <ul>
                  <li>姓名：{name}</li>
                  <li>性别：{sex}</li>
                  <li>年龄：{age + 1}</li>
                </ul>
          )
        }
    }
    //传递数据
    ReactDOM.render(<Person name="tom" age = {41} sex="男"/>,document.getElementById("div"));
</script>
```

如果传递的数据是一个对象，可以更加简便的使用

```js
<script type="text/babel">
    class Person extends React.Component{
        render(){
            return (
                <ul>
                    <li>{this.props.name}</li>
                    <li>{this.props.age}</li>
                    <li>{this.props.sex}</li>
                </ul>
            )
        }
    }
    const p = {name:"张三",age:"18",sex:"女"}
   ReactDOM.render(<Person {...p}/>,document.getElementById("div"));
</script>
```

... 这个符号恐怕都不陌生，这个是一个展开运算符，主要用来展开数组，如下面这个例子：

```js
arr = [1,2,3];
arr1 = [4,5,6];
arr2 = [...arr,...arr1];  //arr2 = [1,2,3,4,5,6]
```

但是他还有其他的用法：

1.复制一个对象给另一个对象{...对象名}。此时这两个对象并没有什么联系了

```js
const p1 = {name:"张三",age:"18",sex:"女"}
const p2 = {...p1};
p1.name = "sss";
console.log(p2)  //{name:"张三",age:"18",sex:"女"}
```

2.在复制的时候，合并其中的属性

```js
 const p1 = {name:"张三",age:"18",sex:"女"}
 const p2 = {...p1,name : "111",hua:"ss"};
 p1.name = "sss";
 console.log(p2)  //{name: "111", age: "18", sex: "女",hua:"ss"}
```

**注意！！** **{...P}并不能展开一个对象**

**props传递一个对象，是因为babel+react使得{..p}可以展开对象，但是只有在标签中才能使用**

### 3.2 props限制

> 注意：
>
> 自 React v15.5 起，`React.PropTypes` 已移入另一个包中。请使用 [`prop-types` 库](https://www.npmjs.com/package/prop-types) 代替。
>
> 我们提供了一个 [codemod 脚本](https://zh-hans.reactjs.org/blog/2017/04/07/react-v15.5.0.html#migrating-from-reactproptypes)来做自动转换。

随着你的应用程序不断增长，你可以通过类型检查捕获大量错误。对于某些应用程序来说，你可以使用 [Flow](https://flow.org/) 或 [TypeScript](https://www.typescriptlang.org/) 等 JavaScript 扩展来对整个应用程序做类型检查。但即使你不使用这些扩展，React 也内置了一些类型检查的功能。要在组件的 props 上进行类型检查，你只需配置特定的 `propTypes` 属性：

react对此提供了相应的解决方法：

- propTypes:类型检查，还可以限制不能为空
- defaultProps：默认值

> 从 ES2022 开始，你也可以在 React 类组件中将 `defaultProps` 声明为静态属性。欲了解更多信息，请参阅 [class public static fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields#public_static_fields)。这种现代语法需要添加额外的编译步骤才能在老版浏览器中工作。

```js

<!-- 准备好一个“容器” -->
<div id="test1"></div>
<div id="test2"></div>
<div id="test3"></div>

<script type="text/babel">
    //创建组件
    class Person extends React.Component{
        render(){
            // console.log(this);
            const {name,age,sex} = this.props
            //props是只读的
            //this.props.name = 'jack' //此行代码会报错，因为props是只读的
            return (
                <ul>
                    <li>姓名：{name}</li>
                    <li>性别：{sex}</li>
                    <li>年龄：{age+1}</li>
                </ul>
            )
        }
    }
    //对标签属性进行类型、必要性的限制
    Person.propTypes = {
        name:PropTypes.string.isRequired, //限制name必传，且为字符串
        sex:PropTypes.string,//限制sex为字符串
        age:PropTypes.number,//限制age为数值
        speak:PropTypes.func,//限制speak为函数
    }
    //指定默认标签属性值
    Person.defaultProps = {
        sex:'男',//sex默认值为男
        age:18 //age默认值为18
    }
    //渲染组件到页面
    ReactDOM.render(<Person name={100} speak={speak}/>,document.getElementById('test1'))
    ReactDOM.render(<Person name="tom" age={18} sex="女"/>,document.getElementById('test2'))

    const p = {name:'老刘',age:18,sex:'女'}
    // console.log('@',...p);
    // ReactDOM.render(<Person name={p.name} age={p.age} sex={p.sex}/>,document.getElementById('test3'))
    ReactDOM.render(<Person {...p}/>,document.getElementById('test3'))

    function speak(){
        console.log('我说话了');
    }
</script>
```

当传入的 `prop` 值类型不正确时，JavaScript 控制台将会显示警告。出于性能方面的考虑，`propTypes` 仅在开发模式下进行检查。

`defaultProps` 用于确保 `this.props.sex` 在父组件没有指定其值时，有一个默认值。`propTypes` 类型检查发生在 `defaultProps` 赋值后，所以类型检查也适用于 `defaultProps`。

**PropTypes**

以下提供了使用不同验证器的例子：

```js
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  // 这些属性都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // 具有额外属性警告的对象
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的必需数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `oneOfType` 中不会起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

**限制单个元素**

你可以通过 `PropTypes.element` 来确保传递给组件的 children 中只包含一个元素。

```js
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // 这必须只有一个元素，否则控制台会打印警告。
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

### 3.3 简写方式

```js
<!-- 准备好一个“容器” -->
<div id="test1"></div>
<div id="test2"></div>
<div id="test3"></div>


<script type="text/babel">
    //创建组件
    class Person extends React.Component{

        constructor(props){
            //构造器是否接收props，是否传递给super，取决于：是否希望在构造器中通过this访问props
            // console.log(props);
            super(props)
            console.log('constructor',this.props);
        }

        //对标签属性进行类型、必要性的限制
        static propTypes = {
            name:PropTypes.string.isRequired, //限制name必传，且为字符串
            sex:PropTypes.string,//限制sex为字符串
            age:PropTypes.number,//限制age为数值
        }

        //指定默认标签属性值
        static defaultProps = {
            sex:'男',//sex默认值为男
            age:18 //age默认值为18
        }

        render(){
            // console.log(this);
            const {name,age,sex} = this.props
            //props是只读的
            //this.props.name = 'jack' //此行代码会报错，因为props是只读的
            return (
                <ul>
                    <li>姓名：{name}</li>
                    <li>性别：{sex}</li>
                    <li>年龄：{age+1}</li>
                </ul>
            )
        }
    }

    //渲染组件到页面
    ReactDOM.render(<Person name="jerry"/>,document.getElementById('test1'))
</script>
```

在使用的时候可以通过 `this.props`来获取值 类式组件的 `props`:

1. 通过在组件标签上传递值，在组件中就可以获取到所传递的值
2. 在构造器里的`props`参数里可以获取到 `props`
3. 可以分别设置 `propTypes` 和 `defaultProps` 两个属性来分别操作 `props`的规范和默认值，两者都是直接添加在类式组件的**原型对象**上的（所以需要添加 `static`）
4. 同时可以通过`...`运算符来简化

### 3.4 函数式组件的使用

> 函数在使用props的时候，是作为参数进行使用的(props)

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>对props进行限制</title>
  </head>
  <body>
    <!-- 准备好一个“容器” -->
    <div id="test1"></div>

    <script type="text/babel">
      //创建组件
      function Person(props) {
        const { name, age, sex } = props
        return (
          <ul>
            <li>姓名：{name}</li>
            <li>性别：{sex}</li>
            <li>年龄：{age}</li>
          </ul>
        )
      }
      Person.propTypes = {
        name: PropTypes.string.isRequired, //限制name必传，且为字符串
        sex: PropTypes.string, //限制sex为字符串
        age: PropTypes.number, //限制age为数值
      }

      //指定默认标签属性值
      Person.defaultProps = {
        sex: '男', //sex默认值为男
        age: 18, //age默认值为18
      }
      //渲染组件到页面
      ReactDOM.render(<Person name="jerry" />, document.getElementById('test1'))
    </script>
  </body>
</html>
```

函数组件的 `props`定义:

1. 在组件标签中传递 `props`的值
2. 组件函数的参数为 `props`
3. 对 `props`的限制和默认值同样设置在原型对象上

### 3.5 props 的只读性

组件无论是使用[函数声明还是通过 class 声明](https://zh-hans.reactjs.org/docs/components-and-props.html#function-and-class-components)，都绝不能修改自身的 props。来看下这个 `sum` 函数：

```js
function sum(a, b) {
  return a + b;
}
```

这样的函数被称为[“纯函数”](https://en.wikipedia.org/wiki/Pure_function)，因为该函数不会尝试更改入参，且多次调用下相同的入参始终返回相同的结果。

相反，下面这个函数则不是纯函数，因为它更改了自己的入参：

```js
function withdraw(account, amount) {
  account.total -= amount;
}
```

React 非常灵活，但它也有一个严格的规则：

**所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。**

当然，应用程序的 UI 是动态的，并会伴随着时间的推移而变化。`state`在不违反上述规则的情况下，state 允许 React 组件随用户操作、网络响应或者其他变化而动态更改输出内容。

## 4.refs

Refs 提供了一种方式，允许我们访问 DOM 节点或在 `render` 方法中创建的 React 元素。

在典型的 React 数据流中，[props](https://zh-hans.reactjs.org/docs/components-and-props.html) 是父组件与子组件交互的唯一方式。要修改一个子组件，你需要使用新的 props 来重新渲染它。但是，在某些情况下，你需要在典型数据流之外强制修改子组件。被修改的子组件可能是一个 React 组件的实例，也可能是一个 DOM 元素。对于这两种情况，React 都提供了解决办法。

> 在我们正常的操作节点时，需要采用DOM API 来查找元素，但是这样违背了 React 的理念，因此有了`refs`

**何时使用 Refs**

下面是几个适合使用 refs 的情况：

- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

避免使用 refs 来做任何可以通过声明式实现来完成的事情。

**有三种操作`refs`的方法，分别为：**

- 字符串形式
- 回调形式
- `createRef`形式

**勿过度使用 Refs**

你可能首先会想到使用 refs 在你的 app 中“让事情发生”。如果是这种情况，请花一点时间，认真再考虑一下 state 属性应该被安排在哪个组件层中。通常你会想明白，让更高的组件层级拥有这个 state，是更恰当的。查看 [状态提升](https://zh-hans.reactjs.org/docs/lifting-state-up.html) 以获取更多有关示例。
p.s. 状态提升就是指A子组件需要用到B子组件state进行展示时，把该state移到父组件中。

### 4.1 字符串形式

在想要获取到一个DOM节点，可以直接在这个节点上添加ref属性。利用该属性进行获取该节点的值。

案例：给需要的节点添加ref属性，此时该实例对象的refs上就会有这个值。就可以利用实例对象的refs获取已经添加节点的值

```js
<input ref="dian" type="text" placeholder="点击弹出" />

inputBlur = () =>{
            alert(this.refs.shiqu.value);
}
```

**注意**

不建议使用它，因为 string 类型的 refs 存在 [一些问题](https://github.com/facebook/react/pull/8333#issuecomment-271648615)。它已过时并可能会在未来的版本被移除。

如果你目前还在使用 `this.refs.textInput` 这种方式访问 refs ，我们建议用[回调函数](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs)或 [`createRef` API](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#creating-refs) 的方式代替。

### 4.2 回调形式

React 也支持另一种设置 refs 的方式，称为“回调 refs”。它能助你更精细地控制何时 refs 被设置和解除。

这种方式会将该DOM作为参数传递过去。

组件实例的`ref`属性传递一个回调函数`c => this.input1 = c `（箭头函数简写），这样会在实例的属性中存储对DOM节点的引用，使用时可通过`this.input1`来使用

```js
<input ref={e => this.input1 = e } type="text" placeholder="点击按钮提示数据"/>
```

`e`会接收到当前节点作为参数，然后将当前节点赋值给实例的`input1`属性上面

**关于回调 refs 的说明**

如果 `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

```js
class Demo extends React.Component {
    state = { isHot: false }

    changeWeather = () => {
      //获取原来的状态
      const { isHot } = this.state
      //更新状态
      this.setState({ isHot: !isHot })
    }

    render() {
      const { isHot } = this.state
      return (
        <div>
          <h2>今天天气很{isHot ? '炎热' : '凉爽'}</h2>
          <input
            ref={c => {
              this.input1 = c
              console.log('@', c)
            }}
            type="text"
          />
          <br />
          <br />
          <button onClick={this.changeWeather}>点我切换天气</button>
        </div>
      )
    }
}
```

刚渲染完会调用一次

![image-20221023153439400](https://i0.hdslb.com/bfs/album/40ec77c4a5ab8d3ca9bcb67eb2f1a2e80bc8d2ed.png)

触发模板更新会调用两次

![image-20221023153510564](https://i0.hdslb.com/bfs/album/6caaacc85071b8b9ed0040e1969be621b4795a61.png)

第一次传递一个null值把之前的属性清空，再重新赋值。

如果不想总是这样重新创建新的函数，可以使用下面的方案

下面的例子描述了一个通用的范例：使用 `ref` 回调函数，在实例的属性中存储对 DOM 节点的引用。

```js
//创建组件
class Demo extends React.Component {
    state = { isHot: false }
	// 在实例上面创建一个函数
    setTextInputRef = e => {
      this.input1 = e
    }

    changeWeather = () => {
      console.log(this.input1)
      //获取原来的状态
      const { isHot } = this.state
      //更新状态
      this.setState({ isHot: !isHot })
    }

    render() {
      const { isHot } = this.state
      return (
        <div>
          <h2>今天天气很{isHot ? '炎热' : '凉爽'}</h2>
          <input ref={this.setTextInputRef} type="text" />
          <br />
          <button onClick={this.changeWeather}>点我切换天气</button>
        </div>
      )
    }
}
```

React 将在组件挂载时，会调用 `ref` 回调函数并传入 DOM 元素，当卸载时调用它并传入 `null`。

你可以在组件间传递回调形式的 refs，就像你可以传递通过 `React.createRef()` 创建的对象 refs 一样。

```js
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />    
     </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}      />
    );
  }
}
```

在上面的例子中，`Parent` 把它的 refs 回调函数当作 `inputRef` props 传递给了 `CustomTextInput`，而且 `CustomTextInput` 把相同的函数作为特殊的 `ref` 属性传递给了 `<input>`。结果是，在 `Parent` 中的 `this.inputElement` 会被设置为与 `CustomTextInput` 中的 `input` 元素相对应的 DOM 节点。

### 4.3 createRef 形式（推荐写法）

**创建 Refs**

Refs 是使用 `React.createRef()` 创建的，并通过 `ref` 属性附加到 React 元素。在构造组件时，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

**访问 Refs**

当 ref 被传递给 `render` 中的元素时，对该节点的引用可以在 ref 的 `current` 属性中被访问。

```js
const node = this.myRef.current;
```

ref 的值根据节点的类型而有所不同：

- 当 `ref` 属性用于 HTML 元素时，构造函数中使用 `React.createRef()` 创建的 `ref` 接收底层 DOM 元素作为其 `current` 属性。
- 当 `ref` 属性用于自定义 class 组件时，`ref` 对象接收组件的挂载实例作为其 `current` 属性。
- **你不能在函数组件上使用 `ref` 属性**，因为他们没有实例。

### 4.4 为 DOM 元素添加 ref

以下代码使用 `ref` 去存储 DOM 节点的引用：

```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef();    
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // 直接使用原生 API 使 text 输入框获得焦点
    // 注意：我们通过 "current" 来访问 DOM 节点
    this.textInput.current.focus();  
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到
    // 构造器里创建的 `textInput` 上
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} 
        />        
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

React 会在组件挂载时给 `current` 属性传入 DOM 元素，并在组件卸载时传入 `null` 值。`ref` 会在 `componentDidMount` 或 `componentDidUpdate` 生命周期钩子触发前更新。

注意：我们不要过度的使用 ref，如果发生时间的元素刚好是需要操作的元素，就可以使用事件对象去替代。

❕试一下进入页面自动对焦input

### 4.5 为 class 组件添加 Ref

如果我们想包装上面的 `CustomTextInput`，来模拟它挂载之后立即被点击的操作，我们可以使用 ref 来获取这个自定义的 input 组件并手动调用它的 `focusTextInput` 方法：

```js
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();  
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();  
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />    
    );
  }
}
```

请注意，这仅在 `CustomTextInput` 声明为 class 时才有效：

```js
class CustomTextInput extends React.Component {  // ...
}
```

### 4.6 Refs 与函数组件

默认情况下，**你不能在函数组件上使用 `ref` 属性**，因为它们没有实例：
（指自定义函数组件没有ref属性）

```js
function MyFunctionComponent() {
  return <input />;
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }
  render() {
    // This will *not* work!
    return (
      <MyFunctionComponent ref={this.textInput} />
    );
  }
}
```

如果要在函数组件中使用 `ref`，你可以使用 [`forwardRef`](https://zh-hans.reactjs.org/docs/forwarding-refs.html)（可与 [`useImperativeHandle`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle) 结合使用），或者可以将该组件转化为 class 组件。

不管怎样，你可以**在函数组件内部使用 `ref` 属性**，只要它指向一个 DOM 元素或 class 组件：

```js
function CustomTextInput(props) {
  // 这里必须声明 textInput，这样 ref 才可以引用它
  const textInput = useRef(null);

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```


# 03 【事件处理】

> React的事件是通过onXxx属性指定事件处理函数
>
> React 使用的是自定义事件，而不是原生的 DOM 事件
>
> React 的事件是通过事件委托方式处理的（为了更加的高效）
>
> 可以通过事件的 `event.target`获取发生的 DOM 元素对象，可以尽量减少 `refs`的使用
>
> 事件中必须返回的是函数

## 1.React事件

React 元素的事件处理和 DOM 元素的很相似，但是有一点语法上的不同：

- React 事件的命名采用小驼峰式（camelCase），而不是纯小写。
- 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。

例如，传统的 HTML：

```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

在 React 中略微不同：

```html
<button onClick={activateLasers}>  
   Activate Lasers
</button>
```

在 React 中另一个不同点是你不能通过返回 `false` 的方式阻止默认行为。你必须显式地使用 `preventDefault`。例如，传统的 HTML 中阻止表单的默认提交行为，你可以这样写：

```js
<form onsubmit="console.log('You clicked submit.'); return false">
  <button type="submit">Submit</button>
</form>
```

在 React 中，可能是这样的：

```js
function Form() {
  function handleSubmit(e) {
    e.preventDefault();    
    console.log('You clicked submit.');
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```

在这里，`e` 是一个合成事件。React 根据 [W3C 规范](https://www.w3.org/TR/DOM-Level-3-Events/)来定义这些合成事件，所以你不需要担心跨浏览器的兼容性问题。React 事件与原生事件不完全相同。如果想了解更多，请查看 [`SyntheticEvent`](https://zh-hans.reactjs.org/docs/events.html) 参考指南。

使用 React 时，你一般不需要使用 `addEventListener` 为已创建的 DOM 元素添加监听器。事实上，你只需要在该元素初始渲染的时候添加监听器即可。

## 2.类式组件绑定事件

当你使用 [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) 语法定义一个组件的时候，通常的做法是将事件处理函数声明为 class 中的方法。例如，下面的 `Toggle` 组件会渲染一个让用户切换开关状态的按钮：

```js
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};
    // 为了在回调中使用 `this`，这个绑定是必不可少的    	this.handleClick = this.handleClick.bind(this);  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }
      
  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/xEmzGg?editors=0010)

你必须谨慎对待 JSX 回调函数中的 `this`，在 JavaScript 中，class 的方法默认不会[绑定](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind) `this`。如果你忘记绑定 `this.handleClick` 并把它传入了 `onClick`，当你调用这个函数的时候 `this` 的值为 `undefined`。

这并不是 React 特有的行为；这其实与 [JavaScript 函数工作原理](https://www.smashingmagazine.com/2014/01/understanding-javascript-function-prototype-bind/)有关。通常情况下，如果你没有在方法后面添加 `()`，例如 `onClick={this.handleClick}`，你应该为这个方法绑定 `this`。

如果觉得使用 `bind` 很麻烦，这里有两种方式可以解决。你可以使用 [public class fields 语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields#public_instance_fields) to correctly bind callbacks:

```js
class LoggingButton extends React.Component {
  // class fields语法 This syntax ensures `this` is bound within handleClick.
  handleClick = () => {
    console.log('this is:', this);
  };
  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

[Create React App](https://github.com/facebookincubator/create-react-app) 默认启用此语法。

如果你没有使用 class fields 语法，你可以在回调中使用[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)：

```js
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
}
```

此语法问题在于每次渲染 `LoggingButton` 时都会创建不同的回调函数。在大多数情况下，这没什么问题，但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。我们通常建议在构造器中绑定或使用 class fields 语法来避免这类性能问题。

## 3.向事件处理程序传递参数

在循环中，通常我们会为事件处理函数传递额外的参数。例如，若 `id` 是你要删除那一行的 ID，以下两种方式都可以向事件处理函数传递参数：

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

上述两种方式是等价的，分别通过[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)和 [`Function.prototype.bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) 来实现。

在这两种情况下，React 的事件对象 `e` 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递，而通过 `bind` 的方式，事件对象以及更多的参数将会被隐式的进行传递。

## 4.收集表单数据

首先我们先来创建一个简单的表单组件：

```jsx
import React from 'react';

const MyForm = () => {
    return (
        <form>
            <div>
                用户名 <input type="text"/>
            </div>
            <div>
                密码 <input type="password"/>
            </div>
            <div>
                电子邮件 <input type="email"/>
            </div>

            <div>
                <button>提交</button>
            </div>
        </form>
    );
};

export default MyForm;
```

首先使用React定义表单和之前传统网页中的表单有一些区别，传统网页中form需要指定action和method两个属性，而表单项也必须要指定name属性，这些属性都是提交表单所必须的。但是在React中定义表单时，这些属性通通都可以不指定，因为React中的表单所有的功能都需要通过代码来控制，包括获取表单值和提交表单，所以这些东西都可以在函数中指定并通过AJAX发送请求，无需直接在表单中设置。

首先我们来研究一下如何获取表单中的用户所填写的内容，要获取用户所填写的内容我们必须要监听表单onChange事件，在表单项发生变化时获取其中的内容，在响应函数中通过事件对象的target.value来获取用户填写的内容。事件响应函数大概是这个样子：

```jsx
const nameChangeHandler= e => {
     //e.target.value 表示当前用户输入的值
};
```

然后我们再将该函数设置为input元素的onChange事件的响应函数：

```jsx
<div>
    用户名 <input type="text" onChange={nameChangeHandler}/>
</div>
```

这样一来当用户输入内容时，nameChangeHandler就会被触发，从而通过e.target.value来获取用户输入的值。通常我们还会为表单项创建一个state用来存储值：

```jsx
const [inputName, setInputName] = useState(''); 
const nameChangeHandler = e => {
    //e.target.value 表示当前用户输入的值
    setInputName(e.target.value);
 };
```

上例中用户名存储到了变量inputName中，inputName也会设置为对应表单项的value属性值，这样一来当inputName发生变化时，表单项中的内容也会随之改变：

```jsx
<div>
    用户名 <input type="text" onChange={nameChangeHandler} value={inputName}/>
</div>
```

如此设置后，当用户输入内容后会触发onChange事件从而调用nameChangeHandler函数，在函数内部调用了setInputName设置了用户输入的用户名。换句话说用户在表单中输入内容会影响到state的值，同时当我们修改state的值时，由于表单项的value属性值指向了state，表单也会随state值一起改变。这种绑定方式我们称为双向绑定，即表单会改变state，state也可以改变表单，在开发中使用双向绑定的表单项是最佳实践。

## 5.受控和非受控组件

先来说说受控组件：

使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。

```js
saveName = (event) =>{
    this.setState({name:event.target.value});
}

savePwd = (event) => {
    this.setState({pwd:event.target.value});
}

render() {
    return (
        <form action="http://www.baidu.com" onSubmit={this.login}>
            用户名：<input value={this.state.name} onChange={this.saveName} type = "text" />
            密码<input value={this.state.pwd} onChange={this.savePwd} type = "password"/>
            <button>登录</button>
        </form>
    )
}
```

由于在表单元素上设置了 `value` 属性，因此显示的值将始终为 `this.state.value`，这使得 React 的 state 成为唯一数据源。由于 `onchange` 在每次按键时都会执行并更新 React 的 state，因此显示的值将随着用户输入而更新。

对于受控组件来说，输入的值始终由 React 的 state 驱动。

**非受控组件：**

非受控组件其实就是表单元素的值不会更新state。输入数据都是现用现取的。

如下：下面并没有使用state来控制属性，使用的是事件来控制表单的属性值。

> 表单提交同样需要通过事件来处理，提交表单的事件通过form标签的onSubmit事件来绑定，处理表单的方式因情况而已，但是一定要注意，必须要取消默认行为，否则会触发表单的默认提交行为：

```js
class Login extends React.Component{

    login = (event) =>{
        event.preventDefault(); //阻止表单默认事件
            console.log(this.name.value);
            console.log(this.pwd.value);
        }
        render() {
            return (
                <form action="http://www.baidu.com" onSubmit={this.login}>
                用户名：<input ref = {e => this.name =e } type = "text" name ="username"/>
                密码：  <input ref = {e => this.pwd =e } type = "password" name ="password"/>
                <button>登录</button>
                </form>
            )
    }
}
```

## 5.函数的柯里化（currying）

**高级函数**

1. 如果函数的参数是函数

2. 如果函数返回一个函数

通过函数调用继续返回函数的方式，实现多次接收参数最后统一处理的函数编码形式

如下，我们将上面的案例简化，创建高级函数：

```js
 class Login extends React.Component{
    state = {name:"",pwd:""};

    //返回一个函数
    saveType = (type) =>{
        return (event) => {
            this.setState({[type]:event.target.value});
        }
    }

    //因为事件中必须是一个函数，所以返回的也是一个函数，这样就符合规范了
    render() {
        return (
            <form>
                <input onChange = {this.saveType('name')} type = "text"/>
                <button>登录</button>
            </form>
        )
    }
}

ReactDOM.render(<Login />,document.getElementById("div"));
```

不使用函数柯里化

```js
 class Login extends React.Component{
    state = {name:"",pwd:""};

    //返回一个函数
    saveType = (type,event) =>{
        this.setState({[type]:event.target.value});
    }

    //因为事件中必须是一个函数，所以返回的也是一个函数，这样就符合规范了
    render() {
        return (
            <form>
                <input onChange = {event => this.saveType('name',event)} type = "text"/>
                <button>登录</button>
            </form>
        )
    }
}

ReactDOM.render(<Login />,document.getElementById("div"));
```
💻 面试题
请实现一个add函数实现以下功能 ：
add(1) // 1
add(1)(2) // 3
add(1)(2)(3) // 6
add(1)(2,3) // 6
add(1,2)(3) // 6
add(1,2,3) // 6

🌟 p.s. fn(a)(b)的意思是先执行fn(a), 其函数返回的函数bn来执行bn(b).
Hence fn() must return a function.
```js
// 函数柯里化，利用递归和闭包实现
const curry = function (fn) { // 接收一个函数
    const len = fn.length; // 初始函数fn的形参个数
    return function t() {
        const innerParamsLength = arguments.length; // t的实参个数
        const argsArr = Array.prototype.slice.call(arguments);

        if (innerParamsLength >= len) { // 递归出口，如果t实参个数不小于fn形参个数，终止递归
            return fn.apply(undefined, argsArr); // 执行改造后的函数
        } else { // 如果t实参个数小于fn形参个数，说明科里化没有完成，继续执行科里化
            return function () {
                const innerArgsArr = Array.prototype.slice.call(arguments);
                const allArgsArr = argsArr.concat(innerArgsArr);
                return t.apply(undefined, allArgsArr);
            }
        }
    }
};

function add(num1, num2, num3) {
    return num1 + num2 + num3
};

const finalFn = curry(add);
finalFn(1, 2, 3); // 6
finalFn(1)(2)(3); // 6
finalFn(1)(2, 3); // 6
```

# 04 【生命周期】

## 1.简介

组件从创建到死亡，会经过一些特定的阶段

 React组件中包含一系列钩子函数{生命周期回调函数}，会在特定的时刻调用

 我们在定义组件的时候，会在特定的声明周期回调函数中，做特定的工作

在 React 中为我们提供了一些生命周期钩子函数，让我们能在 React 执行的重要阶段，在钩子函数中做一些事情。那么在 React 的生命周期中，有哪些钩子函数呢，我们来总结一下

**react生命周期(旧)**

```js
1. 初始化阶段: 由ReactDOM.render()触发---初次渲染
                    1.	constructor()
                    2.	componentWillMount()
                    3.	render()
                    4.	componentDidMount() =====> 常用
                        一般在这个钩子中做一些初始化的事，例如：开启定时器、发送网络请求、订阅消息
2. 更新阶段: 由组件内部this.setSate()或父组件render触发
                    1.	shouldComponentUpdate()
                    2.	componentWillUpdate()
                    3.	render() =====> 必须使用的一个
                    4.	componentDidUpdate()
3. 卸载组件: 由ReactDOM.unmountComponentAtNode()触发
                    1.	componentWillUnmount()  =====> 常用
                        一般在这个钩子中做一些收尾的事，例如：关闭定时器、取消订阅消息
```

![react生命周期(旧)](https://i0.hdslb.com/bfs/album/eca620dfbbcdc3325be4a1f167f9a4ca2a0dfb7a.png)

在最新的react版本中，有些生命周期钩子被抛弃了，具体函数如下：

- `componentWillMount`
- `componentWillReceiveProps`
- `componentWillUpdate`

这些生命周期方法经常被误解和滥用；此外，我们预计，在异步渲染中，它们潜在的误用问题可能更大。我们将在即将发布的版本中为这些生命周期添加 “UNSAFE_” 前缀。（这里的 “unsafe” 不是指安全性，而是表示使用这些生命周期的代码在 React 的未来版本中更有可能出现 bug，尤其是在启用异步渲染之后。）

由此可见，新版本中并不推荐持有这三个函数，取而代之的是带有UNSAFE_ 前缀的三个函数，比如: UNSAFE_ componentWillMount。即便如此，其实React官方还是不推荐大家去使用，在以后版本中有可能会去除这几个函数。

**react生命周期(新)**

````js
1. 初始化阶段: 由ReactDOM.render()触发---初次渲染
                1.	constructor()
                2.	getDerivedStateFromProps 
                3.	render()
                4.	componentDidMount() =====> 常用
                	一般在这个钩子中做一些初始化的事，例如：开启定时器、发送网络请求、订阅消息
2. 更新阶段: 由组件内部this.setSate()或父组件重新render触发
                1.	getDerivedStateFromProps
                2.	shouldComponentUpdate()
                3.	render()
                4.	getSnapshotBeforeUpdate
                5.	componentDidUpdate()
3. 卸载组件: 由ReactDOM.unmountComponentAtNode()触发
                1.	componentWillUnmount()  =====> 常用
                	一般在这个钩子中做一些收尾的事，例如：关闭定时器、取消订阅消息
````

![image-20221023222949399](https://i0.hdslb.com/bfs/album/1ad3acfd13159cfdc364a487dfc4335f7a9a1a06.png)

## 2.初始化阶段

**在组件实例被创建并插入到dom中时，生命周期调用顺序如下**

**旧生命周期：**

1. constructor（props）
2. componentWillMount（）-------------可以用但是不建议使用
3. render（）
4. componentDidMount（）

**新生命周期：**

1. constructor（props）
2. `static getDerivedStateFromProps（props，state）`--替代了`componentWillReceiveProps`
3. render（）
4. componentDidMount（）

### 2.1 constructor

**数据的初始化。**

接收props和context，当想在函数内使用这两个参数需要在super传入参数，当使用constructor时必须使用super，否则可能会有this的指向问题，如果不初始化state或者不进行方法绑定，则可以不为组件实现构造函数；

避免将 props 的值复制给 state！这是一个常见的错误：

```js
constructor(props) {
 super(props);
 // 不要这样做
 this.state = { color: props.color };
}
```

如此做毫无必要（可以直接使用 this.props.color），同时还产生了 bug（更新 prop 中的 color 时，并不会影响 state）。

现在我们通常不会使用 `constructor` 属性，而是改用类加箭头函数的方法，来替代 `constructor`

例如，我们可以这样初始化 `state`

```js
state = {
	count: 0
};
```

### 2.2 componentWillMount（即将废弃）

**该方法只在挂载的时候调用一次，表示组件将要被挂载，并且在 `render` 方法之前调用。**

> 如果存在 `getDerivedStateFromProps` 和 `getSnapshotBeforeUpdate` 就不会执行生命周期`componentWillMount`。

​    在服务端渲染唯一会调用的函数，代表已经初始化数据但是没有渲染dom，因此在此方法中同步调用 `setState()` 不会触发额外渲染。

**这个方法在 React 18版本中将要被废弃，官方解释是在 React 异步机制下，如果滥用这个钩子可能会有 Bug**

### 2.3 static getDerivedStateFromProps（新钩子）

**从props获取state。**

替代了`componentWillReceiveProps，`此方法适用于[罕见的用例](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state)，即 state 的值在任何时候都取决于 props。

这个是 React 新版本中新增的2个钩子之一，据说很少用。

1. 首先，该函数会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用；

2. 该函数必须是静态的；

3. 给组件传递的数据（props）以及组件状态（state），会作为参数到这个函数中；

4. 该函数也必须有返回值，返回一个Null或者state对象。因为初始化和后续更新都会执行这个方法，因此在这个方法返回state对象，就相当于将原来的state进行了覆盖，所以倒是修改状态不起作用。

> 注意：`state` 的值在任何时候都取决于传入的 `props` ，不会再改变

如下

```js
static getDerivedStateFromProps(props, state) {
    return null											
}
ReactDOM.render(<Count count="109"/>,document.querySelector('.test'))
```

`count` 的值不会改变，一直是 109

> [React的生命周期 - 简书](https://www.jianshu.com/p/b331d0e4b398)
>
> 老版本中的componentWillReceiveProps()方法判断前后两个 props 是否相同，如果不同再将新的 props 更新到相应的 state 上去。这样做一来会破坏 state 数据的单一数据源，导致组件状态变得不可预测，另一方面也会增加组件的重绘次数。
>
> 这两者最大的不同就是:
> 在 componentWillReceiveProps 中，我们一般会做以下两件事，一是根据 props 来更新 state，二是触发一些回调，如动画或页面跳转等。
>
> 1. 在老版本的 React 中，这两件事我们都需要在 componentWillReceiveProps 中去做。
> 2. 而在新版本中，官方将更新 state 与触发回调重新分配到了 getDerivedStateFromProps 与 componentDidUpdate 中，使得组件整体的更新逻辑更为清晰。而且在 getDerivedStateFromProps 中还禁止了组件去访问 this.props，强制让开发者去比较 nextProps 与 prevState 中的值，以确保当开发者用到 getDerivedStateFromProps 这个生命周期函数时，就是在根据当前的 props 来更新组件的 state，而不是去做其他一些让组件自身状态变得更加不可预测的事情。

### 2.4 render

**class组件中唯一必须实现的方法。**

> render函数会插入jsx生成的dom结构，react会生成一份虚拟dom树，在每一次组件更新时，在此react会通过其diff算法比较更新前后的新旧DOM树，比较以后，找到最小的有差异的DOM节点，并重新渲染。

> 注意：避免在 `render` 中使用 `setState` ，否则会死循环

当render被调用时，他会检查this.props.和this.state的变化并返回以下类型之一：

1. 通过jsx创建的react元素
2.  数组或者fragments：使得render可以返回多个元素
3.   Portals:可以渲染子节点到不同的dom树上
4.   字符串或数值类型：他们在dom中会被渲染为文本节点
5.   布尔类型或者null：什么都不渲染

### 2.5 componentDidMount

**在组件挂在后（插入到dom树中）后立即调用**

`componentDidMount` 的执行意味着初始化挂载操作已经基本完成，它主要用于组件挂载完成后做某些操作

这个挂载完成指的是：组件插入 DOM tree

​    可以在这里调用Ajax请求，返回的数据可以通过setState使组件重新渲染，或者添加订阅，但是要在conponentWillUnmount中取消订阅

### 2.6 初始化阶段总结

执行顺序 `constructor` -> `getDerivedStateFromProps` 或者 `componentWillMount` -> `render` -> `componentDidMount`

![image-20221023223048451](https://i0.hdslb.com/bfs/album/ea2d0052b360a8aed3ea84796b601d118ce5be13.png)

## 3.更新阶段

**当组件的 props 或 state 发生变化时会触发更新。**

**旧生命周期：**

1. componentWillReceiveProps (nextProps)------------------可以用但是不建议使用

2. shouldComponentUpdate（nextProps,nextState）

3. componetnWillUpdate（nextProps,nextState）----------------可以用但是不建议使用

4. render（）

5. componentDidUpdate（prevProps,precState,snapshot）

**新生命周期：**

1. static getDerivedStateFromProps（nextProps, prevState）
2. shouldComponentUpdate（nextProps,nextState）
3. render（）
4. getSnapshotBeforeUpdate（prevProps,prevState）
5. componentDidUpdate（prevProps,precState,snapshot）

### 3.1 componentWillReceiveProps (即将废弃)

**在已挂载的组件接收新的props之前调用。**

通过对比nextProps和this.props，将nextProps的state为当前组件的state，从而重新渲染组件，可以在此方法中使用this.setState改变state。

```js
componentWillReceiveProps (nextProps) {
    nextProps.openNotice !== this.props.openNotice&&this.setState({
        openNotice:nextProps.openNotice
    }，() => {
      console.log(this.state.openNotice:nextProps)
      //将state更新为nextProps,在setState的第二个参数（回调）可以打         印出新的state
    })
}
```

> 请注意，如果父组件导致组件重新渲染，即使 props 没有更改，也会调用此方法。如果只想处理更改，请确保进行当前值与变更值的比较。
>
> React 不会针对初始 props 调用 UNSAFE_componentWillReceiveProps()。组件只会在组件的 props 更新时调用此方法。调用 this.setState() 通常不会触发该生命周期。

### 3.2 shouldComponentUpdate

在渲染之前被调用，默认返回为true。

​    返回值是判断组件的输出是否受当前state或props更改的影响，默认每次state发生变化都重新渲染，首次渲染或使用forceUpdate(使用`this.forceUpdate()`)时不被调用。

> 他主要用于性能优化，会对 props 和 state 进行浅层比较，并减少了跳过必要更新的可能性。不建议深层比较，会影响性能。如果返回false，则不会调用componentWillUpdate、render和componentDidUpdate

- 唯一用于控制组件重新渲染的生命周期，由于在react中，setState以后，state发生变化，组件会进入重新渲染的流程，在这里return false可以阻止组件的更新，但是不建议，建议使用 PureComponent 
- 因为react父组件的重新渲染会导致其所有子组件的重新渲染，这个时候其实我们是不需要所有子组件都跟着重新渲染的，因此需要在子组件的该生命周期中做判断

### 3.3 componentWillUpdate (即将废弃)

**当组件接收到新的props和state会在渲染前调用，初始渲染不会调用该方法。**

​    shouldComponentUpdate返回true以后，组件进入重新渲染的流程，进入componentWillUpdate，不能在这使用setState，在函数返回之前不能执行任何其他更新组件的操作

> 此方法可以替换为 `componentDidUpdate()`。如果你在此方法中读取 DOM 信息（例如，为了保存滚动位置），则可以将此逻辑移至 `getSnapshotBeforeUpdate()` 中。

### 3.4 getSnapshotBeforeUpdate（新钩子）

**在最近一次的渲染输出之前被提交之前调用，也就是即将挂载时调用，替换componetnWillUpdate。**

相当于淘宝购物的快照，会保留下单前的商品内容，在 React 中就相当于是 即将更新前的状态

它可以使组件在 DOM 真正更新之前捕获一些信息（例如滚动位置），此生命周期返回的任何值都会作为参数传递给 `componentDidUpdate()`。如不需要传递任何值，那么请返回 null

> 和componentWillUpdate的区别
>
> - 在 React 开启异步渲染模式后，在 render 阶段读取到的 DOM 元素状态并不总是和 commit 阶段相同，这就导致在componentDidUpdate 中使用 componentWillUpdate 中读取到的 DOM 元素状态是不安全的，因为这时的值很有可能已经失效了。
> - getSnapshotBeforeUpdate 会在最终的 render 之前被调用，也就是说getSnapshotBeforeUpdate 中读取到的 DOM 元素状态是可以保证与 componentDidUpdate 中一致的。

### 3.5 componentDidUpdate

**组件在更新完毕后会立即被调用，首次渲染不会调用**

可以在该方法调用setState，但是要包含在条件语句中，否则一直更新会造成死循环。

当组件更新后，可以在此处对 DOM 进行操作。如果对更新前后的props进行了比较，可以进行网络请求。（当 props 未发生变化时，则不会执行网络请求）。

```javascript
componentDidUpdate(prevProps,prevState,snapshotValue) {
  // 典型用法（不要忘记比较 props）：
  if (this.props.userID !== prevProps.userID) {
    this.fetchData(this.props.userID);
  }
}
```

> 如果组件实现了 `getSnapshotBeforeUpdate()` 生命周期（不常用），则它的返回值将作为 `componentDidUpdate()` 的第三个参数 “snapshotValue” 参数传递。否则此参数将为 undefined。如果返回false就不会调用这个函数。

### 3.6 getSnapshotBeforeUpdate使用场景

在一个区域内，定时的输出以行话，如果内容大小超过了区域大小，就出现滚动条，但是内容不进行移动 

![BeforeGender](https://i0.hdslb.com/bfs/album/0ce6f820adb5b75e44b1df2332caa58bb8eaa257.gif)

如上面的动图：区域内部的内容展现没有变化，但是可以看见滚动条在变化，也就是说上面依旧有内容在输出，只不过不在这个区域内部展现。

1.首先我们先实现定时输出内容

我们可以使用state状态，改变新闻后面的值，但是为了同时显示这些内容，我们应该为state的属性定义一个数组。并在创建组件之后开启一个定时器，不断的进行更新state。更新渲染组件

```js
 class New extends React.Component{

        state = {num:[]};

        //在组件创建之后,开启一个定时任务
        componentDidMount(){
            setInterval(()=>{
                let {num} = this.state;
                const news = (num.length+1);
                this.setState({num:[news,...num]});
            },2000);
        }

        render(){
            return (
                <div ref = "list" className = "list">{
                    this.state.num.map((n,index)=>{
                    return <div className="news" key={index} >新闻{n}</div>
                    })
                }</div>
            )
        }
  }
  ReactDOM.render(<New />,document.getElementById("div"));
```

2.接下来就是控制滚动条了

我们在组件渲染到DOM之前获取组件的高度，然后用组件渲染之后的高度减去之前的高度就是一条新的内容的高度，这样在不断的累加到滚动条位置上。

````js
getSnapshotBeforeUpdate(){
	return this.refs.list.scrollHeight;
}

componentDidUpdate(preProps,preState,height){
	this.refs.list.scrollTop += (this.refs.list.scrollHeight - height);
}
````

这样就实现了这个功能。

## 4.卸载组件

**当组件从 DOM中移除时会调用如下方法**

### 4.1 componentWillUnmount

**在组件卸载和销毁之前调用**

> 使用这样的方式去卸载`ReactDOM.unmountComponentAtNode(document.getElementById('test'))`

​    在这执行必要的清理操作，例如，清除timer（setTimeout,setInterval），取消网络请求，或者取消在componentDidMount的订阅，移除所有监听

有时候我们会碰到这个warning:

```js
Can only update a mounted or mounting component. This usually means you called setState() on an unmounted component. This is a   no-op. Please check the code for the undefined component.
```

原因：因为你在组件中的ajax请求返回setState,而你组件销毁的时候，请求还未完成，因此会报warning

解决方法：

```javascript
componentDidMount() {
    this.isMount === true
    axios.post().then((res) => {
    this.isMount && this.setState({   // 增加条件ismount为true时
      aaa:res
    })
})
}
componentWillUnmount() {
    this.isMount === false
}
```

`componentWillUnmount()` 中不应调用 `setState()`，因为该组件将永远不会重新渲染。组件实例卸载后，将永远不会再挂载它。

# 05 【条件渲染】

在 React 中，你可以创建不同的组件来封装各种你需要的行为。然后，依据应用的不同状态，你可以只渲染对应状态下的部分内容。

## 基础配置

```js
<style>
    .other {
        color: #ff0000;
    }
</style>
<body>
<div id="app"></div>

<script type="text/babel">
class Demo extends React.Component {
    state = {
        type: 1,
        isLogin:false
    }

    render() {
        const {type} = this.state
        return (
            <div>
                {type}
            </div>
        );
    }
}

ReactDOM.render(<Demo/>, document.getElementById('app'))
</script>
```

## 1.条件判断语句

React 中的条件渲染和 JavaScript 中的一样，使用 JavaScript 运算符 [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) 或者[条件运算符](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)去创建元素来表现当前的状态，然后让 React 根据它们来更新 UI。

- 适合逻辑较多的情况

```js
//1. 第一种方法，声明函数返回dom
showMsg = () => {
    let type = this.state.type
    if (type === 1) {
        return (<h2>第一种写法：type值等于1</h2>)
    } else {
        return (<h2 className="other">第一种写法：type值不等于1</h2>)
    }
}
render() {
    return (
        <div>
            {this.showMsg()}
        </div>
    );
}
```

页面展示：

![image-20221024141313955](https://i0.hdslb.com/bfs/album/e4ccc5074a4a2c0ff03053844115f4b9a7902d23.png)

```js
render() {
  let welcome = ''
  let btnText = ''
  if (this.state.isLogin) {
    welcome = '欢迎回来'
    btnText = '退出'
  } else {
    welcome = '请先登录~'
    btnText = '登录'
  }

  return (
    <div>
        <h2>{welcome}</h2>
        <button>{btnText}</button>
    </div>
  )
}
```

![image-20221024141928366](https://i0.hdslb.com/bfs/album/11e95969df2c279a2c9748e85a17bb4be61a3e5a.png)

## 2.三目运算符

另一种内联条件渲染的方法是使用 JavaScript 中的三目运算符 [`condition ? true : false`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)。

- 适合逻辑比较简单

````js
render() {
  const { type } = this.state
  return (
    <div>
      {
        //3. 第三种方法，利用三目运算符渲染需要渲染的变量
        type === 1 ? (
          <h2>第二种写法：type值等于1</h2>
        ) : (
          <h2 className="other">第三种写法：type值不等于1</h2>
        )
      }
    </div>
  )
}
}
````

![image-20221024142209840](https://i0.hdslb.com/bfs/album/43cea5edfd354e607925f31a2e0e9efb684276ef.png)

在下面这个示例中，我们用它来条件渲染一小段文本

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

同样的，它也可以用于较为复杂的表达式中，虽然看起来不是很直观：

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn
        ? <LogoutButton onClick={this.handleLogoutClick} />
        : <LoginButton onClick={this.handleLoginClick} />
      }
    </div>
  );
}
```

就像在 JavaScript 中一样，你可以根据团队的习惯来选择可读性更高的代码风格。需要注意的是，如果条件变得过于复杂，那你应该考虑如何[提取组件](https://zh-hans.reactjs.org/docs/components-and-props.html#extracting-components)。

## 3.与运算符&&

通过花括号包裹代码，你可以[在 JSX 中嵌入表达式](https://zh-hans.reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)。这也包括 JavaScript 中的逻辑与 (&&) 运算符。它可以很方便地进行元素的条件渲染：

- 适合如果条件成立，渲染某一个组件；如果条件不成立，什么内容也不渲染；

````js
render() {
  const { type } = this.state
  return (
    <div>
      {type === 1 && <h2>第三种写法：type值等于1</h2>}
      {type !== 1 && <h2 className="other">第三种写法：type值不等于1</h2>}
    </div>
  )
}
}
````

![image-20221024142459699](https://i0.hdslb.com/bfs/album/f474a79f14306b2e532b9b0090eca7d4113d7fb3.png)

之所以能这样做，是因为在 JavaScript 中，`true && expression` 总是会返回 `expression`, 而 `false && expression` 总是会返回 `false`。

因此，如果条件是 `true`，`&&` 右侧的元素就会被渲染，如果是 `false`，React 会忽略并跳过它。

请注意，[falsy 表达式](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) 会使 `&&` 后面的元素被跳过，但会返回 falsy 表达式的值。在下面示例中，render 方法的返回值是 `<div>0</div>`。

```js
render() {
  const count = 0;
  return (
    <div>
      {count && <h1>Messages: {count}</h1>}
    </div>
  );
}
```

## 4.元素变量

你可以使用变量来储存元素。 它可以帮助你有条件地渲染组件的一部分，而其他的渲染部分并不会因此而改变。

```js
render() {
  const { type } = this.state
  //2. 第二种方法 声明变量 给变量赋值
  let test = null
  if (type === 1) {
    test = <h2>第四种写法：type值等于1</h2>
  } else {
    test = <h2 className="other">第四种写法：type值不等于1</h2>
  }
  return <div>{test}</div>
}
}
```

![image-20221024142858454](https://i0.hdslb.com/bfs/album/4809a71dd9dacb554f6f009687701cd7aaf2bb83.png)

声明一个变量并使用 `if` 语句进行条件渲染是不错的方式，但有时你可能会想使用更为简洁的语法，那就是内联条件渲染的方法与运算和三目运算符

## 5.阻止组件渲染

在极少数情况下，你可能希望能隐藏组件，即使它已经被其他组件渲染。若要完成此操作，你可以让 `render` 方法直接返回 `null`，而不进行任何渲染。

下面的示例中，`<WarningBanner />` 会根据 prop 中 `warn` 的值来进行条件渲染。如果 `warn` 的值是 `false`，那么组件则不会渲染:

````js
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(state => ({
      showWarning: !state.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

const root = ReactDOM.createRoot(document.getElementById('root')); 
root.render(<Page />);
````

在组件的 `render` 方法中返回 `null` 并不会影响组件的生命周期。例如，上面这个示例中，`componentDidUpdate` 依然会被调用。

# 06 【列表 & Key】

首先，让我们看下在 Javascript 中如何转化列表。

如下代码，我们使用 [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 函数让数组中的每一项变双倍，然后我们得到了一个新的列表 `doubled` 并打印出来：

```js
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);console.log(doubled);
```

代码打印出 `[2, 4, 6, 8, 10]`。

在 React 中，把数组转化为[元素](https://zh-hans.reactjs.org/docs/rendering-elements.html)列表的过程是相似的。

## 1.列表

### 1.1 渲染多个组件

你可以通过使用 `{}` 在 JSX 内构建一个[元素集合](https://zh-hans.reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)。

下面，我们使用 Javascript 中的 [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 方法来遍历 `numbers` 数组。将数组中的每个元素变成 `<li>` 标签，最后我们将得到的数组赋值给 `listItems`：

```js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>  <li>{number}</li>);
```

然后，我们可以将整个 `listItems` 插入到 `<ul>` 元素中：

```js
<ul>{listItems}</ul>
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

````js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((numbers) =>
  <li>{numbers}</li>
);

const root = ReactDOM.createRoot(document.getElementById('root')); 
root.render(<ul>{listItems}</ul>);
````

这段代码生成了一个 1 到 5 的项目符号列表。

![image-20221024211657792](https://i0.hdslb.com/bfs/album/79b008abbd0b656cb0b29631f175dfcb936cacc0.png)

### 1.2 基础列表组件

通常你需要在一个[组件](https://zh-hans.reactjs.org/docs/components-and-props.html)中渲染列表。

我们可以把前面的例子重构成一个组件，这个组件接收 `numbers` 数组作为参数并输出一个元素列表。

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<NumberList numbers={numbers} />);
```

当我们运行这段代码，将会看到一个警告 `a key should be provided for list items`，意思是当你创建一个元素时，必须包括一个特殊的 `key` 属性。我们将在下一节讨论这是为什么。

让我们来给每个列表元素分配一个 `key` 属性来解决上面的那个警告：

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];

const root = ReactDOM.createRoot(document.getElementById('root')); 
root.render( <NumberList numbers={numbers} />);
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

![image-20221024211835947](https://i0.hdslb.com/bfs/album/7d1bee8ee7ee79bb7991b83694bd6161da4a9c3d.png)

## 2.key

### 2.1 基本使用

key 帮助 React 识别哪些元素改变了，比如被添加或删除。因此你应当给数组中的每一个元素赋予一个确定的标识。

```js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

一个元素的 key 最好是这个元素在列表中拥有的一个独一无二的字符串。通常，我们使用数据中的 id 来作为元素的 key：

```js
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

当元素没有确定 id 的时候，万不得已你可以使用元素索引 index 作为 key：

```js
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

如果列表项目的顺序可能会变化，我们不建议使用索引来用作 key 值，因为这样做会导致性能变差，还可能引起组件状态的问题。可以看看 Robin Pokorny 的[深度解析使用索引作为 key 的负面影响](https://robinpokorny.com/blog/index-as-a-key-is-an-anti-pattern/)这一篇文章。如果你选择不指定显式的 key 值，那么 React 将默认使用索引用作为列表项目的 key 值。

要是你有兴趣了解更多的话，这里有一篇文章[深入解析为什么 key 是必须的](https://zh-hans.reactjs.org/docs/reconciliation.html#recursing-on-children)可以参考。

### 2.2 用 key 提取组件

元素的 key 只有放在就近的数组上下文中才有意义。

比方说，如果你[提取](https://zh-hans.reactjs.org/docs/components-and-props.html#extracting-components)出一个 `ListItem` 组件，你应该把 key 保留在数组中的这个 `<ListItem />` 元素上，而不是放在 `ListItem` 组件中的 `<li>` 元素上。

**例子：不正确的使用 key 的方式**

```js
function ListItem(props) {
  const value = props.value;
  return (
    // 错误！你不需要在这里指定 key：
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 错误！元素的 key 应该在这里指定：
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

**例子：正确的使用 key 的方式**

```js
function ListItem(props) {
  // 正确！这里不需要指定 key：
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 正确！key 应该在数组的上下文中被指定
    <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/ZXeOGM?editors=0010)

一个好的经验法则是：在 `map()` 方法中的元素需要设置 key 属性。

### 2.3 key 值在兄弟节点之间必须唯一

数组元素中使用的 key 在其兄弟节点之间应该是独一无二的。然而，它们不需要是全局唯一的。当我们生成两个不同的数组时，我们可以使用相同的 key 值：

```js
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Blog posts={posts} />);
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

key 会传递信息给 React ，但不会传递给你的组件。如果你的组件中需要使用 `key` 属性的值，请用其他属性名显式传递这个值：

```js
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

上面例子中，`Post` 组件可以读出 `props.id`，但是不能读出 `props.key`。

### 2.4 在 JSX 中嵌入 map()

在上面的例子中，我们声明了一个单独的 `listItems` 变量并将其包含在 JSX 中：

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

JSX 允许在大括号中[嵌入任何表达式](https://zh-hans.reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)，所以我们可以内联 `map()` 返回的结果：

```js
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)

这么做有时可以使你的代码更清晰，但有时这种风格也会被滥用。就像在 JavaScript 中一样，何时需要为了可读性提取出一个变量，这完全取决于你。但请记住，如果一个 `map()` 嵌套了太多层级，那可能就是你[提取组件](https://zh-hans.reactjs.org/docs/components-and-props.html#extracting-components)的一个好时机。

## 3.diff算法

### 3.1 什么是虚拟 DOM ？

在谈 diff 算法之前，我们需要先了解虚拟 DOM 。它是一种编程概念，在这个概念里，以一种虚拟的表现形式被保存在内存中。在 React 中，render 执行的结果得到的并不是真正的 DOM 节点，而是 JavaScript 对象

> 虚拟 DOM 只保留了真实 DOM 节点的一些**基本属性，和节点之间的层次关系**，它相当于建立在 JavaScript 和 DOM 之间的一层“缓存”

```js
<div class="hello">
    <span>hello world!</span>
</div>
```

上面的这段代码会转化可以转化为虚拟 DOM 结构

```js
{
    tag: "div",
    props: {
        class: "hello"
    },
    children: [{
        tag: "span",
        props: {},
        children: ["hello world!"]
    }]
}
```

其中对于一个节点必备的三个属性 `tag，props，children`

- tag 指定元素的**标签**类型，如“`li`，`div`”
- props 指定元素身上的属性，如 `class` ，`style`，自定义属性
- children 指定元素是否有**子节点**，参数以**数组**形式传入

而我们在 render 中编写的 JSX 代码就是一种虚拟 DOM 结构。

### 3.2 diff 算法

每个组件中的每个标签都会有一个key,不过有的必须显示的指定，有的可以隐藏。

如果生成的render出来后就不会改变里面的内容，那么你不需要指定key（不指定key时，React也会生成一个默认的标识）,或者将index作为key，只要key不重复即可。

但是如果你的标签是动态的，是有可能刷新的，就必须显示的指定key。使用map进行遍历的时候就必须指定Key:

```js
this.state.num.map((n,index)=>{
	return <div className="news" key={index} >新闻{n}</div>
})
```

这个地方虽然显示的指定了key，但是**官网并不推荐使用Index作为Key去使用**；

这样会很有可能会有效率上的问题

举个例子：

在一个组件中，我们先创建了两个对象，通过循环的方式放入< li>标签中，此时key使用的是index。

```js
person:[
    {id:1,name:"张三",age:18},
    {id:2,name:"李四",age:19}
]

this.state.person.map((preson,index)=>{
  return  <li key = {index}>{preson.name}</li>
})
```

如下图展现在页面中：

![image-20221024225054061](https://i0.hdslb.com/bfs/album/ad5611b1f134b0a842dd2365db974714c98f6a9c.png)

此时，我们想在点击按钮之后动态的添加一个对象，并且放入到li标签中，在重新渲染到页面中。

我们通过修改State来控制对象的添加。

```js
<button onClick={this.addObject}>点击增加对象</button>
addObject = () =>{
    let {person} = this.state;
    const p = {id:(person.length+1),name:"王五",age:20};
    this.setState({person:[p,...person]});
}
```

如下动图所示：

![addObject](https://i0.hdslb.com/bfs/album/ff6d81e4297b4798020721e60df525a2036f796e.gif)

这样看，虽然完成了功能。但是其实存在效率上的问题， 我们先来看一下两个前后组件状态的变化：

![image-20221024225208300](https://i0.hdslb.com/bfs/album/21767b62ed6cd7f93b146dccdbe4b7007ab00c14.png)

我们发现，组件第一个变成了王五，张三和李四都移下去了。因为我们使用Index作为Key，这三个标签的key也就发生了改变【张三原本的key是0，现在变成了1，李四的key原本是1，现在变成了2，王五变成了0】

在组件更新状态重新渲染的时候，就出现了问题：

因为react是通过key来比较组件标签是否一致的，拿这个案例来说：

首先，状态更新导致组件标签更新，react根据Key，判断旧的虚拟DOM和新的虚拟DOM是否一致

key = 0 的时候 旧的虚拟DOM 内容是张三 新的虚拟DOM为王五 ，react认为内容改变，从而重新创建新的真实DOM.

key = 1 的时候 旧的虚拟DOM 内容是李四，新的虚拟DOM为张三，react认为内容改变，从而重新创建新的真实DOM

key = 2 的时候 旧的虚拟DOM没有，创建新的真实DOM

这样原本有两个虚拟DOM可以复用，但都没有进行复用，完完全全的都是新创建的；这就导致效率极大的降低。

其实这是因为我们将新创建的对象放在了首位，如果放在最后其实是没有问题的，但是因为官方并不推荐使用Index作为key值，我们推荐使用id作为key值。从而完全避免这样的情况。

### 3.3 用index作为key可能会引发的问题

key不需要全局唯一，只需在当前列表中唯一即可。元素的key最好是固定的，这里直接举个反例，有些场景我们会使用元素的索引为key像这种：

```jsx
const students = ['孙悟空', '猪八戒', '沙和尚'];
const ele = <ul>{students.map((item, index) => <li key={index}>{item}</li>)}</ul>
```

上例中，我使用了元素的索引（index）作为key来使用，但这有什么用吗？没用！因为index是根据元素位置的改变而改变的，当我们在前边插入一个新元素时，所有元素的顺序都会一起改变，那么它和React中按顺序比较有什么区别吗？没有区别！而且还麻烦了，唯一的作用就是去除了警告。所以我们开发的时候偶尔也会使用索引作为key，但前提是元素的顺序不会发生变化，除此之外不要用索引做key。

1. 若对数据进行:逆序添加、逆序删除等破坏
   顺序操作:会产生没有必要的真实DOM更新 界面效果没问题,但效率低。

2. 如果结构中还包含输入类的DOM:会产生错误DOM更新 界面有问题。

3. 注意! 如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key是没有问题的。

**开发如何选择key?**

最好使用每一条数据的唯一标识作为key 比如id，手机号，身份证号

如果确定只是简单的展示数据，用Index也是可以的

**而这个判断key的比较规则就是Diff算法**

Diff算法其实就是react生成的新虚拟DOM和以前的旧虚拟DOM的比较规则：

- 如果旧的虚拟DOM中找到了与新虚拟DOM相同的key:
  - 如果内容没有变化，就直接只用之前旧的真实DOM
  - 如果内容发生了变化，就生成新的真实DOM
- 如果旧的虚拟DOM中没有找到了与新虚拟DOM相同的key:
  - 根据数据创建新的真实的DOM,随后渲染到页面上

### 3.4 李立超老师对于虚拟DOM的解释

当我们通过 React 操作DOM时，比如通过 `React.createElement()` 创建元素时。我们所创建的元素并不是真正的DOM对象而是React元素。这一点可以通过在控制台中打印对象来查看。React元素是React应用的最小组成部分，通过JSX也就是`React.createElement()`所创建的元素都属于React元素。与浏览器的 DOM 元素不同，React 元素就是一个普通的JS对象，且创建的开销极小。

React元素不是DOM对象，那为什么可以被添加到页面中去呢？实际上每个React元素都会有一个对应的DOM元素，对React元素的所有操作，最终都会转换为对DOM元素操作，也就是所谓的虚拟DOM。要理解虚拟DOM，我们需要先了解它的作用。虚拟DOM就好像我们和真实DOM之间的一个桥梁。有了虚拟DOM，使得我们无需去操作真实的DOM元素，只需要对React元素进行操作，所有操作最终都会映射到真实的DOM元素上。

这不是有点多余吗？直接操作DOM不好吗？为什么要多此一举呢？原因其实很多，这里简单举几个出来。

首先，虚拟DOM简化了DOM操作。凡是用过DOM的都知道Web API到底有多复杂，各种方法，各种属性，数不胜数。查询的、修改的、删除的、添加的等等等等。然而在虚拟DOM将所有的操作都简化为了一种，那就是创建！React元素是不可变对象，一旦创建就不可更改。要修改元素的唯一方式就是创建一个新的元素去替换旧的元素，看起来虽然简单粗暴，实则却是简化了DOM的操作。

其次，解决DOM的兼容性问题。DOM的兼容性是一个历史悠久的问题，如果使用原生DOM，总有一些API会遇到兼容性的问题。使用虚拟DOM就完美的避开了这些问题，所有的操作都是在虚拟DOM上进行的，而虚拟DOM是没有兼容问题的，至于原生DOM是否兼容就不需要我们操心了，全都交给React吧！

最后，我们手动操作DOM时，由于无法完全掌握全局DOM情况，经常会出现不必要的DOM操作，比如，本来只需要修改一个子节点，但却不小心修改了父节点，导致所有的子节点都被修改。效果呈现上可能没有什么问题，但是性能上确实千差万别，修改一个节点和修改多个节点对于系统的消耗可是完全不同的。然而在虚拟DOM中，引入了diff算法，React元素在更新时会通过diff算法和之前的元素进行比较，然后只会对DOM做必要的更新来呈现结果。简单来说，就是拿新建的元素和旧的元素进行比较，只对发生变化的部分对DOM进行更新，减少DOM的操作，从而提升了性能。


# 07 【收集表单数据】

在 React 里，HTML 表单元素的工作方式和其他的 DOM 元素有些不同，这是因为表单元素通常会保持一些内部的 state。例如这个纯 HTML 表单只接受一个名称：

```html
<form>
  <label>
    名字:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="提交" />
</form>
```

此表单具有默认的 HTML 表单行为，即在用户提交表单后浏览到新页面。如果你在 React 中执行相同的代码，它依然有效。但大多数情况下，使用 JavaScript 函数可以很方便的处理表单的提交， 同时还可以访问用户填写的表单数据。实现这种效果的标准方式是使用“受控组件”。

## 状态属性

表单元素有这么几种属于状态的属性：

- `value`，对应 `<input>` 和 `<textarea>` 所有
- `checked`，对应类型为 `checkbox` 和 `radio` 的 `<input>` 所有
- `selected`，对应 `<option>` 所有

*在 HTML 中 `<textarea>` 的值可以由子节点（文本）赋值，但是在 React 中，要用 `value` 来设置。*

表单元素包含以上任意一种状态属性都支持 `onChange` 事件监听状态值的更改。

针对这些状态属性不同的处理策略，表单元素在 React 里面有两种表现形式。

## 1.受控组件

在 HTML 中，表单元素（如`<input>`、 `<textarea>` 和 `<select>`）通常自己维护 state，并根据用户输入进行更新。而在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 [`setState()`](https://zh-hans.reactjs.org/docs/react-component.html#setstate)来更新。

我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作，被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。

例如，如果我们想让前一个示例在提交时打印出名称，我们可以将表单写为受控组件：

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('提交的名字: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          名字:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/VmmPgp?editors=0010)

由于在表单元素上设置了 `value` 属性，因此显示的值将始终为 `this.state.value`，这使得 React 的 state 成为唯一数据源。由于 `handlechange` 在每次按键时都会执行并更新 React 的 state，因此显示的值将随着用户输入而更新。

对于受控组件来说，输入的值始终由 React 的 state 驱动。你也可以将 value 传递给其他 UI 元素，或者通过其他事件处理函数重置，但这意味着你需要编写更多的代码。

**为什么要有受控组件？**

引入受控组件不是说它有什么好处，而是因为 React 的 UI 渲染机制，对于表单元素不得不引入这一特殊的处理方式。

在浏览器 DOM 里面是有区分 *attribute* 和 *property* 的。*attribute* 是在 HTML 里指定的属性，而每个 HTML 元素在 JS 对应是一个 DOM 节点对象，这个对象拥有的属性就是 *property*（可以在 console 里展开一个 DOM 节点对象看一下，HTML *attributes* 只是对应其中的一部分属性），*attribute* 对应的 *property* 会从 *attribute* 拿到初始值，有些会有相同的名称，但是有些名称会不一样，比如 *attribute* `class` 对应的 *property* 就是 `className`。（详细解释：[.prop](http://api.jquery.com/prop/)，[.prop() vs .attr()](http://stackoverflow.com/questions/5874652/prop-vs-attr)）

回到 React 里的 `<input>` 输入框，当用户输入内容的时候，输入框的 `value` *property* 会改变，但是 `value` *attribute* 依然会是 HTML 上指定的值（*attribute* 要用 `setAttribute` 去更改）。

React 组件必须呈现这个组件的状态视图，这个视图 HTML 是由 `render` 生成，所以对于

```javascript
render: function() {
    return <input type="text" value="hello"/>;
}
```

在任意时刻，这个视图总是返回一个显示 `hello` 的输入框。

## 2.非受控组件

在大多数情况下，我们推荐使用 [受控组件](https://zh-hans.reactjs.org/docs/forms.html#controlled-components) 来处理表单数据。在一个受控组件中，表单数据是由 React 组件来管理的。另一种替代方案是使用非受控组件，这时表单数据将交由 DOM 节点来处理。

### 2.1 基本概念

要编写一个非受控组件，而不是为每个状态更新都编写数据处理函数，你可以 [使用 ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 来从 DOM 节点中获取表单数据。

例如，下面的代码使用非受控组件接受一个表单的值：

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/WooRWa?editors=0010)

因为非受控组件将真实数据储存在 DOM 节点中，所以在使用非受控组件时，有时候反而更容易同时集成 React 和非 React 代码。如果你不介意代码美观性，并且希望快速编写代码，使用非受控组件往往可以减少你的代码量。否则，你应该使用受控组件。

如果你还是不清楚在某个特殊场景中应该使用哪种组件，那么 [这篇关于受控和非受控输入组件的文章](https://goshakkk.name/controlled-vs-uncontrolled-inputs-react/) 会很有帮助。

### 2.2 默认值

在 React 渲染生命周期时，表单元素上的 `value` 将会覆盖 DOM 节点中的值。在非受控组件中，你经常希望 React 能赋予组件一个初始值，但是不去控制后续的更新。 在这种情况下, 你可以指定一个 `defaultValue` 属性，而不是 `value`。在一个组件已经挂载之后去更新 `defaultValue` 属性的值，不会造成 DOM 上值的任何更新。

```html
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={this.input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

同样，`<input type="checkbox">` 和 `<input type="radio">` 支持 `defaultChecked`，`<select>` 和 `<textarea>` 支持 `defaultValue`。

## 3.标签变化

### 3.1 textarea 标签

在 HTML 中, `<textarea>` 元素通过其子元素定义其文本:

```html
<textarea>
  你好， 这是在 text area 里的文本
</textarea>
```

而在 React 中，`<textarea>` 使用 `value` 属性代替。这样，可以使得使用 `<textarea>` 的表单和使用单行 input 的表单非常类似：

```js
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: '请撰写一篇关于你喜欢的 DOM 元素的文章.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('提交的文章: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          文章:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```

请注意，`this.state.value` 初始化于构造函数中，因此文本区域默认有初值。

### 3.2 select 标签

在 HTML 中，`<select>` 创建下拉列表标签。例如，如下 HTML 创建了水果相关的下拉列表：

```js
<select>
  <option value="grapefruit">葡萄柚</option>
  <option value="lime">酸橙</option>
  <option selected value="coconut">椰子</option>
  <option value="mango">芒果</option>
</select>
```

请注意，由于 `selected` 属性的缘故，椰子选项默认被选中。React 并不会使用 `selected` 属性，而是在根 `select` 标签上使用 `value` 属性。这在受控组件中更便捷，因为您只需要在根标签中更新它。例如：

```js
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('你喜欢的风味是: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          选择你喜欢的风味:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">葡萄柚</option>
            <option value="lime">酸橙</option>
            <option value="coconut">椰子</option>
            <option value="mango">芒果</option>
          </select>
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/JbbEzX?editors=0010)

总的来说，这使得 `<input type="text">`, `<textarea>` 和 `<select>` 之类的标签都非常相似—它们都接受一个 `value` 属性，你可以使用它来实现受控组件。

> 注意
>
> 你可以将数组传递到 `value` 属性中，以支持在 `select` 标签中选择多个选项：
>
> ```js
> <select multiple={true} value={['B', 'C']}>
> ```

### 3.3 文件 input 标签

在 HTML 中，`<input type="file">` 可以让用户选择一个或多个文件上传到服务器，或者通过使用 [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications) 进行操作。

```html
<input type="file" />
```

在 React 中，`<input type="file" />` 始终是一个非受控组件，因为它的值只能由用户设置，而不能通过代码控制。

您应该使用 File API 与文件进行交互。下面的例子显示了如何创建一个 [DOM 节点的 ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 从而在提交表单时获取文件的信息。

```js
class FileInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.fileInput = React.createRef();
  }
  handleSubmit(event) {
    event.preventDefault();
    alert(
      `Selected file - ${this.fileInput.current.files[0].name}`
    );
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Upload file:
          <input type="file" ref={this.fileInput} />
        </label>
        <br />
        <button type="submit">Submit</button>
      </form>
    );
  }
}

const root = ReactDOM.createRoot(
  document.getElementById('root')
);
root.render(<FileInput />);
```

[**在 CodePen 上尝试**](https://zh-hans.reactjs.org/redirect-to-codepen/uncontrolled-components/input-type-file)


# 08 【状态提升】

## 1.介绍

所谓 **状态提升** 就是将各个子组件的 公共state 提升到它们的父组件进行统一存储、处理（这就是所谓的”单一数据源“），负责`setState`的函数传到下边的子级组件，然后再将父组件处理后的数据或函数props到各子组件中。

那么如果子组件 要 修改父组件的state该怎么办呢？我们的做法就是 将父组件中负责setState的函数，以props的形式传给子组件，然后子组件在需要改变state时调用即可。

**实现方式**

实现方式是 利用最近的共同的父级组件中，用props的方式传过去到两个子组件，props中传的是一个setState的方法，通过子组件触发props传过去的方法，进而调用父级组件的setState的方法，改变了父级组件的state，调用父级组件的render方法，进而同时改变了两个子级组件的render。

这是 两个有关连的同级组件的传值，因为react的单项数据流，所以不在两个组件中进行传值，而是提升到 最近的共同的父级组件中，改变父级的state,进而影响了两个子级组件的render。

> 官网介绍
>
> 通常，多个组件需要反映相同的变化数据，这时我们建议将共享状态提升到最近的共同父组件中去。

## 2.案例

先写一个温度输入组件：

```js
class TemperatureInput extends React.Component {
    state = {
        temperature: ''
    };
    handleChange = (e) => {
        this.setState({
            temperature : e.target.value
        })
    };
    render() {
        return (
            <fieldset>
                <legend>输入{scaleNames[this.props.scale]}:</legend>
                <input type="number" value={this.state.temperature} onChange={this.handleChange}
            </fieldset>
        )
    }
}
```

这个组件就是一个普通的**受控组件**，有`state`和`props`以及处理函数。

我们在写另一个组件：

```js
class Calculator extends React.Component {
    render () {
        return (
            <div>
                <TemperatureInput scale='c'/>
                <TemperatureInput scale='f'/>
            </div>
        )
    }
}
```

这个组件现在没有什么存在的价值，我们仅仅是给两个温度输入组件提供一个父组件，以便我们进行后续的**状态提升**。

现在我们看看网页的样子：

![image-20221025123600431](https://i0.hdslb.com/bfs/album/a4228155682c5b7715204c99d704b8f4b9daf6a6.png)

我们可以输入摄氏度和华氏度，但是我们现在想要让这两个温度保持一致，就是我们如果输入摄氏度，那么下面的华氏度可以自动算出来，如果我们输入华氏度，那么摄氏度就可以自动算出来。

那么我们按照现在这种结构的话，是非常难以实现的，因为我们知道这两个组件之间没有任何关系，它们之间是不知道对方的存在，所以我们需要把它们的状态进行提升，提升到它们的父组件当中。

那我们看看如何做修改，首先把子组件（温度输入组件）的状态（state）全部删除，看看是什么样子:

```js
    class TemperatureInput extends React.Component {

        handleChange = (e) => {

        };

        render() {
            return (
                <fieldset>
                    <legend>输入{scaleNames[this.props.scale]}:</legend>
                    <input type="number" value={this.props.temperature} onChange={this.handleChange}/>
                </fieldset>
            )
        }
    }
```

可以看到所有与`state`有关的东西全部删掉了，然后`input`的`value`也变成了`props`，通过父组件传入。那么现在这个温度输入组件其实就是一个受控组件了，仔细回忆一下我们之前讲的受控组件，看看是不是这样意思？

我们通常会在受控组件发生改变的时候传入一个`onChange`函数来改变受控组件的状态，那么我们这里也是一样，我们通过给 温度输入组件 传入某个函数来让 温度输入组件 中的`input`发生变化的时候调用，当然这个函数我们可以随意命名，假如我们这里叫做`onTemperatureChange`函数，那么我们继续修改子组件：

```jsx
    class TemperatureInput extends React.Component {

        handleChange = (e) => {
            this.props.onTemperatureChange(e.target.value);
        };

        render() {
            return (
                <fieldset>
                    <legend>输入{scaleNames[this.props.scale]}:</legend>
                    <input type="number" value={this.props.temperature} onChange={this.handleChange}/>
                </fieldset>
            )
        }
    }
```

好了，我们的子组件差不多就是这样了，当然我们可以省略那个`handleChange`函数，因为可以看到这个函数就是调用了一下那个`props`里的函数，所以我们完全把`input`的`onChange`这么写 `<input type="number" value={this.props.temperature} onChange={this.props.onTemperatureChange}/>`这么写的话注意`onTemperatrueChange`函数的参数是那个事件，而不是我这里写的`e.target.value`。

再看看我们的父组件如何修改，我们首先补上`state`，以及子组件对应的`onChange`处理方法，以及子组件的值。写好之后大概是这个样子：

```jsx
class Calculator extends React.Component {
    state = {
        celsius: '',
        fahrenheit: ''
    };

    onCelsiusChange = (value) => {
        this.setState({
            celsius: value,
            fahrenheit: tryConvert(value, toFahrenheit)
        });
    };

    onFahrenheitChange = (value) => {
        this.setState({
            celsius: tryConvert(value, toCelsius),
            fahrenheit: value
        });
    };

    render() {
        return (
            <div>
                <TemperatureInput scale='c' temperature={this.state.celsius}
                                  onTemperatureChange={this.onCelsiusChange}/>
                <TemperatureInput scale='f' temperature={this.state.fahrenheit}
                                  onTemperatureChange={this.onFahrenheitChange}/>
            </div>
        )
    }
}
```

这里我们省略的摄氏度与华氏度的转换函数，比较简单，大家自行搜索方法。


# 09 【组合组件】

## 1.包含关系

有些组件无法提前知晓它们子组件的具体内容。在 `Sidebar`（侧边栏）和 `Dialog`（对话框）等展现通用容器（box）的组件中特别容易遇到这种情况。

我们建议这些组件使用一个特殊的 `children` prop 来将他们的子组件传递到渲染结果中：

> 组件标签里面包含的子元素会通过 `props.children` 传递进来。

```js
function One(props) {
 return (
    <div>{props.children}</div>
    //特殊的children props
  );
}

function Two(props) {
  return (
  //这使别的组件可以通过JSX嵌套，来将任意组件作为子组件来传递给他们
  <One>
      <div>Hello</div>
      <div>World</div>
  </One>
);
}
```

![image-20221025135313079](https://i0.hdslb.com/bfs/album/1c38a6b492f958e4474609b0e0205d919275819b.png)

## 2.特例关系问题

有些时候，我们会把一些组件看作是其他组件的特殊实例，比如 `WelcomeDialog` 可以说是 `Dialog` 的特殊实例。

在 React 中，我们也可以通过组合来实现这一点。“特殊”组件可以通过 props 定制并渲染“一般”组件：

```css
.FancyBorder {
  padding: 10px 10px;
  border: 10px solid;
}

.FancyBorder-blue {
  border-color: blue;
}

.Dialog-title {
  margin: 0;
  font-family: sans-serif;
}

.Dialog-message {
  font-size: larger;
}
```

```js
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/kkEaOZ?editors=0010)

组合也同样适用于以 class 形式定义的组件。

```js
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

[**在 CodePen 上尝试**](https://codepen.io/gaearon/pen/gwZbYa?editors=0010)

![image-20221025135929891](https://i0.hdslb.com/bfs/album/d343f27193258344b3ef4adf5b0f7f0f3f558a66.png)



# 10 【初始化脚手架】

## 1.什么是 React 脚手架？

在我们的现实生活中，脚手架最常用的使用场景是在工地，它是为了保证施工顺利的、方便的进行而搭建的，在工地上搭建的脚手架可以帮助工人们高校的去完成工作，同时在大楼建设完成后，拆除脚手架并不会有任何的影响。

在我们的 React 项目中，脚手架的作用与之有异曲同工之妙

React 脚手架其实是一个工具帮我们快速的生成项目的工程化结构，每个项目的结构其实大致都是相同的，所以 React 给我提前的搭建好了，这也是脚手架强大之处之一，也是用 React 创建 SPA 应用的最佳方式

## 2.为什么要用脚手架？

在前面的介绍中，我们也有了一定的认知，脚手架可以帮助我们快速的搭建一个项目结构

在我之前学习 `webpack` 的过程中，每次都需要配置 `webpack.config.js` 文件，用于配置我们项目的相关 `loader` 、`plugin`，这些操作比较复杂，但是它的重复性很高，而且在项目打包时又很有必要，那 React 脚手架就帮助我们做了这些，它不需要我们人为的去编写 `webpack` 配置文件，它将这些配置文件全部都已经提前的配置好了。

## 3.怎么用 React 脚手架？

这也是这篇文章的重点，如何去安装 React 脚手架，并且理解它其中的相关文件作用

首先介绍如何安装脚手架

### 3.1 安装 React 脚手架

首先确保安装了 `npm` 和`Node`，版本不要太古老，具体是多少不大清楚，建议还是用 `npm update` 更新一下

然后打开 cmd 命令行工具，全局安装 `create-react-app`

```
npm i create-react-app -g
```

然后可以**新建**一个文件夹用于存放项目

在当前的文件夹下执行

```
create-react-app hello-react
```

**快速搭建项目**

再在生成好的 `hello-react` 文件夹中执行

```
npm start
```

**启动项目**

接下来我们看看这些文件都有什么作用

### 3.2 使用vite创建react项目

vite官网：https://vitejs.cn

- 什么是vite？—— 新一代前端构建工具。
- 优势如下：
  - 开发环境中，无需打包操作，可快速的冷启动。
  - 轻量快速的热重载（HMR）。
  - 真正的按需编译，不再等待整个应用编译完成。

```bash
## 创建工程

# npm 6.x
$ npm init vite@latest <project-name> --template react  
## 如： npm init vite@latest react-app --template react

# npm 7+，需要加上额外的双短横线
$ npm init vite@latest <project-name> --template react

## 使用 PNPM:
pnpm create vite <project-name> --template react
# pnpm create vite react-app -- --template react

## 进入工程目录
cd <project-name>
## 安装依赖
pnpm install
## 运行
pnpm run dev
```

### 3.3 脚手架项目结构

```css
hello-react
├─ .gitignore               // 自动创建本地仓库
├─ package.json             // 相关配置文件
├─ public                   // 公共资源
│  ├─ favicon.ico           // 浏览器顶部的icon图标
│  ├─ index.html            // 应用的 index.html入口
│  ├─ logo192.png           // 在 manifest 中使用的logo图
│  ├─ logo512.png           // 同上
│  ├─ manifest.json         // 应用加壳的配置文件
│  └─ robots.txt            // 爬虫给协议文件
├─ src                      // 源码文件夹
│  ├─ App.css               // App组件的样式
│  ├─ App.js                // App组件
│  ├─ App.test.js           // 用于给APP做测试
│  ├─ index.css             // 样式
│  ├─ index.js              // 入口文件
│  ├─ logo.svg              // logo图
│  ├─ reportWebVitals.js    // 页面性能分析文件
│  └─ setupTests.js         // 组件单元测试文件
└─ yarn.lock
```

再介绍一下public目录下的 `index.html` 文件中的代码意思

````html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
````

以上是删除代码注释后的全部代码

**第5行**

指定浏览器图标的路径，这里直接采用 `%PUBLIC_URL%` 原因是 `webpack` 配置好了，它代表的意思就是 `public` 文件夹

```html
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
```

**第6行**

用于做移动端网页适配

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

**第七行**

用于配置安卓手机浏览器顶部颜色，兼容性不大好

```html
<meta name="theme-color" content="#000000" />
```

**8到11行**

用于描述网站信息

```html
<meta
	name="description"
    content="Web site created using create-react-app"
/>
```

**第12行**

苹果手机触摸版应用图标

```html
<link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
```

**第13行**

应用加壳时的配置文件

```html
<link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
```

**这里面其实最主要的就是App.js以及index.js，一个是组件，一个是将组件渲染到页面中的。**

## 4.第一个脚手架应用

1. 我们保持public中的Index.html不变

2. 修改src下面的APP.js以及index.js文件

```js
//创建外壳组件APP
import React from 'react'

class App extends React.Component{
    render(){
        return (
            <div>Hello word</div>
        )
    }
}

export default App
```

`index.js`: 【主要的作用其实就是将App这个组件渲染到页面上】

````js
//引入核心库
import React from 'react'
import ReactDOM from 'react-dom'
//引入组件
import App from './App'

ReactDOM.render(<App />,document.getElementById("root"))
````

这样在重新启动应用，就成功了。

![image-20221025142625901](https://i0.hdslb.com/bfs/album/259950b834509b96012d9e30c1b63c2d9f3b6e63.png)

我们也不建议这样直接将内容放入App组件中，尽量还是用内部组件。

我们在顶一个Hello组件：

````js
import React,{Componet} from 'react'

export default class Hello extends Componet{
    render() {
        return (
            <h1>Hello</h1>
        )
    }
}
````

在App组件中，进行使用

```js
import React,{Componet} from 'react'
import Hello form './Hello'

class App extends Component{
    render(){
        return (
            <div>
                <Hello />
            </div>
        )
    }
}
```

这样的结果和前面是一样的。

推荐使用这种目录结构去使用组件

![image-20221025142952888](https://i0.hdslb.com/bfs/album/6d23ae1393d60c7598fa0f9172061b98051a8f8e.png)

## 5.组件

### 5.1 组件基本概念

在React中网页被拆分为了一个一个组件，组件是独立可复用的代码片段。具体来说，组件可能是页面中的一个按钮，一个对话框，一个弹出层等。React中定义组件的方式有两种：基于函数的组件和基于类的组件。本节我们先看看基于函数的组件。

基于函数的组件其实就是一个会返回JSX（React元素）的普通的JS函数，你可以这样定义：

```js
import ReactDOM from "react-dom/client";

// 这就是一个组件
function App(){
    return <h1>我是一个React的组件！</h1>
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App/>);
```

函数式组件主要有两个注意点：

1. 函数名首字母大写
2. 返回值是一个JSX（React元素）

为了使得项目结构更加的清晰，更易于维护，每个组件通常会存储到一个单独的文件中，比如上例中的App组件，可以存储到App.js中，并通过export导出。

```js
App.js
function App(){
    return <h1>我是一个React的组件！</h1>
}

export default App;
```

或者使用箭头函数

```js
const App = () => {
    return <h1>我是一个React的组件！</h1>;
};

export default App;
```

在其他文件中使用时，需要先通过import进行引入：

```js
index.js
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App/>);
```

引入后通过`<组件名/>`或`<组件名></组件名>`即可引入组件。

### 5.2 组件化编码流程

1.拆分组件:拆分界面，抽取组件

2.实现静态组件

3.实现动态组件

- 动态的显示初始化数据
  - 数据类型
  - 数据名称
  - 保存在哪个组件
- 交互

**注意事项：**

1.拆分组件、实现静态组件。注意className、style的写法

2.动态初始化列表，如何确定将数据放在哪个组件的state中？

- 某个组件使用：放在自身的state中
- 某些组件使用：放在他们共同的父组件中【状态提升】

3.关于父子组件之间的通信

- 父组件给子组件传递数据：通过props传递
- 子组件给父组件传递数据：通过props传递，要求父组件提前给子组件传递一个函数

4.注意defaultChecked 和checked区别，defalutChecked只是在初始化的时候执行一次，checked没有这个限制，但是必须添加onChange方法类似的还有：defaultValue 和value

5.状态在哪里，操作状态的方法就在哪里

## 6.CSS样式

### 6.1 内联样式

在React中可以直接通过标签的style属性来为元素设置样式。style属性需要的是一个对象作为值，来为元素设置样式。

```jsx
<div style={{color:'red'}}>
    我是Div
</div>
```

传递样式时，需要注意如果样式名不符合驼峰命名法，需要将其修改为符合驼峰命名法的名字。比如：background-color改为backgroundColor。

如果内联样式编写过多，会导致JSX变得异常混乱，此时也可以将样式对象定义到JSX外，然后通过变量引入。

样式过多，JSX会比较混乱：

```jsx
const StyleDemo = () => {
    return (
        <div style={{color:'red', backgroundColor:'#bfa', fontSize:20, borderRadius:12}}>
            我是Div
        </div>
    );
};

export default StyleDemo;
```

可以这样修改：

```
import React from 'react';

const StyleDemo = () => {
    const divStyle = {color: 'red', backgroundColor: '#bfa', fontSize: 20, borderRadius: 12}

    return (
        <div style={divStyle}>
            我是Div
        </div>
    );
};

export default StyleDemo;
```

相比第一段代码来说，第二段代码中JSX结构更加简洁。

### 6.2 在内联样式中使用State

设置样式时，可以根据不同的state值应用不同的样式，比如我们可以在组件中添加一个按钮，并希望通过点击按钮可以切换div的边框，代码可以这样写：

```jsx
import React, {useState} from 'react';

const StyleDemo = () => {

    const [showBorder, setShowBorder] = useState(false);

    const divStyle = {
        color: 'red',
        backgroundColor: '#bfa',
        fontSize: 20,
        borderRadius: 12,
        border: showBorder?'2px red solid':'none'
    };

    const toggleBorderHandler = ()=> {
      setShowBorder(prevState => !prevState);
    };

    return (
        <div style={divStyle}>
            我是Div
            <button onClick={toggleBorderHandler}>切换边框</button>
        </div>
    );
};

export default StyleDemo;
```

上例中添加一个新的state，命名为showBorder，代码是这样的`const [showBorder, setShowBorder] = useState(false);`当该值为true时，我们希望div可以显示一条2像素的红色边框，当为false时，我们希望div没有边框。默认值为false。

divStyle的最后一个属性是这样设置的`border: showBorder?'2px red solid':'none'`，这里我们根据showBorder的值来设置border样式的值，如果值为true，则设置边框，否则边框设置为none。

`toggleBorderHandler` 是负责修改showBorder的响应函数，当我们点击按钮后函数会对showBorder进行取反，这样我们的样式就可以根据state的不同值而呈现出不同的效果了。

### 6.3 外部样式表

那么如何为React组件引入样式呢？很简单直接在组件中import即可。例如：我们打算为Button组件编写一组样式，并将其存储到Button.css中。我们只需要直接在Button.js中引入Button.css就能轻易完成样式的设置。

Button.css：

```css
button{
    background-color: #bfa;
}
```

Button.js：

```jsx
import './Button.css';
const Button = () => {
    return <button>我是一个按钮</button>;
};
export default Button;
```

使用这种方式引入的样式，需要注意以下几点：

1. CSS就是标准的CSS语法，各种选择器、样式、媒体查询之类正常写即可。
2. 尽量将js文件和css文件的文件名设置为相同的文件名。
3. 引入样式时直接import，无需指定名字，且引入样式必须以./或../开头。
4. 这种形式引入的样式是全局样式，有可能会被其他样式覆盖。

### 6.4 css模块化

当组件逐渐增多起来的时候，我们发现，组件的样式也是越来越丰富，这样就很有可能产生两个组件中样式名称有可能会冲突，这样会根据引入App这个组件的先后顺序，后面的会覆盖前面的，

为了解决这个问题React中还为我们提供了一中方式，CSS Module。

我们可以将CSS Module理解为外部样式表的一种进化版，它的大部分使用方式都和外部样式表类似，不同点在于使用CSS Module后，网页中元素的类名会自动计算生成并确保唯一，所以使用CSS Module后，我们再也不用担心类名重复了

**使用方式**

CSS Module在React中已经默认支持了（前提是使用了react-scripts），所以无需再引入其他多余的模块。使用CSS Module时需要遵循如下几个步骤：

1.将css文件名修改： `index.css` --- >` index.module.css`

2.引入并使用的时候改变方式：

```js
import React,{Component} from 'react'
import hello from './index.module.css'  //引入的时候给一个名称

export default class Hello extends Component{
    render() {
        return (
            <h1 className={hello.title}>Hello</h1>   //通过大括号进行调用
        )
    }
}
```

这就是一个简单的CSS Module的案例，设置完成后你可以自己通过开发者工具查看元素的class属性，你会发现class属性和你设置的并不完全一样，这是因为CSS Module通过算法确保了每一个模块中类名的唯一性。

总之，相较于标准的外部样式表来说，CSS Module就是多了一点——确保类名的唯一，通过内部算法避免了两个组件中出现重复的类名，如果你能保证不会出现重复的类名，其实直接使用外部样式表也是一样的。

## 7.配置代理

本案例需要下载`axios`库`npm install axios`

React本身只关注与页面，并不包含发送ajax请求的代码，所以一般都是集成第三方的一些库，或者自己进行封装。

推荐使用axios。

在使用的过程中很有可能会出现跨域的问题，这样就应该配置代理。

所谓同源（即指在同一个域）就是两个页面具有相同的协议（protocol），主机（host）和端口号（port）， 当一个请求url的**协议、域名、端口**三者之间任意一个与当前页面url不同即为跨域 。

那么react通过代理解决跨域问题呢

> 利用服务器之间访问不会有跨域，在中间开启一个服务器，端口号和项目端口号一样

### 7.1 方法一

> 在package.json中追加如下配置

```json
"proxy":"请求的地址"      "proxy":"http://localhost:5000"  
```

说明：

1. 优点：配置简单，前端请求资源时可以不加任何前缀。
2. 缺点：不能配置多个代理。
3. 工作方式：上述方式配置代理，当请求了3000不存在的资源时，那么该请求会转发给5000 （优先匹配前端资源）

### 7.2 方法二

**方法二**

1. 第一步：创建代理配置文件

   ```css
   在src下创建配置文件：src/setupProxy.js
   ```

2. 编写setupProxy.js配置具体代理规则：

   ```js
   const proxy = require('http-proxy-middleware')
   
   module.exports = function(app) {
     app.use(
       proxy('/api1', {  //api1是需要转发的请求(所有带有/api1前缀的请求都会转发给5000)
         target: 'http://localhost:5000', //配置转发目标地址(能返回数据的服务器地址)
         changeOrigin: true, //控制服务器接收到的请求头中host字段的值
         /*
         	changeOrigin设置为true时，服务器收到的请求头中的host为：localhost:5000
         	changeOrigin设置为false时，服务器收到的请求头中的host为：localhost:3000
         	changeOrigin默认值为false，但我们一般将changeOrigin值设为true
         */
         pathRewrite: {'^/api1': ''} //去除请求前缀，保证交给后台服务器的是正常请求地址(必须配置)
       }),
       proxy('/api2', { 
         target: 'http://localhost:5001',
         changeOrigin: true,
         pathRewrite: {'^/api2': ''}
       })
     )
   }
   ```

说明：

1. 优点：可以配置多个代理，可以灵活的控制请求是否走代理。
2. 缺点：配置繁琐，前端请求资源时必须加前缀。

### 7.3 vite配置proxy

`vite.config.ts`

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
    // 是否自动打开浏览器
    open: true,
    // 代理
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:5000',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, ''),
      },
    },
  },
})

```


