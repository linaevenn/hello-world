# 19 【RTK Query】

## 1.目前前端常见的发起 ajax 请求的方式

- 1、使用原生的`ajax`请求
- 2、使用`jquery`封装好的`ajax`请求
- 3、使用`fetch`发起请求
- 4、第三方的比如`axios`请求
- 5、`angular`中自带的`HttpClient`

就目前前端框架开发中来说我们在开发`vue`、`react`的时候一般都是使用`fetch`或`axios`自己封装一层来与后端数据交互，至于`angular`肯定是用自带的`HttpClient`请求方式，但是依然存在几个致命的弱点，

- 1、对当前请求数据不能缓存，
- 2、一个页面上由多个组件组成，但是刚好有遇到复用相同组件的时候，那么就会发起多次`ajax`请求

> 📢 针对同一个接口发起多次请求的解决方法，目前常见的解决方案

- 1、使用`axios`的取消发起请求，[参考文档](http://www.axios-js.com/zh-cn/docs/#取消)
- 2、`vue`中还没看到比较好的方法
- 3、在`rect`中可以借用类似[react-query](https://react-query.tanstack.com/)工具对请求包装一层
- 4、对于`angular`中直接使用`rxjs`的操作符`shareReplay`

## 2.RTK Query 概述

RTK不仅帮助我们解决了state的问题，同时，它还为我们提供了RTK Query用来帮助我们处理数据加载的问题。RTK Query是一个强大的数据获取和缓存工具。在它的帮助下，Web应用中的加载变得十分简单，它使我们不再需要自己编写获取数据和缓存数据的逻辑。

`rtk-query`是[`redux-toolkit`](https://redux-toolkit.js.org/)里面的一个分之，专门用来优化前端接口请求，目前也只支持在`react`中使用。

**RTK Query** 是一个强大的数据获取和缓存工具。它旨在简化在 Web 应用程序中加载数据的常见情况，**无需自己手动编写数据获取和缓存逻辑**。

RTK Query 是**一个包含在 Redux Toolkit 包中的可选插件**，其功能构建在 Redux Toolkit 中的其他 API 之上。

**Web应用中加载数据时需要处理的问题：**

1. 根据不同的加载状态显示不同UI组件
2. 减少对相同数据重复发送请求
3. 使用乐观更新，提升用户体验
4. 在用户与UI交互时，管理缓存的生命周期

这些问题，RTKQ都可以帮助我们处理。首先，可以直接通过RTKQ向服务器发送请求加载数据，并且RTKQ会自动对数据进行缓存，避免重复发送不必要的请求。其次，RTKQ在发送请求时会根据请求不同的状态返回不同的值，我们可以通过这些值来监视请求发送的过程并随时中止。

我们将 `createAsyncThunk` 与 `createSlice` 一起使用，在发出请求和管理加载状态方面仍然需要进行大量手动工作。我们必须创建异步 thunk，发出实际请求，从响应中提取相关字段，添加加载状态字段，在 `extraReducers` 中添加处理程序以处理 `pending/fulfilled/rejected` 情况，并实际编写正确的状态更新。

在过去的几年里，React 社区已经意识到 **“数据获取和缓存” 实际上是一组不同于 “状态管理” 的关注点**。虽然你可以使用 Redux 之类的状态管理库来缓存数据，但用例差异较大，因此值得使用专门为数据获取用例构建的工具。

RTK Query 在其 API 设计中添加了独特的方法：

- 数据获取和缓存逻辑构建在 Redux Toolkit 的 `createSlice` 和 `createAsyncThunk` API 之上
- 由于 Redux Toolkit 与 UI 无关，因此 RTK Query 的功能可以与任何 UI 层一起使用
- API 请求接口是提前定义的，包括如何从参数生成查询参数和转换响应以进行缓存
- RTK Query 还可以生成封装整个数据获取过程的 React hooks ，为组件提供 `data` 和 `isFetching` 字段，并在组件挂载和卸载时管理缓存数据的生命周期
- RTK Query 提供“缓存数据项生命周期函数”选项，支持在获取初始数据后通过 websocket 消息流式传输缓存更新等用例
- 我们有从 OpenAPI 和 GraphQL 模式生成 API slice 代码的早期工作示例
- 最后，RTK Query 完全用 TypeScript 编写，旨在提供出色的 TS 使用体验

> 📢 `rtk-query`的使用环境，必须是`react`版本大于 17,可以使用`hooks`的版本，因为使用`rtk-query`的查询都是`hooks`的方式，如果你项目简单`redux`都未使用到，本人不建议你用`rtk-query`，可能直接使用`axios`请求更加的简单方便。

## 3.基础开发流程

> 后面这些案例后端接口返回格式都是
>
> ```json
> {
>     "code":200,
>     "data":[]
> }
> ```

- 创建一个store文件夹
- 创建一个index.ts做为主入口
- 创建一个festures/api文件夹用来装所有的API Slice
- 创建一个sudentApiSlice.js文件，并导出简单的加减方法

### 3.1 定义 API Slice

使用 RTK Query，**管理缓存数据的逻辑被集中到每个应用程序的单个“API Slice”中**。就像每个应用程序只有一个 Redux 存储一样，我们现在有一个Slice 用于 *所有* 我们的缓存数据。

我们将从定义一个新的 `sudentApiSlice.js` 文件开始。由于这不是特定于我们已经编写的任何其他“功能”，我们将添加一个新的 `features/api/` 文件夹并将 `sudentApiSlice.js` 放在那里。让我们填写 API Slice 文件，然后分解里面的代码，看看它在做什么：

`features/api/sudentApiSlice.js`

```js
// 从特定于 React 的入口点导入 RTK Query 方法
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/dist/query/react'
// 上面这个引入的会自动创建钩子
// import { createApi } from '@reduxjs/toolkit/query'

// 定义我们的单个 API Slice 对象
//createApi() 用来创建RTKQ中的API对象
// RTKQ的所有功能都需要通过该对象来进行
// createApi() 需要一个对象作为参数
export const sudentApiSlice = createApi({
  reducerPath: 'studentApi', // Api的标识，不能和其他的Api或reducer重复
  // 指定查询的基础信息，发送请求使用的工具
  baseQuery: fetchBaseQuery({
    // 我们所有的请求都有以 “/api 开头的 URL
    baseUrl: 'http://localhost:8080/api',
  }),
  // “endpoints” 代表对该服务器的操作和请求
  endpoints: builder => ({
    // `getStudents` endpoint 是一个返回数据的 “Query” 操作
    getStudents: builder.query({
      // 请求的 URL 是“/api/all/student”
      query: () => '/all/student',
    }),
  }),
})

// Api对象创建后，对象中会根据各种方法自动的生成对应的钩子函数
// 通过这些钩子函数，可以来向服务器发送请求
// 钩子函数的命名规则 getStudents --> useGetStudentsQuery
export const { useGetStudentsQuery } = sudentApiSlice
```

上例是一个比较简单的Api对象的例子，我们来分析一下，首先我们需要调用`createApi()`来创建Api对象。这个方法在RTK中存在两个版本，一个位于`@reduxjs/toolkit/dist/query`下，一个位于`@reduxjs/toolkit/dist/query/react`下。react目录下的版本会自动生成一个钩子，方便我们使用Api。如果不要钩子，可以引入query下的版本，当然我不建议你这么做。

`createApi()`需要一个配置对象作为参数，配置对象中的属性繁多，我们暂时介绍案例中用到的属性：

**reducerPath**

用来设置reducer的唯一标识，主要用来在创建store时指定action的type属性，如果不指定默认为api。

**baseQuery**

用来设置发送请求的工具，就是你是用什么发请求，RTKQ为我们提供了fetchBaseQuery作为查询工具，它对fetch进行了简单的封装，很方便，如果你不喜欢可以改用其他工具，这里暂时不做讨论。

**fetchBaseQuery**

简单封装过的fetch调用后会返回一个封装后的工具函数。需要一个配置对象作为参数，baseUrl表示Api请求的基本路径，指定后请求将会以该路径为基本路径。配置对象中其他属性暂不讨论。

**endpoints**

Api对象封装了一类功能，比如学生的增删改查，我们会统一封装到一个对象中。一类功能中的每一个具体功能我们可以称它是一个端点。endpoints用来对请求中的端点进行配置。

endpoints是一个回调函数，可以用普通方法的形式指定，也可以用箭头函数。回调函数中会收到一个build对象，使用build对象对点进行映射。回调函数的返回值是一个对象，Api对象中的所有端点都要在该对象中进行配置。

对象中属性名就是要实现的功能名，比如获取所有学生可以命名为getStudents，根据id获取学生可以命名为getStudentById。属性值要通过build对象创建，分两种情况：

查询：`build.query({})`

增删改：`build.mutation({})`

例如：

```js
getStudents: builder.query({
  // 请求的 URL 是“/api/all/student”
  query: () => '/all/student',
}),
```

先说query，query也需要一个配置对象作为参数。配置对象里同样有n多个属性，现在直说一个，query方法。注意不要搞混两个query，一个是build的query方法，一个是query方法配置对象中的属性，这个方法需要返回一个子路径，这个子路径将会和baseUrl拼接为一个完整的请求路径。比如：getStudets的最终请求地址是:

```absh
http://localhost:8080/api + /all/student = http://localhost:8080/api/all/student
```

可算是介绍完了，但是注意了这个只是最基本的配置。RTKQ功能非常强大，但是配置也比较麻烦。不过，熟了就好了。

上例中，我们创建一个Api对象studentApi，并且在对象中定义了一个getStudents方法用来查询所有的学生信息。如果我们使用react下的createApi，则其创建的Api对象中会自动生成钩子函数，钩子函数名字为useXxxQuery或useXxxMutation，中间的Xxx就是方法名，查询方法的后缀为Query，修改方法的后缀为Mutation。所以上例中，Api对象中会自动生成一个名为useGetStudentsQuery的钩子，我们可以获取并将钩子向外部暴露。

```js
export const {useGetStudentsQuery} = studentApi;
```

### 3.2 配置 Store

我们现在需要将 API Slice 连接到我们的 Redux 存储。我们可以修改现有的 `store.js` 文件，将 API slice 的 cache reducer 添加到状态中。此外，API slice 会生成需要添加到 store 的自定义 middleware。这个 middleware *必须* 被添加——它管理缓存的生命周期和控制是否过期。

`store/index.js`

```js
import { configureStore } from '@reduxjs/toolkit'
import { sudentApiSlice } from './features/api/sudentApiSlice'

// configureStore创建一个redux数据
const store = configureStore({
  // 合并多个Slice
  reducer: {
    [sudentApiSlice.reducerPath]: sudentApiSlice.reducer,
  },
  middleware: getDefaultMiddleware => getDefaultMiddleware().concat(sudentApiSlice.middleware),
})

export default store
```

我们可以在 `reducer` 参数中重用 `sudentApiSlice.reducerPath` 字段作为计算键，以确保在正确的位置添加缓存 reducer。

我们需要在 store 设置中保留所有现有的标准 middleware，例如“redux-thunk”，而 API slice 的 middleware 通常会在这些 middleware 之后使用。我们可以通过向 `configureStore` 提供 `middleware` 参数，调用提供的 `getDefaultMiddleware()` 方法，并在返回的 middleware 数组的末尾添加 `sudentApiSlice.middleware` 来做到这一点。

store创建完毕同样要设置Provider标签，这里不再展示。

### 3.3 在组件中使用 Query Hooks

接下来，我们来看看如果通过studentApi发送请求。由于我们已经将studentApi中的钩子函数向外部导出了，所以我们只需通过钩子函数即可自动加载到所有的学生信息。比如，现在在App.js中加载信息可以这样编写代码：

```jsx
import React from 'react'
import { useGetStudentsQuery } from './store/features/api/sudentApiSlice'

export default function App() {
  const { data:studentsRes, isLoading, isSuccess, isError, error } = useGetStudentsQuery()

  let content
  if (isLoading) {
    content = '正在加载中'
  } else if (isSuccess) {
    content = studentsRes.data.map(stu => (
      <p key={stu._id}>
        {stu.name} ---
        {stu.age} ---
        {stu.sex}
      </p>
    ))
  } else if (isError) {
    content = error.toString()
  }

  return <div>{content}</div>
}
```

我们能够用对 `useGetStudentsQuery()` 的单个调用来替换多个 `useSelector` 调用和 `useEffect` 调度。

直接调用`useGetStudentsQuery()`它会自动向服务器发送请求加载数据，每个生成的 Query hooks 都会返回一个包含多个字段的“结果”对象，包括：

1. `data` – 来自服务器的实际响应内容。 **在收到响应之前，该字段将是 “undefined”**。
2. `currentData` – 当前的数据
3. `isUninitialized` – 如果为true则表示查询还没开始
4. `data`:来自服务器的实际响应内容。 **在收到响应之前，该字段将是 “undefined”**。
5. `isLoading`: 一个 boolean，指示此 hooks 当前是否正在向服务器发出 *第一次* 请求。（请注意，如果参数更改以请求不同的数据，`isLoading` 将保持为 false。）
6. `isFetching`: 一个 boolean，指示 hooks 当前是否正在向服务器发出 *any* 请求
7. `isSuccess`: 一个 boolean，指示 hooks 是否已成功请求并有可用的缓存数据（即，现在应该定义 data）
8. `isError`: 一个 boolean，指示最后一个请求是否有错误
9. `error`: 一个 serialized 错误对象
10. `refetch `函数，用来重新加载数据

从结果对象中解构字段是很常见的，并且可能将 `data` 重命名为更具体的变量，例如 `studentRes` 来描述它包含的内容。然后我们可以使用状态 boolean 和 `data/error` 字段来呈现我们想要的 UI。 但是，如果你使用的是 TypeScript，你可能需要保持原始对象不变，并在条件检查中将标志引用为 `result.isSuccess`，以便 TS 可以正确推断 `data` 是有效的。

![Snipaste_2022-11-04_22-57-20](https://i0.hdslb.com/bfs/album/f1cc5755cb70b40751511b3dbf6ebdb2cfab7133.png)

这是最终效果：

![image-20221104231936997](https://i0.hdslb.com/bfs/album/e0818fbcb9a2c521bcf4ded7d67336911813198e.png)

## 4.传递参数

### 4.1 定义接收参数

`features/api/sudentApiSlice.js`

这里定义了一个新的接口，通过id获取学生信息

```js
// 从特定于 React 的入口点导入 RTK Query 方法
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/dist/query/react'

export const sudentApiSlice = createApi({
  reducerPath: 'studentApi',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api',
  }),
  endpoints: builder => ({
    getStudentById: builder.query({
      // 从query方法这里接收参数
      query: sutId => `/student/${sutId}`,
    }),
  }),
})

export const { useGetStudentsQuery, useGetStudentByIdQuery } = sudentApiSlice
```

### 4.2 传递参数

`App.jsx`

```jsx
import React from 'react'
import { useGetStudentByIdQuery } from './store/features/api/sudentApiSlice'

export default function App() {
  let stuId = '63652d2c03155b63eea7b9f5'
  const {
    data: studentRes,
    isLoading,
    isSuccess,
    isError,
    error,
  } = useGetStudentByIdQuery(stuId)
  
  let content
  if (isLoading) {
    content = '正在加载中'
  } else if (isSuccess) {
    content = (
      <p>
        {studentRes.data.name} ---{studentRes.data.age} ---{studentRes.data.sex}
      </p>
    )
  } else if (isError) {
    content = error.toString()
  }

  return <div>{content}</div>
}
```

`useGetPostQuery`这个钩子接收的第一个参数传递到`query`方法，使用起来很简单

![image-20221105135329712](https://i0.hdslb.com/bfs/album/8f9e3ec4a807396819fcb6d9ef6628816da0df14.png)

## 5.转换响应

**请求接口可以定义一个 `transformResponse` 处理程序，该处理程序可以在缓存之前提取或修改从服务器接收到的数据**。我们可以有 `transformResponse: (responseData) => responseData.data`，它只会缓存实际的 `student` 对象，而不是整个响应体。

`features/api/sudentApiSlice.js`

```js
getStudentById: builder.query({
  query: sutId => `/student/${sutId}`,
  transformResponse:(responseData, meta, arg)=>{
    console.log(responseData);
    return responseData.data
  }
}),
```

对于上一个案例中通过id获取学生信息的接口，加一个`transformResponse`方法，我们来看看他接受到的参数`responseData`是什么

![image-20221105140454626](https://i0.hdslb.com/bfs/album/8e08b7d67470ca1eeafadf751a723966090f855e.png)

可以看到`responseData`这个参数就是我们的响应体

在使用的过程中，发现这个方法类似于响应拦截器。

我们在`App.jsx`中看看`useGetStudentByIdQuery`这个钩子函数返回的`data`有什么变化

![image-20221105140911763](https://i0.hdslb.com/bfs/album/79bbe6858e5a55b3f62201d3db8643984d69e29b.png)

## 6.RTK Query 缓存简单介绍

> 后面在介绍缓存的灵活使用

### 6.1 什么是相同查询

RTK Query 会将查询查询参数序列化为字符串，并将相同钩子、相同参数的查询视为相同的查询，他们将共享一个请求与缓存数据。

因此，下面两个调用返回结果相同（即使在不同的组件中）：

```js
useGetXXXQuery({ a: 1, b: 2 }) // 订阅 + 1
useGetXXXQuery({ b: 2, a: 1 }) // 订阅 + 1
// ...
```

这是因为：

- 他们使用相同的查询：GetXXX
- 查询参数的序列化结果相同：`'{"a":1,"b":2}'`

你不需要担心嵌套或是字段顺序，或是不同对象不同引用会被认为是不同的查询，因为 RTK Query 已经在默认的序列化函数中处理了相关用例。同时，你也可以提供自己的序列化函数。

### 6.2 引用计数与垃圾回收

当在组件中使用某个查询时，该查询的引用计数会 + 1，当该组件被卸载时，引用计数会 -1。当一个查询的引用计数为 0 时，说明没有任何组件在使用这个查询。此时，经过 `keepUnusedDataFor`（默认为 30 ）秒后，如果缓存仍为被使用过，那么他将被从缓存中移除。

### 6.3 缓存初体验

缓存的配置

`store/index.js`

```js
// configureStore创建一个redux数据
const store = configureStore({
  // 合并多个Slice
  reducer: {
    [sudentApiSlice.reducerPath]: sudentApiSlice.reducer,
  },
  middleware: getDefaultMiddleware => getDefaultMiddleware().concat(sudentApiSlice.middleware),
})

export default store

```

其实就是在这个`store`配置这个中间件，一开始就配好了。

来看看实际效果

先改写下上面的案例

`App.jsx`

```jsx
import React, { useState } from 'react'
import StudentA from './StudentA'
import StudentB from './StudentB'

export default function App() {
  const [tab, setTab] = useState(0)

  let content
  switch (tab) {
    case 0:
      content = '首页'
      break
    case 1:
      content = <StudentA />
      break
    case 2:
      content = <StudentB />
      break
  }

  return (
    <div>
      <p>
        <button onClick={() => setTab(0)}>首页</button>
        <button onClick={() => setTab(1)}>学生A</button>
        <button onClick={() => setTab(2)}>学生B</button>
      </p>
      {content}
    </div>
  )
}
```

`StudentA.jsx`

```jsx
import React from 'react'
import { useEffect } from 'react'
import { useGetStudentByIdQuery } from './store/features/api/sudentApiSlice'

export default function Student() {
  let stuId = '63652d2c03155b63eea7b9f5'
  const { data: studentRes, isLoading, isSuccess, isError, error } = useGetStudentByIdQuery(stuId)

  let content
  if (isLoading) {
    content = '正在加载中'
  } else if (isSuccess) {
    content = (
      <p>
        {studentRes.data.name} ---{studentRes.data.age} ---{studentRes.data.sex}
      </p>
    )
  } else if (isError) {
    content = error.toString()
  }

  useEffect(() => {
    console.log('渲染了')
  }, [])

  return (
    <>
      <p>组件StudentA</p>
      {content}
    </>
  )
}
```

`StudentB.jsx`

```jsx
import React from 'react'
import { useEffect } from 'react'
import { useGetStudentByIdQuery } from './store/features/api/sudentApiSlice'

export default function Student() {
  let stuId = '63652d2c03155b63eea7b9f5'
  const { data: studentRes, isLoading, isSuccess, isError, error } = useGetStudentByIdQuery(stuId)

  let content
  if (isLoading) {
    content = '正在加载中'
  } else if (isSuccess) {
    content = (
      <p>
        {studentRes.data.name} ---{studentRes.data.age} ---{studentRes.data.sex}
      </p>
    )
  } else if (isError) {
    content = error.toString()
  }

  useEffect(() => {
    console.log('渲染了')
  }, [])

  return (
    <>
      <p>组件StudentB</p>
      {content}
    </>
  )
}
```

我们把学生信息抽离成两个组件，里面除了标题都是一样的，在`App.jsx`中设置了个三个按钮控制显示隐藏

切换到`StudentA`组件

![image-20221105151052394](https://i0.hdslb.com/bfs/album/82cdd432c4ba92162d0b00a97c17086e4319de99.png)

切换到`StudentB`组件

![image-20221105151112395](https://i0.hdslb.com/bfs/album/5cb7363a0c93dbea2ce3c1a6f2243ba4012e57c4.png)

可以看到切换到`StudentB`组件并没有重新发起请求，这就是缓存生效了。

**RTK Query 允许多个组件订阅相同的数据，并且将确保每个唯一的数据集只获取一次。** 在内部，RTK Query 为每个请求接口 + 缓存键组合保留一个 action 订阅的引用计数器。如果组件 A 调用 `useGetStudentByIdQuery(stuId)`，则将获取该数据。如果组件 B 挂载并调用 `useGetStudentByIdQuery(stuId)`，则请求的数据完全相同。两种钩子用法将返回完全相同的结果，包括获取的 “data” 和加载状态标志。

当活跃订阅数下降到 0 时，RTK Query 会启动一个内部计时器。 **如果在添加任何新的数据订阅之前计时器到期，RTK Query 将自动从缓存中删除该数据**，因为应用程序不再需要该数据。但是，如果在计时器到期之前添加了新订阅，则取消计时器，并使用已缓存的数据而无需重新获取它。

在这种情况下，我们的 `<StudentA>` 挂载并通过 ID 请求。当我们“切换” 时，`<StudentA>` 组件被路由器卸载，并且活动订阅由于卸载而被删除。RTK Query 立即启动 “remove this post data” 计时器。但是，`<StudentB>` 组件会立即挂载并使用相同的缓存键订阅相同的 `student` 数据。因此，RTK Query 取消了计时器并继续使用相同的缓存数据，而不是从服务器获取数据。

默认情况下，**未使用的数据会在 60 秒后从缓存中删除**，但这可以在根 API Slice 定义中进行配置，也可以使用 `keepUnusedDataFor` 标志在各个请求接口定义中覆盖，该标志指定缓存生存期 秒。

`features/api/sudentApiSlice.js`

```js
getStudentById: builder.query({
  // 从query方法这里接收参数
  query: sutId => `/student/${sutId}`,
  keepUnusedDataFor: 60, // 设置数据缓存的时间，单位秒 默认60s
}),
```

## 7.mutation 请求接口

我们已经看到了如何通过定义查询请求接口从服务器获取数据，但是向服务器发送更新呢？

RTK Query 让我们定义 **mutation 请求接口** 来更新服务器上的数据。让我们添加一个可以让我们添加新学生的 Mutation。

### 7.1 添加新的 Mutations 后请求接口

添加 Mutation 请求接口与添加查询请求接口非常相似。 最大的不同是我们使用 `builder.mutation()` 而不是 `builder.query()` 来定义请求接口。 此外，我们现在需要将 HTTP 方法更改为“POST”请求，并且我们还必须提供请求的正文。

`features/api/sudentApiSlice.js`

```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/dist/query/react'

export const sudentApiSlice = createApi({
  reducerPath: 'studentApi', 
  baseQuery: fetchBaseQuery({
    baseUrl: '/api',
  }),
  endpoints: builder => ({
    getStudents: builder.query({
      query: () => '/all/student',
    }),
    getStudentById: builder.query({
      query: sutId => `/student/${sutId}`
    }),
    addNewStudent: builder.mutation({
      query: student => ({
        url: '/student',
        method: 'POST',
        // 将整个post对象作为请求的主体
        body: student,
      }),
    }),
  }),
})

export const { useGetStudentsQuery, useGetStudentByIdQuery, useAddNewStudentMutation } =
  sudentApiSlice
```

这里我们的 `query` 选项返回一个包含 `{url, method, body}` 的对象。 由于我们使用 `fetchBaseQuery` 来发出请求，`body` 字段将自动为我们进行 JSON 序列化。

与查询请求接口一样，API slice 会自动为 Mutation 请求接口生成一个 React hooks - 在本例中为 `useAddNewPostMutation`。

### 7.2 在组件中使用 Mutation Hooks

每当我们单击“添加”按钮时，我们以前得调度了一个异步 thunk 来添加。 为此，它必须导入 `useDispatch` 和 `addNewPost` thunk。 Mutation hooks 取代了这两者，并且使用模式非常相似。

```jsx
import React, { useState } from 'react'
import { useAddNewStudentMutation } from './store/features/api/sudentApiSlice'

export default function Home() {
  const [name, setName] = useState('')
  const [age, setAge] = useState(0)
  const [sex, setSex] = useState('')

  // 获取添加的钩子，useMutation的钩子返回的是一个数组
  // 数组中有两个东西，第一个是操作的触发器，第二个是结果集
  const [addNewStudent, { isLoading }] = useAddNewStudentMutation()

  const canSubmit = [name, age, sex].every(() => true) && !isLoading

  const onAddStuClicked = async () => {
    if (!canSubmit) return
    try {
      await addNewStudent({ name, age, sex }).unwrap()
      setName('')
      setAge(0)
      setSex('')
    } catch (err) {
      console.error('Failed to add student: ', err)
    }
  }
  
  return (
    <div>
      <h2>首页</h2>
      <p>
        <button onClick={onAddStuClicked}>添加学生</button>
      </p>
      <form>
        姓名：
        <input type="text" value={name} onChange={e => setName(e.target.value)} /> <br />
        年龄：
        <input type="number" value={age} onChange={e => setAge(+e.target.value)} /> <br />
        性别：
        <input type="text" value={sex} onChange={e => setSex(e.target.value)} />
      </form>
    </div>
  )
}
```

Mutation hooks 返回一个包含两个值的数组：

- 第一个值是触发函数。当被调用时，它会使用你提供的任何参数向服务器发出请求。这实际上就像一个已经被包装以立即调度自身的 thunk。
- 第二个值是一个对象，其中包含有关当前正在进行的请求（如果有）的元数据。这包括一个 `isLoading` 标志以指示请求是否正在进行中。

我们可以用 `useAddNewStudentMutation` hooks 中的触发函数和 `isLoading` 标志替换现有的 thunk 调度和组件加载状态，组件的其余部分保持不变。

与 thunk 调度一样，我们使用初始 post 对象调用 `addNewStudent`。 这会返回一个带有` .unwrap() `方法的特殊 Promise ，我们可以 `await addNewStudent().unwrap()` 使用标准的 `try/catch` 块来处理任何潜在的错误。

![image-20221105193209910](https://i0.hdslb.com/bfs/album/218da1adc96ca99b2f9ee57c5b7a8c96ef48448d.png)

## 8.useQuery Hook 参数

查询钩子需要两个参数：`(queryArg?, queryOptions?)`

参数将被传递到底层回调以生成URL。`queryArg` `query`

该对象接受几个可用于控制数据获取行为的附加参数：`queryOptions`

- [skip](https://redux-toolkit.js.org/rtk-query/usage/conditional-fetching)  - 允许查询“跳过”为该渲染运行。默认为`false`
- [pollingInterval](https://redux-toolkit.js.org/rtk-query/usage/polling)  - 允许查询按提供的时间间隔（以毫秒为单位指定）自动重新获取。默认为*（关闭）*`0`
- [selectFromResult](https://redux-toolkit.js.org/rtk-query/usage/queries#selecting-data-from-a-query-result) - 允许更改钩子的返回值以获取结果的子集，针对返回的子集进行渲染优化。
- [refetchOnMountOrArgChange](https://redux-toolkit.js.org/rtk-query/api/createApi#refetchonmountorargchange) - 允许强制查询始终在挂载时重新取回迁（何时提供）。允许在自上次查询同一缓存（当设置为`true`）以来已经过去了足够的时间（以秒为单位）时强制查询重新获取。默认为`true` `number` `false`
- [refetchOnFocus](https://redux-toolkit.js.org/rtk-query/api/createApi#refetchonfocus)  - 允许在浏览器窗口重新获得焦点时强制查询重新获取。默认为`false`
- [refetchOnReconnect](https://redux-toolkit.js.org/rtk-query/api/createApi#refetchonreconnect)  - 允许在重新获得网络连接时强制查询重新获取。默认为`false`

### 8.1 条件提取

默认为`false`。一旦挂载组件，查询钩子就会自动开始获取数据。但是，在某些用例中，您可能希望延迟获取数据，直到某些条件变为真。RTK 查询支持条件提取以启用该行为。

如果要阻止查询自动运行，可以在钩子中使用参数`skip`

跳过示例

```jsx
const Pokemon = ({ name, skip }) => {
  const { data, error, status } = useGetPokemonByNameQuery(name, {
    skip,
  });

  return (
    <div>
      {name} - {status}
    </div>
  );
};
```

- 如果查询缓存了数据：
  - 缓存的数据**将不会在**初始加载时使用，并且将忽略来自任何相同查询的更新，直到删除条件`skip`
  - 查询的状态为`uninitialized`
  - 初始加载后设置的 ifis，将使用缓存结果`skip: false`
- 如果查询没有缓存数据：
  - 查询的状态为`uninitialized`
  - 使用开发工具查看查询时，查询将不存在于该状态
  - 查询不会在装载时自动获取
  - 当添加具有相同查询的其他组件时，查询不会自动运行

这里我想演示的例子是如果我们给钩子函数传递的参数是一个`undefined`，这个时候发起请求是会报错的，我们可以使用`skip`来来跳过这次无法进行的请求。

```js
import React from 'react'
import {useGetStudentByIdQuery} from "./store/features/api/sudentApiSlice"

const StudentForm = (props) => {
    // 调用钩子来加载数据
    const {data:stuData, isSuccess, isFetching} = useGetStudentByIdQuery(props.stuId, {
        skip:!props.stuId
    })
    ...
}

export default StudentForm;
```

这里如果父组件传过来的`stuId`是个`undefined`,那么这次就不会发起请求了。

### 8.2 轮询

默认值为`0`。轮询使您能够通过使查询按指定的时间间隔运行来产生“实时”效果。若要为查询启用轮询，请以毫秒为单位的间隔将值传递给钩子

```jsx
import React from 'react'
import { useGetPokemonByNameQuery } from './services/pokemon'

export const Pokemon = ({ name }: { name: string }) => {
  // 每过3s会自动调用一次这个钩子函数
  const { data, status, error, refetch } = useGetPokemonByNameQuery(name, {
    pollingInterval: 3000,
  })

  return <div>{data}</div>
}
```

### 8.3 从查询结果中选择数据

`selectFromResult`允许您以高性能方式从查询结果中获取特定段。使用此功能时，除非所选项的基础数据已更改，否则组件不会重新呈现。如果所选项是较大集合中的一个元素，它将忽略对同一集合中元素的更改。

`AllStudent.jsx`

```jsx
import React from 'react'
import { useGetStudentsQuery } from './store/features/api/sudentApiSlice'

export default function App() {
  const {
    data: studentsRes,
    isLoading,
    isSuccess,
    isError,
    error,
  } = useGetStudentsQuery(null, {
    selectFromResult: result => {
      console.log(result)
      return result
    },
  })

  let content
  if (isLoading) {
    content = '正在加载中'
  } else if (isSuccess) {
    content = studentsRes.data.map(stu => (
      <p key={stu._id}>
        {stu.name} ---
        {stu.age} ---
        {stu.sex}
      </p>
    ))
  } else if (isError) {
    content = error.toString()
  }

  return <div>{content}</div>
}
```

先看这个`selectFromResult`方法的参数是什么

![image-20221105195834361](https://i0.hdslb.com/bfs/album/0b94738405c63d074254fc70485f6449e4f9f222.png)

这里我们可以对学生数据进行过滤

```js
selectFromResult: result => {
  let res = result.data
  if (res) {
    result.data = {
      ...res,
      data: res.data.filter(stu => stu.age > 20),
    }
  }
  return result
},
```

![image-20221105201840326](https://i0.hdslb.com/bfs/album/1dd255644e0bc2800e6fd13a4a5440fe5818dc38.png)

### 8.4 refetchOnMountOrArgChange

默认为`false`。此设置允许您控制缓存结果是否已经可用 RTK 查询将仅提供缓存的结果，或者是否应该设置为 或 自上次成功查询结果以来已经过去了足够的时间。

- `false`- 除非查询尚不存在*，否则*不会导致执行查询。
- `true`- 在添加查询的新订阅者时，将始终重新获取。行为与调用回调或传入操作创建者相同。
- `number` - **值以秒为单位**。如果提供了一个数字，并且缓存中存在现有查询，它将比较当前时间与上次实现的时间戳，并且仅在经过足够时间时才重新获取。

如果同时指定此选项`skip: true`，则在 false 之前**不会对其进行计算**。

```js
  const {
    data: studentsRes,
    isLoading,
    isSuccess,
    isError,
    error,
  } = useGetStudentsQuery(null, {
	 refetchOnMountOrArgChange:false
  })
```

> 注意
>
> [fetchBaseQuery |Redux Toolkit (redux-toolkit.js.org)](https://redux-toolkit.js.org/rtk-query/api/createApi#refetchonmountorargchange)
>
> 您可以在`createApi`中全局设置此项`refetchOnMountOrArgChange`，但您也可以覆盖默认值，并通过传递给每个单独的钩子调用或类似地通过 passingwhen 调度[`启动`](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints#initiate)操作来获得更精细的控制。`createApi`

### 8.5 refetchOnFocus

默认值为`false`。此设置允许您控制 RTK 查询是否在应用程序窗口重新获得焦点后尝试重新获取所有订阅的查询。

如果同时指定此选项`skip: true`，则在 false 之前**不会对其进行计算**。

注意：要求已调用[`安装程序侦听器`](https://redux-toolkit.js.org/rtk-query/api/setupListeners)。

> 注意
>
> [fetchBaseQuery |Redux Toolkit (redux-toolkit.js.org)](https://redux-toolkit.js.org/rtk-query/api/createApi#refetchonfocus)
>
> 您可以在`createApi`中全局设置中此项`refetchOnFocus`，但也可以覆盖默认值，并通过传递给每个单独的钩子调用或在调度[`启动`](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints#initiate)操作时进行更精细的控制。
>
> 如果您指定手动分派查询的时间，RTK 查询将无法自动为您重新获取。

想使用还得为`store`添加一个配置才行

`store/index.js`

```js
import { configureStore } from '@reduxjs/toolkit'
import { setupListeners } from '@reduxjs/toolkit/query'

// configureStore创建一个redux数据
const store = configureStore({
 ...
})
    
// 设置以后，将会支持 refetchOnFocus refetchOnReconnect
setupListeners(store.dispatch) 

export default store
```

然后我们看下效果

![image-20221105203424540](https://i0.hdslb.com/bfs/album/b1516ee2e8d603cde783a51a7b1438bdf6735714.png)

从`devtool`回来点一下网页会重新发一次请求，然后从别的网站点回来也会重新发起请求。

### 8.6 refetchOnReconnect

默认值为`false`，此设置允许您控制 RTK 查询在重新获得网络连接后是否尝试重新获取所有订阅的查询。

如果同时指定此选项`skip: true`，则在 false 之前**不会对其进行计算**。

注意：要求已调用[`安装程序侦听器`](https://redux-toolkit.js.org/rtk-query/api/setupListeners)。

> 注意
>
> 您可以在`createApi`中全局设置此项`refetchOnReconnect`，但也可以覆盖默认值，并通过传递给每个单独的钩子调用或在调度[`启动`](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints#initiate)操作时进行更精细的控制。
>
> 如果您指定手动分派查询的时间，RTK 查询将无法自动为您重新获取。`track: false`

## 9.刷新缓存数据

当我们点击`添加学生`时，我们可以在浏览器 DevTools 中查看 Network 选项卡，确认 HTTP `POST` 请求成功。 但是，如果我们回到`所有学生组件`，新的学生信息并不会被展示出来。我们在内存中仍然有相同的缓存数据。

我们需要告诉 RTK Query 刷新其缓存的学生列表，以便我们可以看到我们刚刚添加的新学生信息。

### 9.1 手动刷新

第一个选项是手动强制 RTK Query 重新获取给定请求接口的数据。Query hooks 结果对象包含一个 “refetch” 函数，我们可以调用它来强制重新获取。 我们可以暂时将“重新获取学生列表”按钮添加到`<AllStudent>`，并在添加新学生后单击该按钮。

`AllStudent.jsx`

```jsx
import React from 'react'
import { useGetStudentsQuery } from './store/features/api/sudentApiSlice'

export default function App() {
  const {
    data: studentsRes,
    isLoading,
    isSuccess,
    isError,
    error,
    refetch,
  } = useGetStudentsQuery()

  let content
  if (isLoading) {
    content = '正在加载中'
  } else if (isSuccess) {
    content = studentsRes.data.map(stu => (
      <p key={stu._id}>
        {stu.name} ---
        {stu.age} ---
        {stu.sex}
      </p>
    ))
  } else if (isError) {
    content = error.toString()
  }

  return (
    <div>
      <p>
        <button onClick={refetch}>重新获取学生列表</button>
      </p>
      {content}
    </div>
  )
}
```

首先先从首页添加一个学生数据,然后回到`所有学生组件`

![image-20221106161045089](https://i0.hdslb.com/bfs/album/60d83adac6a06c893f4642031d0af0256e755a2b.png)

这个时候由于有缓存，用的还是之前的数据，我们使用`reFetch`方法来强制刷新数据

![image-20221106161736244](https://i0.hdslb.com/bfs/album/867b54faf816fb065d0b2a6393652e9f5c49952f.png)

### 9.2 缓存失效自动刷新-数据标签

有时需要让用户手动单击以重新获取数据，但对于正常使用而言绝对不是一个好的解决方案。

我们知道我们的服务器拥有所有帖子的完整列表，包括我们刚刚添加的帖子。 理想情况下，我们希望我们的应用程序在 Mutation 请求完成后自动重新获取更新的帖子列表。 这样我们就知道我们的客户端缓存数据与服务器所拥有的数据是同步的。

**RTK Query 让我们定义查询和 mutations 之间的关系，以启用自动数据重新获取，使用标签**。标签是一个字符串或小对象，可让你命名某些类型的数据和缓存的 *无效* 部分。当缓存标签失效时，RTK Query 将自动重新获取标记有该标签的请求接口。

基本标签使用需要向我们的 API slice 添加三条信息：

- API slice 对象中的根 `tagTypes` 字段，声明数据类型的字符串标签名称数组，例如 `'student'`
- 查询请求接口中的 “providesTags” 数组，列出了一组描述该查询中数据的标签
- Mutation 请求接口中的“invalidatesTags”数组，列出了每次 Mutation 运行时失效的一组标签

我们可以在 API slice 中添加一个名为 `'student'` 的标签，让我们在添加新帖子时自动重新获取 `getStudents` 请求接口：

`features/api/sudentApiSlice.js`

```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/dist/query/react'

export const sudentApiSlice = createApi({
  reducerPath: 'studentApi', 
  baseQuery: fetchBaseQuery({
    baseUrl: '/api',
  }),
  tagTypes: ['student'],
  endpoints: builder => ({
    getStudents: builder.query({
      query: () => '/all/student',
      providesTags: [{ type: 'student', id: 'LIST' }],
    }),
    addNewStudent: builder.mutation({
      query: student => ({
        url: '/student',
        method: 'POST',
        // 将整个post对象作为请求的主体
        body: student,
      }),
      invalidatesTags: [{ type: 'student', id: 'LIST' }],
    }),
  }),
})

export const { useGetStudentsQuery,useAddNewStudentMutation } = sudentApiSlice
```

这就是我们所需要的！ 现在，如果我们单击`添加学生`，然后回到`AllStudent`组件重新发起请求，渲染新的数据

请注意，这里的文字字符串 `'student'` 没有什么特别之处。 我们可以称它为“Fred”、“qwerty”或其他任何名称。 它只需要在每个字段中使用相同的字符串，以便 RTK Query 知道“当发生这种 Mutation 时，使列出相同标签字符串的所有请求接口无效”。

## 10.RTKQ 结合 Axios

先来看看一个简单的案例：

```js
import React from "react"
import ReactDOM from "react-dom/client"
import { Provider } from "react-redux"
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/dist/query/react"
import { configureStore } from "@reduxjs/toolkit"
import { setupListeners } from "@reduxjs/toolkit/dist/query"

const productApi = createApi({
    reducerPath: "productApi",
    baseQuery: fetchBaseQuery({
        baseUrl:
            "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store",
    }),
    endpoints(build) {
        return {
            getProducts: build.query({
                query() {
                    return {
                        url: "/products.json",
                    }
                },
            }),
        }
    },
})

const { useGetProductsQuery } = productApi

const store = configureStore({
    reducer: {
        [productApi.reducerPath]: productApi.reducer,
    },

    middleware: (getDefaultMiddleware) =>
        getDefaultMiddleware().concat(productApi.middleware),
})

setupListeners(store.dispatch)

const App = () => {
    const { data, isSuccess } = useGetProductsQuery()

    return (
        <div>
            App
            <hr />
            {isSuccess && JSON.stringify(data)}
        </div>
    )
}

const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
    <Provider store={store}>
        <App />
    </Provider>
)
```

上例中`productApi`用来调用product数据，定义api时的baseQuery属性用来指定我们要使用的发送请求的工具，其中的fetchBaseQuery是RTKQ中为我们提供的工具，它对Fetch进行了包装，设置后RTKQ中将会使用Fetch做为发送请求的工具。

### 10.1 BaseQuery

要设置通过Axios发送请求，关键就在于BaseQuery。只需要使用Axios的BaseQuery替换掉fetchBaseQuery即可。但是可惜的是RTKQ中并没有为我们提供Axios的BaseQuery，所以我们需要自定义一个BaseQuery才能达到目的。

BaseQuery本身就是一个函数，定义BaseQuery直接定义一个函数即可，可以通过函数的参数来指定查询中要使用的默认参数，比如baseUrl，参数可以根据自己的实际需要指定：

```js
const myBaseQuery = ({baseUrl} = {baseUrl:""}) => {
    
}
```

BaseQuery需要一个函数作为返回值，这个函数将会成为最终的发送请求的工具，且函数的返回值将会作为执行结果返回。我们可以将发送请求的逻辑编写到函数中，并且根据不同的情况设置返回值。

先看看返回值的格式，返回值的格式有两种，一种是请求成功返回的数据，一种是请求失败返回的数据：

```js
return { data: YourData } // 请求成功返回的数据
return { error: YourError } // 请求失败返回的数据
```

我们先尝试定义一个简单的BaseQuery：

```js
const myBaseQuery = () => {
  return () => {
      if(Math.random() > .5){
        return {
          data:{name:"孙悟空"}
        }
      }else{
        return {
          error:{message:"出错了"}
        }
      }
  }
}
```

这个BaseQuery不会真的去加载数据，而是根据随机数返回不同的数据。随机数大于0.5时会返回成功的数据，否则返回错误的数据。接下来修改Api的代码，将fetchBaseQuery修改为，myBaseQuery：

```js
const productApi = createApi({
    reducerPath: "productApi",
    baseQuery: myBaseQuery(),
    endpoints(build) {
        return {
            getProducts: build.query({
                query() {
                    return {
                        url: "/products.json",
                    }
                },
            }),
        }
    },
})
```

### 10.2 AxiosBaseQuery

如果你能理解myBaseQuery，下边我们尝试编写一个axiosBaseQuery：

```js
const axiosBaseQuery = ({baseUrl} = {baseUrl:""}) => {
    return ({url, method, data, params}) => {
        return axios({
            url: baseUrl + url,
            method,
            data,
            params
        })
    }
}
```

直接使用axiosBaseQuery替换掉之前的BaseQuery，即可在RTKQ中使用Axios来发送请求了，同时我们也可以根据需要在BaseQuery中对axios做一些更详细的配置。

## 11.小总结

- RTK Query 是 Redux Toolkit 中包含的数据获取和缓存解决方案
  - RTK Query 为你抽象了管理缓存服务器数据的过程，无需编写加载状态、存储结果和发出请求的逻辑
  - RTK Query 建立在 Redux 中使用的相同模式之上，例如异步 thunk
- RTK Query 对每个应用程序使用单个 “API slice”，使用 `createApi` 定义
  - RTK Query 提供与 UI 无关和特定于 React 的 `createApi` 版本
  - API slice 为不同的服务器操作定义了多个“请求接口”
  - 如果使用 React 集成，API slice 包括自动生成的 React hooks
- 查询请求接口允许从服务器获取和缓存数据
  - Query Hooks 返回一个 “data” 值，以及加载状态标志
  - 查询可以手动重新获取，或者使用标签自动重新获取缓存失效
- Mutation 请求接口允许更新服务器上的数据
  - Mutation hooks 返回一个发送更新请求的“触发”函数，以及加载状态
  - 触发函数返回一个可以解包并等待的 Promise



# 20 【react中使用ts】

官方文档：[React TypeScript Cheatsheets | React TypeScript Cheatsheets (react-typescript-cheatsheet.netlify.app)](https://react-typescript-cheatsheet.netlify.app/)

> 找了好久才找到

## 1.创建一个组件

下面我们将要创建一个`Hello`组件。 这个组件接收任意一个我们想对之打招呼的名字（我们把它叫做`name`），并且有一个可选数量的感叹号做为结尾（通过`enthusiasmLevel`）。

若我们这样写`<Hello name="Daniel" enthusiasmLevel={3} />`，这个组件大至会渲染成`<div>Hello Daniel!!!</div>`。 如果没指定`enthusiasmLevel`，组件将默认显示一个感叹号。 若`enthusiasmLevel`为`0`或负值将抛出一个错误。

下面来写一下`Hello.tsx`：

```tsx
// src/components/Hello.tsx

import * as React from 'react';

export interface Props {
  name: string;
  enthusiasmLevel?: number;
}

function Hello({ name, enthusiasmLevel = 1 }: Props) {
  if (enthusiasmLevel <= 0) {
    throw new Error('You could be a little more enthusiastic. :D');
  }

  return (
    <div className="hello">
      <div className="greeting">
        Hello {name + getExclamationMarks(enthusiasmLevel)}
      </div>
    </div>
  );
}

export default Hello;

// helpers

function getExclamationMarks(numChars: number) {
  return Array(numChars + 1).join('!');
}
```

注意我们定义了一个类型`Props`，它指定了我们组件要用到的属性。 `name`是必需的且为`string`类型，同时`enthusiasmLevel`是可选的且为`number`类型（你可以通过名字后面加`?`为指定可选参数）。

我们创建了一个无状态的函数式组件（Stateless Functional Components，SFC）`Hello`。 具体来讲，`Hello`是一个函数，接收一个`Props`对象并拆解它。 如果`Props`对象里没有设置`enthusiasmLevel`，默认值为`1`。

使用函数是React中定义组件的[两种方式](https://facebook.github.io/react/docs/components-and-props.html#functional-and-class-components)之一。 如果你喜欢的话，也*可以*通过类的方式定义：

```tsx
class Hello extends React.Component<Props, object> {
  render() {
    const { name, enthusiasmLevel = 1 } = this.props;

    if (enthusiasmLevel <= 0) {
      throw new Error('You could be a little more enthusiastic. :D');
    }

    return (
      <div className="hello">
        <div className="greeting">
          Hello {name + getExclamationMarks(enthusiasmLevel)}
        </div>
      </div>
    );
  }
}
```

当我们的[组件具有某些状态](https://facebook.github.io/react/docs/state-and-lifecycle.html)的时候，使用类的方式是很有用处的。 但在这个例子里我们不需要考虑状态 - 事实上，在`React.Component<Props, object>`我们把状态指定为了`object`，因此使用SFC更简洁。 当在创建可重用的通用UI组件的时候，在表现层使用组件局部状态比较适合。 针对我们应用的生命周期，我们会审视应用是如何通过Redux轻松地管理普通状态的。

现在我们已经写好了组件，让我们仔细看看`index.tsx`，把`<App />`替换成`<Hello ... />`。

首先我们在文件头部导入它：

```js
import Hello from './components/Hello.tsx';
```

然后修改`render`调用：

```tsx
ReactDOM.render(
  <Hello name="TypeScript" enthusiasmLevel={10} />,
  document.getElementById('root') as HTMLElement
);
```

这里还有一点要指出，就是最后一行`document.getElementById('root') as HTMLElement`。 这个语法叫做*类型断言*，有时也叫做*转换*。 当你比类型检查器更清楚一个表达式的类型的时候，你可以通过这种方式通知TypeScript。

这里，我们之所以这么做是因为`getElementById`的返回值类型是`HTMLElement | null`。 简单地说，`getElementById`返回`null`是当无法找对对应`id`元素的时候。 我们假设`getElementById`总是成功的，因此我们要使用`as`语法告诉TypeScript这点。

TypeScript还有一种感叹号（`!`）结尾的语法，它会从前面的表达式里移除`null`和`undefined`。 所以我们也*可以*写成`document.getElementById('root')!`，但在这里我们想写的更清楚些。

## 2.React中内置函数

> React中内置函数由很多，我们就挑几个常用的来学习一下。

### 2.1 React.FC< P >

> React.FC<>是函数式组件在TypeScript使用的一个泛型，FC就是FunctionComponent的缩写，事实上React.FC可以写成React.FunctionComponent。

```tsx
import React from 'react';
 
interface demo1PropsInterface {
    attr1: string,
    attr2 ?: string,
    attr3 ?: 'w' | 'ww' | 'ww'
};
 
// 函数组件，其也是类型别名
// type FC<P = {}> = FunctionComponent<P>;
// FunctionComponent<T>是一个接口，里面包含其函数定义和对应返回的属性
// interface FunctionComponent<P = {}> {
//      // 接口可以表示函数类型，通过给接口定义一个调用签名实现
//      (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
//      propTypes?: WeakValidationMap<P> | undefined;
//      contextTypes?: ValidationMap<any> | undefined;
//      defaultProps?: Partial<P> | undefined;
//      displayName?: string | undefined;
// }
const Demo1: React.FC<demo1PropsInterface> = ({
    attr1,
    attr2,
    attr3
}) => {
    return (
        <div>hello demo1 {attr1}</div>
    );
};
 
export default Demo1;
```

### 2.2 React.Component< P, S >

> React.Component< P, S > 是定义class组件的一个泛型，第一个参数是props、第二个参数是state。

```tsx
import React from "react";
 
// props的接口
interface demo2PropsInterface {
    props1: string
};
 
// state的接口
interface demo2StateInterface {
    state1: string
};
 
class Demo2 extends React.Component<demo2PropsInterface, demo2StateInterface> {
    constructor(props: demo2PropsInterface) {
        super(props);
        this.state = {
            state1: 'state1'
        }
    }
 
    render() {
        return (
            <div>{this.state.state1 + this.props.props1}</div>
        );
    }
}
 
export default Demo2;
```

### 2.3 React.Reducer<S, A>

> useState的替代方案，接收一个形如（state, action) => newState的reducer，并返回当前state以及其配套的dispatch方法。语法如下所示：`const [state, dispatch] = useReducer(reducer, initialArg, init);`

```tsx
import React, {useReducer, useContext} from "react";
 
interface stateInterface {
    count: number
};

interface actionInterface {
    type: string,
    data: {
        [propName: string]: any
    }
};
 
const initialState = {
    count: 0
};
 
// React.Reducer其实是类型别名，其实质上是type Reducer<S, A> = (prevState: S, action: A) => S;
// 因为reducer是一个函数，其接受两个泛型参数S和A，返回S类型
const reducer: React.Reducer<stateInterface, actionInterface> = (state, action) => {
    const {type, data} = action;
    switch (type) {
        case 'increment': {
            return {
                ...state,
                count: state.count + data.count
            };
        }
        case 'decrement': {
            return {
                ...state,
                count: state.count - data.count
            };
        }
        default: {
            return state;
        }
    }
}
```

### 2.4 `React.Context<T>`

1. React.createContext

> 其会创建一个Context对象，当React渲染一个订阅了这个Context对象的组件，这个组件会从组件树中离自身最近的那个匹配的Provider中读取到当前的context值。【注：只要当组件所处的树没有匹配到Provider时，其defaultValue参数参会生效】

```jsx

const MyContext = React.createContext(defaultValue);
 
const Demo = () => {
  return (
      // 注：每个Context对象都会返回一个Provider React组件，它允许消费组件订阅context的变化。
    <MyContext.Provider value={xxxxxx}>
      // ……
    </MyContext.Provider>
  );
```

2. useContext

> 接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。语法如下所示：`const value = useContext(MyContext);`

```jsx
import React, {useContext} from "react";
const MyContext = React.createContext('');
 
const Demo3Child: React.FC<{}> = () => {
    const context = useContext(MyContext);
    return (
        <div>
            {context}
        </div>
    );
}
 
const Demo3: React.FC<{}> = () => {
 
    return (
        <MyContext.Provider value={'222222'}>
            <MyContext.Provider value={'33333'}>
                <Demo3Child />
            </MyContext.Provider>
        </MyContext.Provider>
    );
};
```

3. 使用

```tsx
import React, {useReducer, useContext} from "react";
 
interface stateInterface {
    count: number
};

interface actionInterface {
    type: string,
    data: {
        [propName: string]: any
    }
};
 
const initialState = {
    count: 0
};
 
const reducer: React.Reducer<stateInterface, actionInterface> = (state, action) => {
    const {type, data} = action;
    switch (type) {
        case 'increment': {
            return {
                ...state,
                count: state.count + data.count
            };
        }
        case 'decrement': {
            return {
                ...state,
                count: state.count - data.count
            };
        }
        default: {
            return state;
        }
    }
}
 
// React.createContext返回的是一个对象，对象接口用接口表示
// 传入的为泛型参数，作为整个接口的一个参数
// interface Context<T> {
//      Provider: Provider<T>;
//      Consumer: Consumer<T>;
//      displayName?: string | undefined;
// }
const MyContext: React.Context<{
    state: stateInterface,
    dispatch ?: React.Dispatch<actionInterface>
}> = React.createContext({
    state: initialState
});
 
const Demo3Child: React.FC<{}> = () => {
    const {state, dispatch} = useContext(MyContext);
    const handleClick = () => {
        if (dispatch) {
            dispatch({
                type: 'increment',
                data: {
                    count: 10
                }
            })
        }
    };
    return (
        <div>
            {state.count}
            <button onClick={handleClick}>增加</button>
        </div>
    );
}
 
const Demo3: React.FC<{}> = () => {
    const [state, dispatch] = useReducer(reducer, initialState);
 
    return (
        <MyContext.Provider value={{state, dispatch}}>
            <Demo3Child />
        </MyContext.Provider>
    );
};
 
export default Demo3;
```

## 3.React中事件处理函数

> React中的事件是我们在编码中经常用的，例如onClick、onChange、onMouseMove等，那么应该如何用呢？

### 3.1 不带event参数

> 当对应的事件处理函数不带event参数时，这个时候用起来很简单，如下所示：

```jsx
const Test: React.FC<{}> = () => {
    const handleClick = () => {
        // 做一系列处理
    };
    return (
        <div>
            <button onClick={handleClick}>按钮</button>
        </div>
    );
};
```

### 3.2 带event参数

1. 带上event参数，报错

```jsx
const Test: React.FC<{}> = () => {
    // 报错了，注意不要这么写……
    const handleClick = event => {
        // 做一系列处理
        event.preventDefault();
    };
    return (
        <div>
            <button onClick={handleClick}>按钮</button>
        </div>
    );
};
```

2. 点击onClick参数，跳转到index.d.ts文件

```ts
// onClick是MouseEventHandler类型
onClick?: MouseEventHandler<T> | undefined;
 
// 那MouseEventHandler<T>又是啥？原来是个类型别名，泛型是Element类型
type MouseEventHandler<T = Element> = EventHandler<MouseEvent<T>>;
 
// 那么泛型Element又是什么呢？其是一个接口，通过继承该接口实现了很多其它接口
interface Element { }
interface HTMLElement extends Element { }
interface HTMLButtonElement extends HTMLElement { }
interface HTMLInputElement extends HTMLElement { }
// ……
```

至此，就知道该位置应该怎么实现了

```tsx
const Test: React.FC<{}> = () => {
    const handleClick: React.MouseEventHandler<HTMLButtonElement> = event => {
        // 做一系列处理
        event.preventDefault();
    };
    return (
        <div>
            <button onClick={handleClick}>按钮</button>
        </div>
    );
};
```

### 3.3 表单事件

```tsx
// 如果不考虑性能的话，可以使用内联处理，注解将自动正确生成
const el = (
    <button onClick=(e=>{
        //...
    })/>
)
// 如果需要在外部定义类型
handlerChange = (e: React.FormEvent<HTMLInputElement>): void => {
    //
}
// 如果在=号的左边进行注解
handlerChange: React.ChangeEventHandler<HTMLInputElement> = e => {
    //
}
// 如果在form里onSubmit的事件,React.SyntheticEvent,如果有自定义类型，可以使用类型断言
<form
  ref={formRef}
  onSubmit={(e: React.SyntheticEvent) => {
    e.preventDefault();
    const target = e.target as typeof e.target & {
      email: { value: string };
      password: { value: string };
    };
    const email = target.email.value; // typechecks!
    // etc...
  }}
>
  <div>
    <label>
      Email:
      <input type="email" name="email" />
    </label>
  </div>
  <div>
    <input type="submit" value="Log in" />
  </div>
</form>
```

### 3.4 事件类型列表

```css
AnimationEvent ： css动画事件
ChangeEvent：<input>， <select>和<textarea>元素的change事件
ClipboardEvent： 复制，粘贴，剪切事件
CompositionEvent：由于用户间接输入文本而发生的事件(例如，根据浏览器和PC的设置，如果你想在美国键盘上输入日文，可能会出现一个带有额外字符的弹出窗口)
DragEvent：在设备上拖放和交互的事件
FocusEvent: 元素获得焦点的事件
FormEvent: 当表单元素得失焦点/value改变/表单提交的事件
InvalidEvent: 当输入的有效性限制失败时触发(例如<input type="number" max="10">，有人将插入数字20)
KeyboardEvent: 键盘键入事件
MouseEvent： 鼠标移动事件
PointerEvent： 鼠标、笔/触控笔、触摸屏)的交互而发生的事件
TouchEvent： 用户与触摸设备交互而发生的事件
TransitionEvent： CSS Transition，浏览器支持度不高
UIEvent：鼠标、触摸和指针事件的基础事件。
WheelEvent: 在鼠标滚轮或类似的输入设备上滚动
SyntheticEvent：所有上述事件的基础事件。是否应该在不确定事件类型时使用
// 因为InputEvent在各个浏览器支持度不一样，所以可以使用KeyboardEvent代替
```

## 4.普通函数

1. 一个具体类型的输入输出函数

```ts
// 参数输入为number类型，通过类型判断直接知道输出也为number
function testFun1 (count: number) {
    return count * 2;
}
```

2. 一个不确定类型的输入、输出函数，但是输入、输出函数类型一致

```ts
// 用泛型
function testFun2<T> (arg: T): T {
    return arg;
}
```

3. async函数，返回的为Promise对象，可以使用then方法添加回调函数，Promise是一个泛型函数，T泛型变量用于确定then方法时接收的第一个回调函数的参数类型。

```tsx
// 用接口
interface PResponse<T> {
    result: T,
    status: string
};
 
// 除了用接口外，还可以用对象
// type PResponse<T> = {
//   	result: T,
//    status: string
// };
 
async function testFun3(): Promise<PResponse<number>> {
    return {
        status: 'success',
        result: 10
    }
}
```

## 5.React Prop 类型

- 如果你有配置 `Eslint` 等一些代码检查时，一般函数组件需要你定义返回的类型，或传入一些 `React` 相关的类型属性。
  这时了解一些 `React` 自定义暴露出的类型就很有必要了。例如常用的 `React.ReactNode`。

```tsx
export declare interface AppProps {
    children1: JSX.Element; // ❌ bad, 没有考虑数组类型
    children2: JSX.Element | JSX.Element[]; // ❌ 没考虑字符类型
    children3: React.ReactChildren; // ❌ 名字唬人，工具类型，慎用
    children4: React.ReactChild[]; // better, 但没考虑 null
    children: React.ReactNode; // ✅ best, 最佳接收所有 children 类型
    functionChildren: (name: string) => React.ReactNode; // ✅ 返回 React 节点

    style?: React.CSSProperties; // React style

    onChange?: React.FormEventHandler<HTMLInputElement>; // 表单事件! 泛型参数即 `event.target` 的类型
}
```

`defaultProps` 默认值问题。

```typescript
type GreetProps = { age: number } & typeof defaultProps;
const defaultProps = {
    age: 21,
};

const Greet = (props: GreetProps) => {
    // etc
};
Greet.defaultProps = defaultProps;
```

- 你可能不需要 defaultProps

```typescript
type GreetProps = { age?: number };

const Greet = ({ age = 21 }: GreetProps) => { 
    // etc 
};
```

## 6.Hooks

项目基本上都是使用函数式组件和 `React Hooks`。
接下来介绍常用的用 TS 编写 Hooks 的方法。

### 6.1 useState

- 给定初始化值情况下可以直接使用

```tsx
import { useState } from 'react';
// ...
const [val, toggle] = useState(false);
// val 被推断为 boolean 类型
// toggle 只能处理 boolean 类型
```

- 没有初始值（undefined）或初始 null

```tsx
type AppProps = { message: string };
const App = () => {
    const [data] = useState<AppProps | null>(null);
    // const [data] = useState<AppProps | undefined>();
    return <div>{data && data.message}</div>;
};
```

- 更优雅，链式判断

```typescript
// data && data.message
data?.message
```

### 6.2 useEffect

- 使用 `useEffect` 时传入的函数简写要小心，它接收一个无返回值函数或一个清除函数。

```tsx
function DelayedEffect(props: { timerMs: number }) {
    const { timerMs } = props;

    useEffect(
        () =>
            setTimeout(() => {
                /* do stuff */
            }, timerMs),
        [timerMs]
    );
    // ❌ bad example! setTimeout 会返回一个记录定时器的 number 类型
    // 因为简写，箭头函数的主体没有用大括号括起来。
    return null;
}
```

- 看看 `useEffect`接收的第一个参数的类型定义。

```tsx
// 1. 是一个函数
// 2. 无参数
// 3. 无返回值 或 返回一个清理函数，该函数类型无参数、无返回值 。
type EffectCallback = () => (void | (() => void | undefined));
```

- 了解了定义后，只需注意加层大括号。

```tsx
function DelayedEffect(props: { timerMs: number }) {
    const { timerMs } = props;

    useEffect(() => {
        const timer = setTimeout(() => {
            /* do stuff */
        }, timerMs);

        // 可选
        return () => clearTimeout(timer);
    }, [timerMs]);
    // ✅ 确保函数返回 void 或一个返回 void|undefined 的清理函数
    return null;
}
```

- 同理，async 处理异步请求，类似传入一个 `() => Promise<void>` 与 `EffectCallback` 不匹配。

```tsx
// ❌ bad
useEffect(async () => {
    const { data } = await ajax(params);
    // todo
}, [params]);
```

- 异步请求，处理方式：

```tsx
// ✅ better
useEffect(() => {
    (async () => {
        const { data } = await ajax(params);
        // todo
    })();
}, [params]);

// 或者 then 也是可以的
useEffect(() => {
    ajax(params).then(({ data }) => {
        // todo
    });
}, [params]);
```

### 6.3 useRef

`useRef` 一般用于两种场景

1. 引用 `DOM` 元素；
2. 不想作为其他 `hooks` 的依赖项，因为 `ref` 的值引用是不会变的，变的只是 `ref.current`。

- 使用 `useRef` ，可能会有两种方式。

```tsx
const ref1 = useRef<HTMLElement>(null!);
const ref2 = useRef<HTMLElement | null>(null);
```

- 非 null 断言 `null!`。断言之后的表达式非 null、undefined

```tsx
function MyComponent() {
    const ref1 = useRef<HTMLElement>(null!);
    useEffect(() => {
        doSomethingWith(ref1.current);
        // 跳过 TS null 检查。e.g. ref1 && ref1.current
    });
    return <div ref={ref1}> etc </div>;
}
```

- 不建议使用 `!`，存在隐患，Eslint 默认禁掉。

```tsx
function TextInputWithFocusButton() {
    // 初始化为 null, 但告知 TS 是希望 HTMLInputElement 类型
    // inputEl 只能用于 input elements
    const inputEl = React.useRef<HTMLInputElement>(null);
    const onButtonClick = () => {
        // TS 会检查 inputEl 类型，初始化 null 是没有 current 上是没有 focus 属性的
        // 你需要自定义判断! 
        if (inputEl && inputEl.current) {
            inputEl.current.focus();
        }
        // ✅ best
        inputEl.current?.focus();
    };
    return (
        <>
            <input ref={inputEl} type="text" />
            <button onClick={onButtonClick}>Focus the input</button>
        </>
    );
}
```

### 6.4 useReducer

使用 `useReducer` 时，多多利用 Discriminated Unions 来精确辨识、收窄确定的 `type` 的 `payload` 类型。
一般也需要定义 `reducer` 的返回类型，不然 TS 会自动推导。

- 又是一个联合类型收窄和避免拼写错误的精妙例子。

```tsx
const initialState = { count: 0 };

// ❌ bad，可能传入未定义的 type 类型，或码错单词，而且还需要针对不同的 type 来兼容 payload
// type ACTIONTYPE = { type: string; payload?: number | string };

// ✅ good
type ACTIONTYPE =
    | { type: 'increment'; payload: number }
    | { type: 'decrement'; payload: string }
    | { type: 'initial' };

function reducer(state: typeof initialState, action: ACTIONTYPE) {
    switch (action.type) {
        case 'increment':
            return { count: state.count + action.payload };
        case 'decrement':
            return { count: state.count - Number(action.payload) };
        case 'initial':
            return { count: initialState.count };
        default:
            throw new Error();
    }
}

function Counter() {
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
        <>
            Count: {state.count}            
		   <button onClick={() => dispatch({ type: 'decrement', payload: '5' })}>-</button>
            <button onClick={() => dispatch({ type: 'increment', payload: 5 })}>+</button>
        </>
    );
}
```

### 6.5 useContext

一般 `useContext` 和 `useReducer` 结合使用，来管理全局的数据流。

- 例子

```tsx
interface AppContextInterface {
    state: typeof initialState;
    dispatch: React.Dispatch<ACTIONTYPE>;
}

const AppCtx = React.createContext<AppContextInterface>({
    state: initialState,
    dispatch: (action) => action,
});
const App = (): React.ReactNode => {
    const [state, dispatch] = useReducer(reducer, initialState);

    return (
        <AppCtx.Provider value={{ state, dispatch }}>
            <Counter />
        </AppCtx.Provider>
    );
};

// 消费 context
function Counter() {
    const { state, dispatch } = React.useContext(AppCtx);
    return (
        <>
            Count: {state.count}            
		   <button onClick={() => dispatch({ type: 'decrement', payload: '5' })}>-</button>
            <button onClick={() => dispatch({ type: 'increment', payload: 5 })}>+</button>
        </>
    );
}
```

### 6.6 useImperativeHandle, forwardRef

推荐使用一个自定义的 `innerRef` 来代替原生的 `ref`，否则要用到 `forwardRef` 会搞的类型很复杂。

```tsx
type ListProps = {
  innerRef?: React.Ref<{ scrollToTop(): void }>
}

function List(props: ListProps) {
  useImperativeHandle(props.innerRef, () => ({
    scrollToTop() { }
  }))
  return null
}
```

结合刚刚 useRef 的知识，使用是这样的：

```tsx
function Use() {
  const listRef = useRef<{ scrollToTop(): void }>(null!)

  useEffect(() => {
    listRef.current.scrollToTop()
  }, [])

  return (
    <List innerRef={listRef} />
  )
}
```

很完美，是不是？

可以在线调试 [useImperativeHandle 的例子](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAKjgQwM5wEoFNkGN4BmUEIcA5FDvmQNwCwAUKJLHAN5wCuqWAyjMhhYANFx4BRAgSz5R3LNgJyeASXBYog4ADcsACWQA7ACYAbLHAC+cIiXKU8MWo0YwAnmAsAZYKhgAFYjB0AF52Rjg4YENDDUUAfgAuTCoYADpFAB4OVFxiU1MAFQhisAAKAEpk7QhgYysAPkZLFwYCTkN8YAhDOB8-MrAg1GT+gOGK8IZI+TVPTRgdfSMzLEHhtOjYqEVRSrgQhrgytgjIuFz8opKIcsmOFumrCoqzyhhOKF7DTgLm1vanUWPTgAFUePtTk9cD0-HBTL4YIoDmIFFgCNkLnkIAViqVKtVavVLA0yj8CgBCV4MM7ySTSfBlfaHKbneGIxRpXCfSiGdKXHHXfHUyKWUQAbQAutS3lgPl9jmdIpkxlEYnF0SE2Ai-IprAB6JpPamWIA)。

### 6.6 自定义 Hooks

`Hooks` 的美妙之处不只有减小代码行的功效，重点在于能够做到逻辑与 UI 分离。做纯粹的逻辑层复用。

- 例子：当你自定义 Hooks 时，返回的数组中的元素是确定的类型，而不是联合类型。可以使用 const-assertions 。

```typescript
export function useLoading() {
    const [isLoading, setState] = React.useState(false);
    const load = (aPromise: Promise<any>) => {
        setState(true);
        return aPromise.finally(() => setState(false));
    };
    return [isLoading, load] as const; // 推断出 [boolean, typeof load]，而不是联合类型 (boolean | typeof load)[]
}
```

- 也可以断言成 `tuple type` 元组类型。

```typescript
export function useLoading() {
    const [isLoading, setState] = React.useState(false);
    const load = (aPromise: Promise<any>) => {
        setState(true);
        return aPromise.finally(() => setState(false));
    };
    return [isLoading, load] as [
        boolean, 
        (aPromise: Promise<any>) => Promise<any>
    ];
}
```

- 如果对这种需求比较多，每个都写一遍比较麻烦，可以利用泛型定义一个辅助函数，且利用 TS 自动推断能力。

```typescript
function tuplify<T extends any[]>(...elements: T) {
    return elements;
}

function useArray() {
    const numberValue = useRef(3).current;
    const functionValue = useRef(() => {}).current;
    return [numberValue, functionValue]; // type is (number | (() => void))[]
}

function useTuple() {
    const numberValue = useRef(3).current;
    const functionValue = useRef(() => {
    }).current;
    return tuplify(numberValue, functionValue); // type is [number, () => void]
}
```


# 21 【styled-components的使用】

## 1.为什么要用这个

我们都知道，我们从最开始学css的时候，为了避免写的样式影响到另外的地方。所以我们这样来写的。

```css
#userConten .userBtn button{
  font-size: 18px;
}
```

首先给一个元素写了一个唯一id | class，然后在这个里面写对应的样式，就可以避免影响到其它地方的代码。但是，如果项目是多人协作，那就可能存在命名冲突了，所以我们想要一种技术来让整个项目起的类名都是唯一的id。避免样式冲突等问题。所以css in js 就来了。

> 简单来说CSS-in-JS就是将应用的CSS样式写在JavaScript文件里面，而不是独立为一些.css，.scss或者less之类的文件，这样你就可以在CSS中使用一些属于JS的诸如模块声明，变量定义，函数调用和条件判断等语言特性来提供灵活的可扩展的样式定义
>
>
> 使用这个技术写的库有很多，react中火的是styled-components，vue中css scope也是这个思想，每个组件都有它的scopeId，样式进行绑定，css modules也是同样的。react中css in js为什么火，框架本身就是html css js 写在一个组件混着写，虽然有些违背一些主流说法，但这就是它的特点，毕竟本身就就可以说html in js，再来一个css in js也很正常。

实现这个的库有很多，在react中最火的就是styled-components。

## 2.简介

**styled-components 是作者对于如何增强 React 组件中 CSS 表现这个问题的思考结果** 通过聚焦于单个用例,设法优化了开发者的体验和面向终端用户的输出.

Styled Components 的[官方网站](https://link.juejin.cn?target=https%3A%2F%2Fstyled-components.com%2Fdocs%2Fbasics)将其优点归结为：

- **Automatic critical CSS**：`styled-components` 持续跟踪页面上渲染的组件，并自动注入样式。结合使用**代码拆分**, 可以实现仅加载所需的最少代码。
- **解决了 class name 冲突**：`styled-components` 为样式生成唯一的 class name，开发者不必再担心 class name 重复、覆盖以及拼写的问题。（`CSS Modules` 通过哈希编码局部类名实现这一点）
- **CSS 更容易移除**：使用 `styled-components` 可以很轻松地知道代码中某个 class 在哪儿用到，因为每个样式都有其关联的组件。如果检测到某个组件未使用并且被删除，则其所有的样式也都被删除。
- **简单的动态样式**：可以很简单直观的实现根据组件的 `props` 或者全局主题适配样式，无需手动管理多个 classes。（这一点很赞）
- **无痛维护**：无需搜索不同的文件来查找影响组件的样式，无论代码多庞大，维护起来都是小菜一碟。
- **自动提供前缀**：按照当前标准写 CSS,其余的交给 `styled-components` 处理。

因为 `styled-components` 做的只是在 runtime 把 CSS 附加到对应的 HTML 元素或者组件上，它完美地支持所有 CSS。 媒体查询、伪选择器，甚至嵌套都可以工作。但是要注意，`styled-components` 是 `React` 下的 `CSS-in-JS` 的实践，因此下面的所有例子的技术栈都是 `React`。

## 3.安装

安装样式化组件只需要一个命令

```clike
npm install --save styled-components
yarn add styled-components
```

如果使用像 [yarn](https://yarnpkg.com/) 这样支持 “resolution” package.json 字段的包管理器，还要添加一个与主要版本范围对应的条目。这有助于避免因项目中安装的多个版本的样式化组件而引起的一整类问题。

在`package.json`:

```json
{
  "resolutions": {
    "styled-components": "^5"
  }
}
```

> **注意**
>
> 强烈推荐使用 styled-components 的 [babel 插件](https://www.styled-components.com/docs/tooling#babel-plugin) (当然这不是必须的).它提供了许多益处,比如更清晰的类名,SSR 兼容性,更小的包等等.
>
> `.babelrc`
>
> ```json
> {
>   "plugins": [
>     "babel-plugin-styled-components"
>   ]
> }
> ```

如果没有使用模块管理工具或者包管理工具,也可以使用官方托管在 unpkg CDN 上的构建版本.只需在HTML文件底部添加以下`<script>`标签:

```js
<script src="https://unpkg.com/styled-components/dist/styled-components.min.js"></script>
```

添加 styled-components 之后就可以访问全局的 `window.styled` 变量.

```jsx
const Component = window.styled.div`
  color: red;
`
```

> 注意
>
> 这用使用方式需要页面在 styled-components script 之前引入 [react CDN bundles](https://reactjs.org/docs/cdn-links.html)

VsCode 有一款插件 `vscode-styled-components` 能识别 `styled-components` ，并能自动进行 CSS 高亮、补全、纠正等。

![image-20221211221654403](https://i0.hdslb.com/bfs/album/04bec8ffd8a7532cf22bd5e1a0515d43a43410b6.png)

## 4.基本使用

**样式化组件**利用标记的模板文本来设置组件的样式。

它删除了组件和样式之间的映射。这意味着当你定义你的样式时，你实际上是在创建一个普通的 React 组件，它附加了你的样式。

以下的例子创建了两个简单的附加了样式的组件, 一个`Wrapper`和一个`Title`:

```jsx
import styled from 'styled-components'

/*
创建一个Title组件，
将render一个带有样式的h1标签
*/
const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;

/*
创建一个Wrapper组件，
将render一个带有样式的section标签
*/
const Wrapper = styled.section`
  padding: 4em;
  background: papayawhip;
`;

// 使用 Title and Wrapper 得到下面效果图
render(
  <Wrapper>
    <Title>
      Hello World!
    </Title>
  </Wrapper>
);
```

![image-20221214220428483](https://i0.hdslb.com/bfs/album/5a4ed6c8bb21b92717d3f9b411ea85fe1958aa4b.png)

值得注意的是`styled-components`创建的组件首字母必须以大写开头。

几乎所有基础的HTML标签styled都支持，比如`div`，`h1`，`span`…

`styled.xxx`后面的`.xxx`代表的是最终解析后的标签，如果是`styled.a`那么解析出来就是`a`标签，`styled.div`解析出来就是`div`标签。

> 注意
>
> styled-components 会为我们自动创建 CSS 前缀

## 5.基于props动态实现

我们可以将 props 以插值的方式传递给`styled component`,以调整组件样式.

下面这个 `Button` 组件持有一个可以改变`color`的`primary`属性. 将其设置为 ture 时,组件的`background-color`和`color`会交换.

```jsx
const Button = styled.button`
  background: ${props => props.primary ? "palevioletred" : "white"};
  color: ${props => props.primary ? "white" : "palevioletred"};
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

render(
  <>
    <Button>Normal</Button>
    <Button primary>Primary</Button>
  </>
);
```

![image-20221214221122731](https://i0.hdslb.com/bfs/album/2708314abbb6a73be7a3b64ccdc91ffd2f662ec4.png)

> 对于react开发者来说，这个还是比较香的。有人说用了这个之后，检查元素无法定位元素，其实它本身name是可以展示的，dev开发时候有一个插件配一下即可[styled-components: Tooling](https://styled-components.com/docs/tooling#control-the-components-displayname)

## 6.样式继承

可能我们希望某个经常使用的组件,在特定场景下可以稍微更改其样式.当然我们可以通过 props 传递插值的方式来实现,但是对于某个只需要重载一次的样式来说这样做的成本还是有点高.

创建一个继承其它组件样式的新组件,最简单的方式就是用构造函数`styled()`包裹被继承的组件.下面的示例就是通过继承上一节创建的按钮从而实现一些颜色相关样式的扩展:

```jsx
// 上一节创建的没有插值的 Button 组件
const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

// 一个继承 Button 的新组件, 重载了一部分样式
const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <TomatoButton>Tomato Button</TomatoButton>
  </div>
);
```

![image-20221211224440580](https://i0.hdslb.com/bfs/album/679163923fffbd8edaef117ae3466711e5090210.png)

可以看到,新的`TomatoButton`仍然和`Button`类似,我们只是添加了两条规则.

在某些情况下，您可能需要更改样式化组件渲染的标签或组件。这在构建导航栏时很常见，例如导航栏中同时存在链接和按钮,但是它们的样式应该相同.

在这种情况下,我们也有替代办法(escape hatch). 我们可以使用多态 ["as" polymorphic prop](https://www.styled-components.com/docs/api#as-polymorphic-prop) 动态的在不改变样式的情况下改变元素:

```jsx
const Button = styled.button`
  display: inline-block;
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <Button as="a" href="/">Link with Button styles</Button>
    <TomatoButton as="a" href="/">Link with Tomato Button styles</TomatoButton>
  </div>
);
```

这也完美适用于自定义组件:

```jsx
const Button = styled.button`
  display: inline-block;
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

const ReversedButton = props => <button {...props} children={props.children.split('').reverse()} />

render(
  <div>
    <Button>Normal Button</Button>
    <Button as={ReversedButton}>Custom Button with Normal Button styles</Button>
  </div>
);
```

> 比如:` styled("div")`,`styled.tagname`的方式就是 styled(tagname)`的别名.

## 7.条件渲染

`styled-components`最核心的一点，我个人认为也是这一点，让`styled-components`变得如此火热，我们直接先看下代码：

字符串前面那个`css`可加可不加，不加也是能够正常进行渲染的，但是还是推荐加，如果你不加的话在编辑器中就会失去提示的功能，编辑器会把它当作字符串而不是CSS样式。

```jsx
import { useState } from "react";
import styled, { css } from "styled-components";

const Box = styled.div`
  ${(props) =>
    props?.small
      ? css`
          width: 100px;
          height: 100px;
        `
      : css`
          width: 200px;
          height: 200px;
        `}

  background-color: red;
`;

export default function App() {
  const [small, setSmall] = useState(true);

  return (
    <div>
      <Box small={small} />
      <button onClick={() => setSmall(!small)}>切换</button>
    </div>
  );
}
```

![](https://pic2.zhimg.com/v2-580805bccf2c0c94c3c49f55ef5bb6b9_b.webp)

可以看到，使用`styled-components`编写组件样式的过程会变得异常的简单，如果你用的是CSS，那么你是无法通过React的Props进行更改CSS中的属性，你只能通过Props动态更改`dom`上绑定的类名，就如同下面的代码一样。

```jsx
import { useState } from "react";
import "./styles.css";

export default function App() {
  const [small, setSmall] = useState(true);

  return (
    <div>
      <div className={small ? "box-small" : "box"} />
      <button onClick={() => setSmall(!small)}>切换</button>
    </div>
  );
}
```

这样看起来`styled-components`没有什么特别的，甚至上面的写法还比较麻烦？其实`styled-components`的威力不止于此，我们看一下下面的例子：

```jsx
import { useState } from "react";
import styled, { css } from "styled-components";

const Box = styled.div`
  ${(props) => css`
    width: ${props?.size}px;
    height: ${props?.size}px;
  `}
  background-color: red;
`;

export default function App() {
  const [size, setSize] = useState(100);

  return (
    <div>
      <Box size={size} />
      <button onClick={() => setSize(size + 2)}>变大</button>
    </div>
  );
}
```

渲染如下：

![](https://pic3.zhimg.com/v2-164922bac2a36f19e4ed6dade1d2e4da_b.webp)

如果是通过CSS属性就非常难以实现这种效果，只有靠React官方提供的`style-in-js`方案，直接编写行内属性：

```jsx
import { useState } from "react";

export default function App() {
  const [size, setSize] = useState(100);

  return (
    <div>
      <div style={{ width: size, height: size, backgroundColor: "red" }} />
      <button onClick={() => setSize(size + 2)}>变大</button>
    </div>
  );
}
```

## 8.普通样式

如果使用过Vue的同学应该很清楚，在`.vue`文件中有个`style`标签，你只需要加上了`scoped`就可以进行样式隔离，而`styled-components`其实完全具有Vue的`style`标签的能力，你只需要在最外面包一层，然后就可以实现Vue中样式隔离的效果。

```jsx
import styled from 'styled-components'

const AppStyle = styled.div`
	.box {
		width: 100px;
		height: 100px;
		background-color: red;
	}
`
const Div = styled.div``

export default function App() {
	return (
		<AppStyle>
			<Div className="box"></Div>
		</AppStyle>
	)
}
```

![image-20221215201355737](https://i0.hdslb.com/bfs/album/49682ba2e36861f1646130abaab2e650cf517143.png)

甚至还可以配合上面的条件渲染进行使用，也非常的方便：

```jsx
import { useState } from "react";
import styled, { css } from "styled-components";

const AppStyle = styled.div`
  ${({ change }) =>
    change
      ? css`
          .box {
            width: 200px;
            height: 200px;
            background-color: blue;
          }
        `
      : css`
          .box {
            width: 100px;
            height: 100px;
            background-color: red;
          }
        `}
`;

export default function App() {
  const [change, setChange] = useState(false);

  return (
    <AppStyle change={change}>
      <div className="box" />
      <button
        onClick={() => {
          setChange(true);
        }}
      >
        更换
      </button>
    </AppStyle>
  );
}
```

渲染效果如下图所示：

![](https://pic4.zhimg.com/v2-d1c2d229d8d862560e42d0739d0d7283_b.webp)

## 9.attrs

为了避免仅为传递一些props来渲染组件或元素而使用不必要的wrapper, 可以使用 [`.attrs` constructor](https://www.styled-components.com/docs/api#attrs). 通过它可以添加额外的 props 或 attributes 到组件.

在一些HTML标签中是有一些属性的，比如`input`标签中，有`type`这个属性，我们就可以使用`attrs`给上一个默认值，还可以实现不传对应的属性则给一个默认值，如果传入对应的属性则使用传入的那个属性值。

```jsx
import styled from "styled-components";

const Input = styled.input.attrs((props) => ({
  // 直接指定一个值
  type: "text",

  // 给定一个默认值，可以传入Props进行修改
  size: props.size || "1em"
}))`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;

  margin: ${(props) => props.size};
  padding: ${(props) => props.size};
`;

export default function App() {
  return (
    <div>
      <Input placeholder="A small text input" />
      <br />
      <Input placeholder="A bigger text input" size="2em" />
    </div>
  );
}
```

渲染效果：

![image-20221215202119951](https://i0.hdslb.com/bfs/album/244b058f6f724781ae27003dd579283ec25525d5.png)

有继承的话，以继承后的组件中的属性为准

```jsx
const Input = styled.input.attrs((props) => ({
  type: "text",
  size: props.size || "1em"
}))`
  border: 2px solid palevioletred;
  margin: ${(props) => props.size};
  padding: ${(props) => props.size};
`;

// 有继承的话，以继承后的组件中的属性为准
const PasswordInput = styled(Input).attrs({
  type: "password"
})`
  border: 2px solid aqua;
`;

export default function App() {
  return (
    <div>
      <Input placeholder="A bigger text input" size="2em" />
      <br />
      <PasswordInput placeholder="A bigger password input" size="2em" />
    </div>
  );
}
```

最后渲染结果：

![image-20221215202344333](https://i0.hdslb.com/bfs/album/add05b9c0998959410d1fa526f77ddfe7813362e.png)

## 10.动画

虽然使用`@keyframes`的 CSS 动画不限于单个组件,但我们仍希望它们不是全局的(以避免冲突). 这就是为什么 styled-components 导出 `keyframes helper` 的原因: 它将生成一个可以在 APP 应用的唯一实例。

动画需要使用`keyframes`进行声明，如下所示：

```jsx
import styled, { keyframes } from "styled-components";

// 通过keyframes创建动画
const rotate = keyframes`
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
`;

// 创建动画的组件
const Rotate = styled.span`
  display: inline-block;
  animation: ${rotate} 2s linear infinite;
  padding: 2rem 1rem;
  font-size: 1.2rem;
`;

export default function App() {
  return (
    <div>
      <Rotate>&lt;    &gt;</Rotate>
    </div>
  );
}
```

渲染结果：

![](https://pic1.zhimg.com/v2-34c02bbceda71df53e6099ac928c0fd4_b.webp)

## 11.Coming from CSS

### 11.1 styled-components 如何在组件中工作?

### styled-components 如何在组件中工作?

如果你熟悉在组件中导入 CSS(例如 CSSModules),那么下面的写法你一定不陌生:

```jsx
import React from 'react'
import styles from './styles.css'

export default class Counter extends React.Component {
  state = { count: 0 }

  increment = () => this.setState({ count: this.state.count + 1 })
  decrement = () => this.setState({ count: this.state.count - 1 })

  render() {
    return (
      <div className={styles.counter}>
        <p className={styles.paragraph}>{this.state.count}</p>
        <button className={styles.button} onClick={this.increment}>
          +
        </button>
        <button className={styles.button} onClick={this.decrement}>
          -
        </button>
      </div>
    )
  }
}
```

由于 Styled Component 是 HTML 元素和作用在元素上的样式规则的组合, 我们可以这样编写`Counter`:

```jsx
import React from 'react'
import styled from 'styled-components'

const StyledCounter = styled.div`
  /* ... */
`
const Paragraph = styled.p`
  /* ... */
`
const Button = styled.button`
  /* ... */
`

export default class Counter extends React.Component {
  state = { count: 0 }

  increment = () => this.setState({ count: this.state.count + 1 })
  decrement = () => this.setState({ count: this.state.count - 1 })

  render() {
    return (
      <StyledCounter>
        <Paragraph>{this.state.count}</Paragraph>
        <Button onClick={this.increment}>+</Button>
        <Button onClick={this.decrement}>-</Button>
      </StyledCounter>
    )
  }
}
```

注意,我们在`StyledCounter`添加了"Styled"前缀,这样组件`Counter` 和`StyledCounter` 不会明明冲突,而且可以在 React Developer Tools 和 Web Inspector 中轻松识别.

### 11.2 使用伪元素、选择器、嵌套语法

由于 `styled-components` 采用 [stylis](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fthysultan%2Fstylis.js) 作为预处理器，因此提供了对伪元素、伪选择器以及嵌套写法的支持（跟 `Les` 很类似）。其中，`&` 指向组件本身：

```jsx
const Thing = styled.div`
  color: blue;
`
```

伪元素和伪类无需进一步细化,而是自动附加到了组件:

```jsx
const Thing = styled.button`
  color: blue;

  ::before {
    content: '🚀';
  }

  :hover {
    color: red;
  }
`

render(
  <Thing>Hello world!</Thing>
)
```

对于更复杂的选择器,可以使用与号(&)来指向主组件.以下是一些示例:

```JSX
const ScDiv = styled.div`
   color: blue;

  &:hover {
    color: red; // 被 hover 时的样式
  }

  & ~ & {
    background: tomato; // ScDiv 作为 ScDiv 的 sibling
  }

  & + & {
    background: lime; // 与 ScDiv 相邻的 ScDiv
  }

  &.something {
    background: orange; // 带有 class .something 的 ScDiv
  }

  .something-child & {
    border: 1px solid; // 不带有 & 时指向子元素，因此这里表示在带有 class .something-child 之内的 ScDiv
`;

render(
  <React.Fragment>
    <ScDiv>Hello world!</ScDiv>
    <ScDiv>How ya doing?</ScDiv>
    <ScDiv className="something">The sun is shining...</ScDiv>
    <ScDiv>Pretty nice day today.</ScDiv>
    <ScDiv>Don't you think?</ScDiv>
    <div className="something-else">
      <ScDiv>Splendid.</ScDiv>
    </div>
  </React.Fragment>
)
复制代码
```

渲染的结果如图所示：

![image-20221212205623181](https://i0.hdslb.com/bfs/album/268e0c946f791bbe6809ad8cd634137654671204.png)

如果只写选择器而不带&,则指向组件的子节点.

```jsx
const Thing = styled.div`
  color: blue;

  .something {
    border: 1px solid; // an element labeled ".something" inside <Thing>
    display: block;
  }
`

render(
  <Thing>
    <label htmlFor="foo-button" className="something">Mystery button</label>
    <button id="foo-button">What do I do?</button>
  </Thing>
)
```

最后,&可以用于增加组件的差异性;在处理混用 styled-components 和纯 CSS 导致的样式冲突时这将会非常有用:

```jsx
const Thing = styled.div`
  && {
    color: blue;
  }
`

const GlobalStyle = createGlobalStyle`
  div${Thing} {
    color: red;
  }
`

render(
  <React.Fragment>
    <GlobalStyle />
    <Thing>
      I'm blue, da ba dee da ba daa
    </Thing>
  </React.Fragment>
)
```

## 12.媒体查询

开发响应式 web app 时媒体查询是不可或缺的工具.

以下是一个非常简单的示例,展示了当屏宽小于700px时,组件如何改变背景色:

```jsx
const Content = styled.div`
  background: papayawhip;
  height: 3em;
  width: 3em;

  @media (max-width: 700px) {
    background: palevioletred;
  }
`;

render(
  <Content />
);
```

由于媒体查询很长,并且常常在应用中重复出现,因此有必要为其创建模板.

由于 JavaScript 的函数式特性,我们可以轻松的定义自己的标记模板字符串用于包装媒体查询中的样式.我们重写一下上个例子来试试:

```jsx
const sizes = {
  desktop: 992,
  tablet: 768,
  phone: 576,
}

// Iterate through the sizes and create a media template
const media = Object.keys(sizes).reduce((acc, label) => {
  acc[label] = (...args) => css`
    @media (max-width: ${sizes[label] / 16}em) {
      ${css(...args)}
    }
  `

  return acc
}, {})

const Content = styled.div`
  height: 3em;
  width: 3em;
  background: papayawhip;

  /* Now we have our methods on media and can use them instead of raw queries */
  ${media.desktop`background: dodgerblue;`}
  ${media.tablet`background: mediumseagreen;`}
  ${media.phone`background: palevioletred;`}
`;

render(
  <Content />
);
```

## 13.as prop

as - 转变组件类型，比如将一个div转变为button

```jsx
const Component = styled.div`
  color: red;
`;

render(
  <Component
    as="button"
    onClick={() => alert('It works!')}
  >
    Hello World!
  </Component>
)
```

```jsx
export default () => {
  return (
    // as(可以是组件名,也可以是普通标签名): 表示要渲染出来的标签或组件
    // 这个例子表示: 继承了 ScExtendedButton 样式的 a 标签
    <ScExtendedButton as="a" href="#">
      Extends Link with Button styles
    </ScExtendedButton>
  )
}
```

## 14.样式化任意组件

### 14.1 样式化组件

```jsx
const Link = ({ className, children }) => (
  // className 属性附加到 DOM 元素上
  <a className={className}>
    {children}
  </a>
)

const StyledLink = styled(Link)`
  color: red;
  font-weight: bold;
`

render(
  <div>
    <Link>Unstyled Link</Link>
    <StyledLink>Styled Link</StyledLink>
  </div>
)
```

### 14.2 样式化第三方组件

```jsx
import { Button } from 'antd'

const ScButton = styled(Button)`
  margin-top: 12px;
  color: green;
`

render(
  <div>
    <ScButton>Styled Fusion Button</ScButton>
  </div>
)
```

## 15.主题切换

### 15.1 基本使用

`styled-components` 通过导出 `<ThemeProvider>` 组件从而能支持主题切换。 `<ThemeProvider>`是基于 React 的 [Context API](https://link.juejin.cn?target=https%3A%2F%2Freact.docschina.org%2Fdocs%2Fcontext.html) 实现的，可以为其下面的所有 React 组件提供一个主题。在渲染树中，任何层次的所有样式组件都可以访问提供的主题。例如：

```JSX
import styled, {ThemeProvider} from "styled-components";

// 通过使用 props.theme 可以访问到 ThemeProvider 传递下来的对象
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

// 为 Button 指定默认的主题
Button.defaultProps = {
  theme: {
    main: "palevioletred"
  }
}

const theme = {
  main: "mediumseagreen"
};

render(
  <div>
    <Button>Normal</Button>
    // 采用了 ThemeProvider 提供的主题的 Button
    <ThemeProvider theme={theme}>
      <Button>Themed</Button>
    </ThemeProvider>
  </div>
);
```

![image-20221213202527317](https://i0.hdslb.com/bfs/album/a459ac651570269799e029b59de5e831ecf3406a.png)

### 15.2 函数主题

`ThemeProvider` 的 `theme`除了可以接受对象之外，还可以接受函数。函数的参数是父级的 `theme`对象。此外，还可以通过使用 theme prop 来处理 `ThemeProvider `未定义的情况（这跟上面的 `defaultProps`是一样的效果），或覆盖 `ThemeProvider`的 theme。例如：

```JSX
const ScButton = styled.button`
  color: ${props => props.theme.fg};
  border: 2px solid ${props => props.theme.fg};
  background: ${props => props.theme.bg};
`;

const theme = {
  fg: "palevioletred",
  bg: "white"
};

const invertTheme = ({ fg, bg }) => ({
  fg: bg,
  bg: fg
});

render(
  // ThemeProvider 未定义的情况
  <ScButton theme={{
    	fg: 'red',
      bg: 'white'
    }}>Default Theme</ScButton>
  <ThemeProvider theme={theme}>
    <div>
      <ScButton>Default Theme</ScButton>
    	// theme 接收的是一个函数，函数的参数是父级的 theme
      <ThemeProvider theme={invertTheme}>
        <ScButton>Inverted Theme</ScButton>
      </ThemeProvider>
      // 覆盖 ThemeProvider的 theme
      <ScButton theme={{
        fg: 'red',
      	bg: 'white'
        }}>Override Theme</ScButton>
    </div>
  </ThemeProvider>
);
```

![image-20221213202602311](https://i0.hdslb.com/bfs/album/96d14da04cb8d1af1b68f6c65cbc097a6c62a9da.png)

### 15.3 在 styled-components 外使用主题

如果需要在`styled-components`外使用主题,可以使用高阶组件`withTheme`:

```jsx
import { withTheme } from 'styled-components'

class MyComponent extends React.Component {
  render() {
    console.log('Current theme: ', this.props.theme)
    // ...
  }
}

export default withTheme(MyComponent)
```

**通过useContext React hook**

使用React Hooks时，还可以使用useContext访问样式化组件之外的当前主题。

```jsx
import { useContext } from 'react'
import { ThemeContext } from 'styled-components'

const MyComponent = () => {
  const themeContext = useContext(ThemeContext)

  console.log('Current theme: ', themeContext)
  // ...
}
```

**通过useTheme自定义挂钩**

使用React Hooks时，您还可以使用useTheme访问样式组件之外的当前主题。

```jsx
import { useTheme } from 'styled-components'

const MyComponent = () => {
  const theme = useTheme()

  console.log('Current theme: ', theme)
  // ...
}
```

### 15.4 theme prop

主题可以通过`theme prop`传递给组件.通过使用`theme prop`可以绕过或重写`ThemeProvider`所提供的主题.

```jsx
// Define our button
const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;

  /* Color the border and text with theme.main */
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

// Define what main theme will look like
const theme = {
  main: "mediumseagreen"
};

render(
  <div>
    <Button theme={{ main: "royalblue" }}>Ad hoc theme</Button>
    <ThemeProvider theme={theme}>
      <div>
        <Button>Themed</Button>
        <Button theme={{ main: "darkorange" }}>Overridden</Button>
      </div>
    </ThemeProvider>
  </div>
);
```

![image-20221213202829952](https://i0.hdslb.com/bfs/album/a19bfea901275dc20ac9ccbf8035751ec54daf77.png)

## 16.Refs

通过传递`ref prop`给 styled component 将获得:

- 底层 DOM 节点 (如果 styled 的对象是基本元素如 div)
- React 组件实例 (如果 styled 的对象是 React Component)

```jsx
const Input = styled.input`
  padding: 0.5em;
  margin: 0.5em;
  color: palevioletred;
  background: papayawhip;
  border: none;
  border-radius: 3px;
`;

class Form extends React.Component {
  constructor(props) {
    super(props);
    this.inputRef = React.createRef();
  }

  render() {
    return (
      <Input
        ref={this.inputRef}
        placeholder="Hover to focus!"
        onMouseEnter={() => {
          this.inputRef.current.focus()
        }}
      />
    );
  }
}

render(
  <Form />
);
```

![image-20221213203049891](https://i0.hdslb.com/bfs/album/61178534345ac369d7df02e8e57f00ed7af706a0.png)

> 注意
>
> v3 或更低的版本请使用 [innerRef prop](https://www.styled-components.com/docs/api#innerref-prop) instead.

## 17.样式对象

styled-components 支持将 CSS 写成 JavaScript 对象.对于已存在的样式对象,可以很轻松的将其迁移到 styled-components.

```jsx
// Static object
const Box = styled.div({
  background: 'palevioletred',
  height: '50px',
  width: '50px'
});

// Adapting based on props
const PropsBox = styled.div(props => ({
  background: props.background,
  height: '50px',
  width: '50px'
}));

render(
  <div>
    <Box />
    <PropsBox background="blue" />
  </div>
);
```

![image-20221213203538145](https://i0.hdslb.com/bfs/album/56e4f204838e39d4de8233f0d0ea7db56f671fd3.png)

## 18.CSS Prop实现内联样式

避免创建新的组件，直接应用样式，需要用到 styled-components 提供的 babel-plugin: [styled-components.com/docs/toolin…](https://link.juejin.cn/?target=https%3A%2F%2Fstyled-components.com%2Fdocs%2Ftooling%23babel-plugin)

```jsx
<div
  css={`
    background: papayawhip;
    color: ${props => props.theme.colors.text};
  `}
/>

<MyComponent css="padding: 0.5em 1em;"/>
```

参考：https://styled-components.com/docs/tooling#babel-plugin

## 19.mixin

```js
import styled, { css } from 'styled-components';
import { Button as FusionButton } from 'antd';

const mixinCommonCSS = css`
  margin-top: 12px;
  border: 1px solid grey;
  borde-radius: 4px;
`;

const ScButton = styled.button`
  ${mixinCommonCSS}
  color: yellow;
`;

const ScFusionButton = styled(FusionButton)`
  ${mixinCommonCSS}
  color: blue;
`;
```

## 20.性能问题

**Styled-Components 定义的组件一定要放在组件函数定义之外（对于 Class 类型的组件，不要放在** `render` **方法内 ）。因为在 react 组件的 render 方法中声明样式化的组件，会导致每次渲染都会创建一个新组建。 这意味着 React 将不得不在每个后续渲染中丢弃并重新计算 DOM 子树的那部分，而不是仅仅计算它们之间变化的差异，从而导致性能瓶颈和不可预测的行为。**

```jsx
// ❌ 绝对不要这样写
const Header = () => {
  const Title = styled.h1`
    font-size: 10px;
  `

  return (
    <div>
      <Title />
    </div>
  )
}

// ✅应该要这样写
const Title = styled.h1`
  font-size: 10px;
`

const Header = () => {
  return (
    <div>
      <Title />
    </div>
  )
}
```

此外，如果 `styled-components` 的目标是一个简单的 HTML 元素（例如 `styled.div`），那么 `styled-components` 将传递所有原生的 `HTML Attributes` 给 `DOM`。如果是自定义 `React` 组件（例如` styled(MyComponent`)），则 `styled-components` 会传递所有的 `props`。

## 21.配合TypeScript

React+TypeScript一直是神组合，React可以完美的搭配TypeScript。

但在TypeScript中使用得先安装`@types/styled-components`类型声明库：

`npm install @types/styled-components -D`

如在是要在TypeScript中，那么需要对`styled-components`组件的属性类型进行声明，不然会报错，虽然不会影响最终的编译结果：

![image-20221211230301186](https://i0.hdslb.com/bfs/album/f876c6a68c903f5b9423b4110b715eb50ec54e1c.png)

下面的组件类型就需要进行声明：

![image-20221211230316835](https://i0.hdslb.com/bfs/album/3f64cf75d4ac15cdb9ca6114bf6b01ea5851d975.png)

下面例子展示了一个样式化的 `Button` 接收 `primary` 属性，并根据该属性调整背景颜色 `background` 以及 `color`。

```JSX
import React, {
  ButtonHTMLAttributes
} from 'react';
import styled from 'styled-components';

interface IScButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
    primary?: boolean;
}

const ScWrapper = styled.div`
    margin-top: 12px;
`;

const ScButton = styled.button<IScButtonProps> `
    background: ${props => props.primary ? "blue" : "white"};
    color: ${props => props.primary ? "white" : "blue"};
    border: 2px solid palevioletred;
    border-radius: 3px;
    padding: 0.25em 1em;
`;

export default () => {
  return (
   <ScWrapper>
       <ScButton>Normal</ScButton>
       <ScButton primary>Primary</ScButton>
  </ScWrapper>
  );
};
```



# 22 【在react中使用Emotion】

## 1.CSS in JS 的优点

[CSS in JS](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMicheleBertoli%2Fcss-in-js) 已逐渐发展为 React 应用中写样式的一个主流的方案，著名组件库 [material-ui](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmui-org%2Fmaterial-ui) 也已经使用 CSS in JS 来实现。 CSS in JS 的实现方式有两种: 唯一CSS选择器和内联样式。因此

1. 不用关心繁琐的 Class 命名规则
2. 不用担心样式被覆盖
3. 便利的样式复用（样式都是 js 对象或字符串）
4. 减少冗余的 CSS 代码，极致的样式按需加载

[Emotion](https://link.juejin.cn?target=https%3A%2F%2Femotion.sh%2Fdocs%2Fintroduction) 是 CSS in JS 的众多实现方案中的其中一个，下面介绍一下它的使用。

*说明：以下的介绍都来自于[Emotion官方文档](https://link.juejin.cn?target=https%3A%2F%2Femotion.sh%2Fdocs%2Fintroduction)*

**安装**

```shell
 npm i @emotion/styled @emotion/react
```

**使用**

[Emotion](https://link.juejin.cn/?target=https%3A%2F%2Femotion.sh%2Fdocs%2Fintroduction) 有两种写 CSS 的方式：[css-prop](https://link.juejin.cn/?target=https%3A%2F%2Femotion.sh%2Fdocs%2Fcss-prop) 和 [Styled Components](https://link.juejin.cn/?target=https%3A%2F%2Femotion.sh%2Fdocs%2Fstyled)。

## 2.Css Prop

 添加预设或将杂注设置为注释后，`React.createElement`编译后的 jsx 代码将使用` emotion` 的函数而不是` .jsx`

### 2.1 Babel Preset

此方法**不适用于**[创建 React App](https://github.com/facebook/create-react-app) 或其他不允许自定义 Babel 配置的项目。 请改用 [JSX 注释方法](https://emotion.sh/docs/css-prop#jsx-pragma)。

`.babelrc`

```json
{
  "presets": ["@emotion/babel-preset-css-prop"]
}
```

> [完整的`@emotion/babel-preset-css-prop` 文档](https://emotion.sh/docs/@emotion/babel-preset-css-prop)

> If you are using the compatible React version (`>=16.14.0`) then you can opt into using [the new JSX runtimes](https://link.juejin.cn?target=https%3A%2F%2Freactjs.org%2Fblog%2F2020%2F09%2F22%2Fintroducing-the-new-jsx-transform.html) by using such configuration:
>
> 如果 React 版本 `>=16.14.0` , 可以使用如下的配置来使用新的 jsx 运行时。

```json
{
  "presets": [
    [
      "@babel/preset-react",
      { "runtime": "automatic", "importSource": "@emotion/react" }
    ]
  ],
  "plugins": ["@emotion/babel-plugin"]
}
```

### 2.2 JSX 注释

将` jsx 注释`设置在使用道具的源文件的顶部。 此选项最适合测试 prop 功能或在 babel 配置不可配置的项目（create-react-app、codesandbox 等）中。

```jsx
/** @jsx jsx */
import { jsx } from '@emotion/react'
```

`/** @jsx jsx */` 不生效的时候可以改为 `/** @jsxImportSource @emotion/react */` 来尝试。

### 2.3 tsconfig.json

这里指的是使用 [babel 编译 typescript](https://link.juejin.cn?target=https%3A%2F%2Fbabeljs.io%2Fdocs%2Fen%2Fbabel-preset-typescript) 时的配置

```JSON
{
  "compilerOptions": {
    ...
    // "jsx": "react",
    "jsxImportSource": "@emotion/react",
    ...
  }
}
```

### 2.4 Object Styles 和 String Styles

Emotion 支持 **js 对象**和 **js 字符串**两种形式的样式定义。

**Object Styles**

```jsx
/** @jsx jsx */
import { jsx } from '@emotion/react'

render(
  <div
    css={{
      backgroundColor: 'hotpink',
      '&:hover': {
        color: 'lightgreen'
      }
    }}
  >
    This has a hotpink background.
  </div>
)
```

![image-20221213124704208](https://i0.hdslb.com/bfs/album/4b3bee53c08921bd645abafad29715eb8f035fda.png)

> [Object Style Documentation](https://emotion.sh/docs/object-styles)

**String Styles**

要传递字符串样式，您必须使用 `@emotion/react`导出的`css` ，它可以用作[标记模板文字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)，如下所示。

```jsx
// this comment tells babel to convert jsx to calls to a function called jsx instead of React.createElement
/** @jsx jsx */
import { css, jsx } from '@emotion/react'

const color = 'darkgreen'

render(
  <div
    css={css`
      background-color: hotpink;
      &:hover {
        color: ${color};
      }
    `}
  >
    This has a hotpink background.
  </div>
)
```

![image-20221213124738854](https://i0.hdslb.com/bfs/album/48f48eb15a459ad1c0995a5aa214c0b2c0287f84.png)

无论是**Object Styles**还是**String Styles**，我们都可以直接在定义样式的时候读取上下文的 js 变量，这个可以让我们很方便地更改样式。

## 3.Styled Components

**Styled Components 基础用法** Styled Components 导出了一些带有 html 标签的内置组件。

### 3.1 写一个带样式的组件

`styled`和`css`非常相似，除了你用 html 标签或 React 组件调用它，然后用字符串样式的模板文字或对象样式的常规函数调用来调用它。

语法：styled.元素名`样式`

```jsx
import styled from '@emotion/styled'
const Button = styled.button`
  color: turquoise;
`
render(<Button>This my button component.</Button>)
```

![image-20221213205055196](https://i0.hdslb.com/bfs/album/60cc31773acca433bc2fa995080c711ff866476a.png)

### 3.2 通过参数控制样式

**Styled Components 的 Props** Styled Components 生成的组件也可以根据传入的 Props 来更改样式

```jsx
import styled from '@emotion/styled'

const Button = styled.button`
  color: ${props => (props.primary ? 'hotpink' : 'turquoise')};
`

const Container = styled.div(props => ({
  display: 'flex',
  flexDirection: props.column && 'column'
}))

render(
  <Container column>
    <Button>This is a regular button.</Button>
    <Button primary>This is a primary button.</Button>
  </Container>
)
```

![image-20221213211843757](https://i0.hdslb.com/bfs/album/2c23d647bf900fa4c8d593c8384fbf4971b3550b.png)

### 3.3 通过传className创建组件

语法：

```jsx
styled( ({className}) => (<p className={className}>text</
p>) )`样式`
```

 分析：相当于把样式通过className传递给了元素

```jsx
import styled from '@emotion/styled'
const Basic = ({ className }) => (
  <div className={className}>Some text</div>
)
const Fancy = styled(Basic)`
  color: hotpink;
`
render(<Fancy />)
```

![image-20221213212041581](https://i0.hdslb.com/bfs/album/66d270a59df509fa94d5988c28ebd530db563790.png)

### 3.4 创建与某个组件相同的样式

有时您想使用一个组件创建一些样式，然后再次将这些样式用于另一个组件，该方法可用于此目的。

语法：`样式组件.withComponent('元素')`

```jsx
import styled from '@emotion/styled'
const Section = styled.section`
  background: #333;
  color: #fff;
`
// Aside样式跟Section样式相同
const Aside = Section.withComponent('aside')
render(
  <div>
    <Section>This is a section</Section>
    <Aside>This is an aside</Aside>
  </div>
)
```

![image-20221213212523562](https://i0.hdslb.com/bfs/album/178bbf531f764220bba43a0b43fca2567d001b5a.png)

### 3.5 嵌套写法 

#### 3.5.1 ${子组件}

与[styled-components](https://www.styled-components.com/docs/faqs#can-i-refer-to-other-components)类似，当使用[@emotion/babel-plugin](https://emotion.sh/docs/@emotion/babel-plugin)时，`emotion`允许`emotion components`像常规CSS选择器一样被嵌套。

语法：父组件 = styled.元素`${子组件} {样式}`

```jsx
import styled from '@emotion/styled'

const Child = styled.div`
  color: red;
`

const Parent = styled.div`
  ${Child} {
    color: green;
  }
`

render(
  <div>
    <Parent>
      <Child>Green because I am inside a Parent</Child>
    </Parent>
    <Child>Red because I am not inside a Parent</Child>
  </div>
)
```

![image-20221213212915098](https://i0.hdslb.com/bfs/album/d05c31aa64be14c6898fb2f9cc5a1265a43f401c.png)

#### 3.5.2 对象(键值对)

组件选择器也可以与对象样式一起使用

语法:

```jsx
父组件 = styled.元素(
    {
      [子组件]: {样式}
    }
)
```

```jsx
import styled from '@emotion/styled'

const Child = styled.div({
  color: 'red'
})

const Parent = styled.div({
  [Child]: {
    color: 'green'
  }
})

render(
  <div>
    <Parent>
      <Child>green</Child>
    </Parent>
    <Child>red</Child>
  </div>
)
```

![image-20221213213342914](https://i0.hdslb.com/bfs/album/733e503cddf0831b7f9313fda0c9b93c6cef1bfb.png)

### 3.6 对象样式

```jsx
import styled from '@emotion/styled'
const H1 = styled.h1(
  {
    fontSize: 20
  },
  props => ({ color: props.color,  width:props.width })
)
render(<H1 color="lightgreen" width="200px">This is lightgreen.</H1>)
```

![image-20221213214851893](https://i0.hdslb.com/bfs/album/57fa8f06b69d2c9365e3cf44e45cf020c2a19fa7.png)

### 3.7 自定义 prop 转发

默认情况下，Emotion 会将所有 props（`theme`除外）传递给自定义组件，并且仅传递作为字符串标签的有效 html 属性的 prop。可以通过传递自定义函数来自定义此设置。您还可以使用`shouldForwardProp`来过滤掉无效的 html 属性。

```jsx
import isPropValid from '@emotion/is-prop-valid'
import styled from '@emotion/styled'

const H1 = styled('h1', {
  shouldForwardProp: prop => isPropValid(prop) && prop !== 'color'
})(props => ({
  color: props.color
}))

render(<H1 color="lightgreen">This is lightgreen.</H1>)
```

![image-20221213215311231](https://i0.hdslb.com/bfs/album/86abd793988549866422840d0d2548bda93026bb.png)

### 3.8 动态样式

您可以创建基于 props 的动态样式，并在样式中使用它们。

```jsx
import styled from '@emotion/styled'
import { css } from '@emotion/react'

const dynamicStyle = props =>
  css`
    color: ${props.color};
  `

const Container = styled.div`
  ${dynamicStyle};
`
render(<Container color="lightgreen">This is lightgreen.</Container>)
```

![image-20221213220140864](https://i0.hdslb.com/bfs/album/d9ef4d5fa40c63a7f54c12d3d35c334e91743c2f.png)

### 3.9 as prop 

要使用样式化组件中的样式但要更改呈现的元素，可以使用as prop。

```jsx
import styled from '@emotion/styled'

const Button = styled.button`
  color: hotpink;
`

render(
  <Button as="a" href="https://github.com/emotion-js/emotion">
    Emotion on GitHub
  </Button>
)
```

![image-20221213220134546](https://i0.hdslb.com/bfs/album/cb866b485db2dcfec11d62fac83e22acb1773889.png)

### 3.10 嵌套元素样式写法

我们可以使用以下方法嵌套选择器：`&`

```jsx
import styled from '@emotion/styled'
const Example = styled('span')`
  color: lightgreen;
  & > a {
    color: hotpink;
  }
`
render(
  <Example>
    This is <a>nested</a>.
  </Example>
)
```

![image-20221213220109366](https://i0.hdslb.com/bfs/album/ea0fe7df42f32ecf6776ffe9deccff5ce4f85f7f.png)

## 4.Composition

组合是`emotion`中最强大、最有用的模式之一。您可以通过在另一个样式块中插入从`css`返回的值来组合样式。

### 4.1 样式复用

在 Emotion 中，我们可以把通用样式用变量声明，然后在不同的组件中共享。

```jsx
import { css } from '@emotion/react'

const base = css`
  color: hotpink;
`

render(
  <div
    css={css`
      ${base};
      background-color: #eee;
    `}
  >
    This is hotpink.
  </div>
)
```

上面的 `base` 样式在 render 时被使用。如果我们有其它的组件用到 base 样式，我们也可以导入 base 这个变量来使用。

### 4.2 样式优先级

```jsx
import { css } from '@emotion/react'

const danger = css`
  color: red;
`

const base = css`
  background-color: darkgreen;
  color: turquoise;
`

render(
  <div>
    <div css={base}>This will be turquoise</div>
    <div css={[danger, base]}>
      This will be also be turquoise since the base styles overwrite the danger
      styles.
    </div>
    <div css={[base, danger]}>This will be red</div>
  </div>
)
```

![image-20221213221609976](https://i0.hdslb.com/bfs/album/99cc1b67631d8bbb2ae38d6f50b8fe2f46571037.png)

写样式的时候难免会需要覆盖样式的情况，这时候我们可以像上面一样调整 `base` 和 `danger` 的先后顺序来覆盖（后面的样式优先级较高）。

## 5.Object Styles

带对象的写作风格是一种直接构建在`emotion`核心的强大模式。您可以使用`camelCase`来编写css属性，而不是像普通css那样使用`kebab-case`大小写，例如背景色将是backgroundColor。对象样式对于css属性特别有用，因为您不需要像字符串样式那样的css调用，但是对象样式也可以与样式一起使用。

### 5.1 使用 css props

```jsx
render(
  <div
    css={{
      color: 'darkorchid',
      backgroundColor: 'lightgray'
    }}
  >
    This is darkorchid.
  </div>
)
```

![image-20221213223856868](https://i0.hdslb.com/bfs/album/14181788a3309a5df5fbf2a5ede49f23b6e071c2.png)

### 5.2 使用`styled`

```jsx
import styled from '@emotion/styled'

const Button = styled.button(
  {
    color: 'darkorchid'
  },
  props => ({
    fontSize: props.fontSize
  })
)

render(<Button fontSize={16}>This is a darkorchid button.</Button>)
```

![image-20221213224008883](https://i0.hdslb.com/bfs/album/642dfb1e199286104af27b8bad8990ecc212e902.png)

### 5.3 子选择器

```jsx
render(
  <div
    css={{
      color: 'darkorchid',
      '& .name': {
        color: 'orange'
      }
    }}
  >
    This is darkorchid.
    <div className="name">This is orange</div>
  </div>
)
```

![image-20221213224107609](https://i0.hdslb.com/bfs/album/b349553deeaa0c1bb843f422d22c652c004721d6.png)

### 5.4 媒体查询

```jsx
render(
  <div
    css={{
      color: 'darkorchid',
      '@media(min-width: 420px)': {
        color: 'orange'
      }
    }}
  >
    This is orange on a big screen and darkorchid on a small screen.
  </div>
)
```

![image-20221213224217334](https://i0.hdslb.com/bfs/album/22fb6f288af1223c71747a4e36152ad6d64a8d23.png)

### 5.5 Numbers

```jsx
render(
  <div
    css={{
      padding: 8,
      zIndex: 200
    }}
  >
    This has 8px of padding and a z-index of 200.
  </div>
)
```

![image-20221213224256993](https://i0.hdslb.com/bfs/album/8babd6621e973346e38780fd52d46fe63cdc4b7d.png)

### 5.6 Arrays

嵌套数组被展平

```jsx
render(
  <div
    css={[
      { color: 'darkorchid' },
      { backgroundColor: 'hotpink' },
      { padding: 8 }
    ]}
  >
    This is darkorchid with a hotpink background and 8px of padding.
  </div>
)
```

![image-20221213224346539](https://i0.hdslb.com/bfs/album/92d2578e8eac63bbc96f4417725d0f6d970cd057.png)

### 5.7 用`css`

您也可以将`css`与对象样式一起使用。

```jsx
import { css } from '@emotion/react'

const hotpink = css({
  color: 'hotpink'
})

render(
  <div>
    <p css={hotpink}>This is hotpink</p>
  </div>
)
```

![image-20221213224458835](https://i0.hdslb.com/bfs/album/a5393da38448be4d92c7b11729fed649538c5ae2.png)

### 5.8 Composition - 样式复用

[Learn more composition in Emotion](https://emotion.sh/docs/composition).

```jsx
import { css } from '@emotion/react'

const hotpink = css({
  color: 'hotpink'
})

const hotpinkHoverOrFocus = css({
  '&:hover,&:focus': hotpink
})

const hotpinkWithBlackBackground = css(
  {
    backgroundColor: 'black',
    color: 'green'
  },
  hotpink
)

render(
  <div>
    <p css={hotpink}>This is hotpink</p>
    <button css={hotpinkHoverOrFocus}>This is hotpink on hover or focus</button>
    <p css={hotpinkWithBlackBackground}>
      This has a black background and is hotpink. Try moving where hotpink is in
      the css call and see if the color changes.
    </p>
  </div>
)
```

![image-20221213224550509](https://i0.hdslb.com/bfs/album/aba3db5b7cf387f6af88b34f3948d2a23a12bd19.png)

## 6.Nested Selectors

有时，将选择器嵌套到当前类或 React 组件中的元素很有用。下面显示了带有元素选择器的示例。

```jsx
import { css } from '@emotion/react'

const paragraph = css`
  color: turquoise;

  a {
    border-bottom: 1px solid currentColor;
    cursor: pointer;
  }
`
render(
  <p css={paragraph}>
    Some text. <a>A link with a bottom border.</a>
  </p>
)
```

![image-20221213224756200](https://i0.hdslb.com/bfs/album/2197b572f97c83f1a4a221a56f679f9d487405eb.png)

当组件是子组件时，使用 `&` 来选择自己并设置样式

```jsx
import { css } from '@emotion/react'

const paragraph = css`
  color: turquoise;

  header & {
    color: green;
  }
`
render(
  <div>
    <header>
      <p css={paragraph}>This is green since it's inside a header</p>
    </header>
    <p css={paragraph}>This is turquoise since it's not inside a header.</p>
  </div>
)
```

![image-20221213224813608](https://i0.hdslb.com/bfs/album/f2a168ff43f2f5be762d1b03832c5474cf620e63.png)

## 7.Media Queries

在`emotion`中使用媒体查询就像在常规 css 中使用媒体查询一样，只是您不必在块内指定选择器，您可以将 css 直接放在 css 块中。

```jsx
import { css } from '@emotion/react'

render(
  <p
    css={css`
      font-size: 30px;
      @media (min-width: 420px) {
        font-size: 50px;
      }
    `}
  >
    Some text!
  </p>
)
```

![image-20221213225036540](https://i0.hdslb.com/bfs/album/263d631dd7b4764a5b7b996461c6b92945aad50a.png)

## 8.Global Styles

有时您可能希望插入全局 css，例如resets 或 font faces。您可以使用该`Global`组件来执行此操作。它接受一个 `styles`prop，该 prop 接受与`css` prop 相同的值，除了全局插入样式。当样式更改或全局组件卸载时，也会删除全局样式。

```jsx
import { Global, css } from '@emotion/react'

render(
  <div>
    <Global
      styles={css`
        .some-class {
          color: hotpink !important;
        }
      `}
    />
    <Global
      styles={{
        '.some-class': {
          fontSize: 50,
          textAlign: 'center'
        }
      }}
    />
    <div className="some-class">This is hotpink now!</div>
  </div>
)
```

![image-20221213225321584](https://i0.hdslb.com/bfs/album/7988629dc67da128788bcc054e19a1199f8c6712.png)

## 9.Keyframes

您可以使用`@emotive/react`中的`keyframes`来定义动画。`keyframe`接受css关键帧定义，并返回一个可以在样式中使用的对象。您可以像`css`一样使用字符串或对象。

```jsx
import { css, keyframes } from '@emotion/react'

const bounce = keyframes`
  from, 20%, 53%, 80%, to {
    transform: translate3d(0,0,0);
  }

  40%, 43% {
    transform: translate3d(0, -30px, 0);
  }

  70% {
    transform: translate3d(0, -15px, 0);
  }

  90% {
    transform: translate3d(0,-4px,0);
  }
`

render(
  <div
    css={css`
      animation: ${bounce} 1s ease infinite;
    `}
  >
    some bouncing text!
  </div>
)
```

![image-20221213225454209](https://i0.hdslb.com/bfs/album/d28d6aae15d22a847285ba8bb2a7a82271e416ab.png)

## 10.Attaching Props - 附加额外的属性

一些 css-in-js 库提供了将 props 附加到组件的 API，而不是让我们自己的 API 来做到这一点，我们建议创建一个常规的 react 组件，使用 css prop 并像附加任何其他 React 组件一样附加 props。

请注意，如果 css 是通过 props 传递下来的，它将优先于组件中的 css。

```jsx
import { css } from '@emotion/react'

const pinkInput = css`
  background-color: pink;
`
const RedPasswordInput = props => (
  <input
    type="password"
    css={css`
      background-color: red;
      display: block;
    `}
    {...props}
  />
)

render(
  <div>
    <RedPasswordInput placeholder="red" />
    <RedPasswordInput placeholder="pink" css={pinkInput} />
  </div>
)
```

![image-20221213230119759](https://i0.hdslb.com/bfs/album/844df046c45d332f079a5a1b949d80a4450359aa.png)

## 11.Theming

主题包含在`@emotion/react`中。
将`ThemeProvider`添加到应用程序的顶层，并在样式组件中使用`props.theme`访问主题，或者提供一个接受主题作为css属性的函数。

### 11.1 css prop

```jsx
import { ThemeProvider } from '@emotion/react'

const theme = {
  colors: {
    primary: 'hotpink'
  }
}

render(
  <ThemeProvider theme={theme}>
    <div css={theme => ({ color: theme.colors.primary })}>some other text</div>
  </ThemeProvider>
)
```

![image-20221213230310767](https://i0.hdslb.com/bfs/album/b7f0c6de7e1976e5de1881f5fb71510b1aa064d2.png)

### 11.2 styled

```jsx
import { ThemeProvider } from '@emotion/react'
import styled from '@emotion/styled'

const theme = {
  colors: {
    primary: 'hotpink'
  }
}

const SomeText = styled.div`
  color: ${props => props.theme.colors.primary};
`

render(
  <ThemeProvider theme={theme}>
    <SomeText>some text</SomeText>
  </ThemeProvider>
)
```

![image-20221213230337193](https://i0.hdslb.com/bfs/album/c1f82f03c9823ade2b249fdc155e655ac449b237.png)

### 11.3 useTheme hook

```jsx
import { ThemeProvider, useTheme } from '@emotion/react'

const theme = {
  colors: {
    primary: 'hotpink'
  }
}

function SomeText(props) {
  const theme = useTheme()
  return <div css={{ color: theme.colors.primary }} {...props} />
}

render(
  <ThemeProvider theme={theme}>
    <SomeText>some text</SomeText>
  </ThemeProvider>
)
```

![image-20221213230505784](https://i0.hdslb.com/bfs/album/e62c366c81154fc1fc57c9deaa07ccb756351394.png)

## 12.TypeScript

Emotion包括`@emotion/react` and `@emotion/styled`的TypeScript定义。这些定义通过对象语法、HTML/SVG标记名和属性类型推断css属性的类型。

> **@emotion/react**
>
> 这种方法使用好像比较麻烦，可以去看看官网
>
> [Emotion – Package Summaries](https://emotion.sh/docs/typescript)

`@emotion/styled`与TypeScript配合使用，无需任何额外配置。

### 12.1 HTML/SVG elements

```tsx
import styled from '@emotion/styled'

const Link = styled('a')`
  color: red;
`

const Icon = styled('svg')`
  stroke: green;
`

const App = () => <Link href="#">Click me</Link>
```

------

```tsx
import styled from '@emotion/styled';

const NotALink = styled('div')`
  color: red;
`;

const App = () => (
  <NotALink href="#">Click me</NotALink>
            ^^^^^^^^ Property 'href' does not exist [...]
);
```

`withComponent`

```tsx
import styled from '@emotion/styled'

const NotALink = styled('div')`
  color: red;
`

const Link = NotALink.withComponent('a')

const App = () => <Link href="#">Click me</Link>

// No errors!
```

### 12.2 定义 props 类型

您可以定义`styled components` props 的类型。

```tsx
import styled from '@emotion/styled'

type ImageProps = {
  src: string
  width: number
}

// Using a css block
const Image0 = styled.div<ImageProps>`
  width: ${props => props.width};
  background: url(${props => props.src}) center center;
  background-size: contain;
`
const Image0 = styled('div')<ImageProps>`
  width: ${props => props.width};
  background: url(${props => props.src}) center center;
  background-size: contain;
`

// Or with object styles
const Image1 = styled('div')<ImageProps>(
  {
    backgroundSize: 'contain'
  },
  props => ({
    width: props.width,
    background: `url(${props.src}) center center`
  })
)
```

### 12.3 React Components

`Emotion `还可以设置React组件的样式，并根据预期推断组件 props。

```tsx
import React, { FC } from 'react'
import styled from '@emotion/styled'

interface ComponentProps {
  className?: string
  label: string
}

const Component: FC<ComponentProps> = ({ label, className }) => (
  <div className={className}>{label}</div>
)

const StyledComponent0 = styled(Component)`
  color: ${props => (props.label === 'Important' ? 'red' : 'green')};
`

const StyledComponent1 = styled(Component)({
  color: 'red'
})

const App = () => (
  <div>
    <StyledComponent0 label="Important" />
    <StyledComponent1 label="Yea! No need to re-type this label prop." />
  </div>
)
```

