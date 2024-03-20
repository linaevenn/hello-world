# 23 【UmiJS入门】

## 1.Umi 介绍

![Umi](https://img.alicdn.com/imgextra/i3/O1CN01eBiy611b67KLFOxi3_!!6000000003415-2-tps-200-200.png)

### 1.1 Umi 是什么？

Umi，中文发音为「乌米」，是可扩展的企业级前端应用框架。Umi 以路由为基础的，同时支持配置式路由和约定式路由，保证路由的功能完备，并以此进行功能扩展。然后配以生命周期完善的插件体系，覆盖从源码到构建产物的每个生命周期，支持各种功能扩展和业务需求。

Umi 有很多非常有意思的特性，比如。

1、**企业级**，在安全性、稳定性、最佳实践、约束能力方面会考虑更多
2、**插件化**，啥都能改，Umi 本身也是由插件构成
3、**MFSU**，比 Vite 还快的 Webpack 打包方案
4、基于 React Router 6 的完备路由
5、默认最快的请求
6、SSR & SSG
7、稳定白盒性能好的 ESLint 和 Jest
8、React 18 的框架级接入
9、Monorepo 最佳实践
...

### 1.2 什么时候不用 Umi？

如果你的项目，

1、需要支持 IE 8 或更低版本的浏览器
2、需要支持 React 16.8.0 以下的 React
3、需要跑在 Node 14 以下的环境中
4、有很强的 webpack 自定义需求和主观意愿
5、需要选择不同的路由方案
...

Umi 可能不适合你。

### 1.3 为什么不是？

1. `create-react-app`

   create-react-app 是脚手架，和 Umi、next.js、remix、ice、modern.js 等元框架不是同一类型。脚手架可以让我们快速启动项目，对于单一的项目够用，但对于团队而言却不够。因为使用脚手架像泼出去的水，一旦启动，无法迭代。同时脚手架所能做的封装和抽象都非常有限。

2. `next.js`

   如果要做 SSR，next.js 是非常好的选择（当然，Umi 也支持 SSR）；而如果只做 CSR，Umi 会是更好的选择。相比之下，Umi 的扩展性会更好；并且 Umi 做了很多更贴地气的功能，比如配置式路由、补丁方案、antd 的接入、微前端、国际化、权限等；同时 Umi 会更稳定，因为他锁了能锁的全部依赖，定期主动更新，某一个子版本的 Umi，不会因为重装依赖之后而跑不起来。

3. `remix`

   Remix 是我非常喜欢的框架，Umi 4 从中~~抄~~（学）了不少东西。但 Remix 是 Server 框架，其内置的 loader 和 action 都是跑在 server 端的，所以会对部署环境会有一定要求。Umi 将 loader、action 以及 remix 的请求机制同时运用到 client 和 server 侧，不仅 server 请求快，纯 CSR 的项目请求也可达到理论的最快值。同时 Remix 基于 esbuild 做打包，可能不适用于对兼容性有要求或者依赖尺寸特别大的项目。

### 1.4 插件和插件集

Umi 通过提供插件和插件集的机制来满足不同场景和业务的需求。插件是为了扩展一个功能，而插件集是为了扩展一类业务。比如要支持 vue，我们可以有 `@umijs/preset-vue`，包含 vue 相关的构建和运行时；比如要支持 h5 的应用类型，可以有 `@umijs/preset-h5`，把 h5 相关的功能集合到一起。

如果要类比，插件集和 babel 的 preset，以及 eslint 的 config 都类似。

### 1.5 import all from umi

很多人可能都第一次听到。import all from umi 意思是所有 import 都来自 `umi`。比如 dva 不是 `import { connect } from 'dva'`，而是 `import { connect } from 'umi'`，从 umi 中导出。导出的方法不仅来自 umi 自身，还来自 umi 插件。

这是两年前 Umi 3 加的功能，最近发现 Remix、prisma、vitekit 等框架和工具都有类似实现。

```ts
// 大量插件为 umi 提供额外导出内容
import { connect, useModel, useIntl, useRequest, MicroApp, ... } from 'umi';
```

这带来的好处是。通过 Umi 将大量依赖管理起来，用户无需手动安装；同时开发者在代码中也会少很多 import 语句。

## 2.快速上手

### 2.1 环境准备

首先得有 node，并确保 node 版本是 14 或以上。（推荐用 [nvm](https://github.com/nvm-sh/nvm) 来管理 node 版本，windows 下推荐用 [nvm-windows](https://github.com/coreybutler/nvm-windows)）

然后需要包管理工具。node 默认包含 npm，但也可以选择其他方案，

- [pnpm](https://pnpm.io/installation), umi 团队推荐
- [Yarn](https://yarnpkg.com/getting-started/install)

### 2.2 创建项目

先找个地方建个空目录。

```bash
mkdir myapp && cd myapp
```

通过官方工具创建项目，

```bash
pnpm dlx create-umi@latest
✔ Install the following package: create-umi? (Y/n) · true
✔ Pick Npm Client › pnpm
✔ Pick Npm Registry › taobao
Write: .gitignore
Write: .npmrc
Write: .umirc.ts
Copy:  layouts/index.tsx
Write: package.json
Copy:  pages/index.tsx
Copy:  pages/users.tsx
Copy:  pages/users/foo.tsx
> @ postinstall /private/tmp/sorrycc-vylwuW
> umi setup
info  - generate files
```

也可以使用yarn和npm

```bash
$ npx create-umi@latest
$ yarn create umi
```

国内建议选 **pnpm + taobao 源**，速度提升明显。这一步会自动安装依赖，同时安装成功后会自动执行 `umi setup` 做一些文件预处理等工作。

选择后会自动生成一个最基本的 Umi 项目，并根据选中的客户端和镜像源安装依赖：

```bash
.
├── package.json
├── pnpm-lock.yaml
├── src
│   ├── assets
│   │   └── yay.jpg
│   ├── layouts
│   │   ├── index.less
│   │   └── index.tsx
│   └── pages
│       ├── docs.tsx
│       └── index.tsx
├── tsconfig.json
└── typings.d.ts
```

这样就一键完成 Umi 项目的初始化了。

### 2.3 参数选项

使用 `create-umi` 创建项目时，可用的参数如下：

| option         | description                |
| -------------- | -------------------------- |
| `--no-git`     | 创建项目，但不初始化 Git   |
| `--no-install` | 创建项目，但不自动安装依赖 |

## 3.运行时配置

运行时配置和配置的区别是他跑在浏览器端，基于此，我们可以在这里写函数、tsx、import 浏览器端依赖等等，注意不要引入 node 依赖。

### 3.1 配置方式

约定 `src/app.tsx` 为运行时配置。

### 3.2 配置

Umi 在 `.umirc.ts` 或 `config/config.ts` 中配置项目和插件，支持 es6。一份常见的配置如下，

```bash
export default {
  base: '/docs/',
  publicPath: '/public/',
  hash: true,
  history: {
    type: 'hash',
  },
}
```

### 3.3 配置文件

如果项目的配置不复杂，推荐在 `.umirc.ts` 中写配置； 如果项目的配置比较复杂，可以将配置写在 `config/config.ts` 中，并把配置的一部分拆分出去，比如路由配置可以拆分成单独的 `routes.ts`：

```typescript
// config/routes.ts

export default [
    { exact: true, path: '/', component: 'index' },
];
```

```ts
// config/config.ts

import { defineConfig } from 'umi';
import routes from './routes';

export default defineConfig({
  routes: routes,
});
```

推荐两种配置方式二选一，`.umirc.ts` 优先级更高。

### 3.4 TypeScript 提示

如果你想在写配置时也有提示，可以通过 umi 的 `defineConfig` 方法定义配置，

```js
import { defineApp } from 'umi';
export default defineApp({
  layout: () => {
    return {
      title: 'umi',
    };
  },
});
 
// or
import { RuntimeConfig } from 'umi';
export const layout: RuntimeConfig['layout'] = () => {
  return {
    title: 'umi',
  };
};
```

### 3.5 本地临时配置

可以新建 `.umirc.local.ts`，这份配置会和 `.umirc.ts` 做 deep merge 后形成最终配置。

> 注：`.umirc.local.ts` 仅在 `umi dev` 时有效。`umi build` 时不会被加载。

比如，

```js
// .umirc.ts 或者 config/config.ts
export default { a: 1, b: 2 };

// .umirc.local.ts 或者 config/config.local.ts
export default { c: 'local' };
```

拿到的配置是：

```js
{
  a: 1,
  b: 2,
  c: 'local',
}
```

注意：

- `config/config.ts` 对应的是 `config/config.local.ts`
- `.local.ts` 是本地验证使用的临时配置，请将其添加到 `.gitignore`，**务必不要提交到 git 仓库中**
- `.local.ts` 配置的优先级最高，比 `UMI_ENV` 指定的配置更高

## 4.目录结构

这里罗列了 Umi 项目中约定(或推荐)的目录结构，在项目开发中，请遵照这个目录结构组织代码。

```bash
.
├── config
│   └── config.ts
├── dist
├── mock
│   └── app.ts｜tsx
├── src
│   ├── .umi
│   ├── .umi-production
│   ├── app.ts
│   ├── layouts
│   │   ├── BasicLayout.tsx
│   │   ├── index.less
│   ├── models
│   │   ├── global.ts
│   │   └── index.ts
│   ├── pages
│   │   ├── index.less
│   │   └── index.tsx
│   ├── utils // 推荐目录
│   │   └── index.ts
│   ├── services // 推荐目录
│   │   └── api.ts
│   ├── global.ts
│   ├── global.(css|less|sass|scss)
│   ├── overrides.(css|less|sass|scss)
│   ├── favicon.(ico|gif|png|jpg|jpeg|svg|avif|webp)
│   └── loading.tsx
├── node_modules
│   └── .cache
│       ├── bundler-webpack
│       ├── mfsu
│       └── mfsu-deps
├── .env
├── plugin.ts 
├── .umirc.ts // 与 config/config 文件 2 选一
├── package.json
├── tsconfig.json
└── typings.d.ts
```

### 4.1 package.json

包含插件和插件集，以 `@umijs/preset-`、`@umijs/plugin-`、`umi-preset-` 和 `umi-plugin-` 开头的依赖会被自动注册为插件或插件集。

### 4.2 .env

环境变量，比如：

```
PORT=8888
COMPRESS=none
```

### 4.3 .umirc.ts

> 与 `config/config.ts` 文件功能相同，2 选 1 。`.umirc.ts` 文件优先级较高

配置文件，包含 Umi 内置功能和插件的配置。

配置文件的优先级见：[UMI_ENV](https://umijs.org/docs/guides/env-variables#umi_env)

### 4.4 config/config.ts

> 与 `.umirc.ts` 文件功能相同，2 选 1 。`.umirc.ts` 文件优先级较高

配置文件，包含 Umi 内置功能和插件的配置。

### 4.5 dist 目录

执行 `umi build` 后，产物默认会存放在这里。可通过配置修改产物输出路径。

### 4.6 mock 目录

存储 mock 文件，此目录下所有 `js` 和 `ts` 文件会被解析为 mock 文件。用于本地的模拟数据服务。

### 4.7 public 目录

此目录下所有文件会被 copy 到输出路径。

### 4.8 `src` 目录

#### 4.8.1 .umi 目录

dev 时的临时文件目录，比如入口文件、路由等，都会被临时生成到这里。**不要提交 .umi 目录到 git 仓库，他们会在 `umi dev` 时被删除并重新生成。**

#### 4.8.2 .umi-production 目录

build 时的临时文件目录，比如入口文件、路由等，都会被临时生成到这里。**不要提交 .umi-production 目录到 git 仓库，他们会在 `umi build` 时被删除并重新生成。**

#### 4.8.3 app.[ts｜tsx]

运行时配置文件，可以在这里扩展运行时的能力，比如修改路由、修改 render 方法等。运行时配置是跑在浏览器端，因此我们可以在这里写函数、jsx 语法，import 浏览器端依赖等等。

#### 4.8.4 layouts/index.tsx

约定式路由时的全局布局文件，实际上是在路由外面套了一层。比如，你的路由是：

```js
[
  { path: '/', component: './pages/index' },
  { path: '/users', component: './pages/users' },
]
```

从组件角度可以简单的理解为如下关系：

```jsx
<layout>
  <page>1</page>
  <page>2</page>
</layout>
```

#### 4.8.5 pages 目录

所有路由组件存放在这里。使用约定式路由时，约定 `pages` 下所有的 `(j|t)sx?` 文件即路由。使用约定式路由，意味着不需要维护可怕的路由配置文件。最常用的有基础路由和动态路由（用于详情页等，需要从 url 取参数的情况）

**1.基础路由**

假设 `pages` 目录结构如下：

```diff
+ pages/
  + users/
    - index.js
  - index.js
```

那么，会自动生成路由配置如下：

```js
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/', component: './pages/users/index.js' },
];
```

**2.动态路由**

约定，带 `$` 前缀的目录或文件为动态路由。若 `$` 后不指定参数名，则代表 `*` 通配，比如以下目录结构：

```diff
+ pages/
  + foo/
    - $slug.js
  + $bar/
    - $.js
  - index.js
```

会生成路由配置如下：

```js
[
  { path: '/', component: './pages/index.js' },
  { path: '/foo/:slug', component: './pages/foo/$slug.js' },
  { path: '/:bar/*', component: './pages/$bar/$.js' },
];
```

**3../src/pages/404.js**

当访问的路由地址不存在时，会自动显示 404 页面。只有 build 之后生效。调试的时候可以访问 `/404` 。

#### 4.8.6 global.(j|t)sx?

在入口文件最前面被自动引入，可以考虑在此加入 polyfill。Umi 区别于其他前端框架，没有显式的程序主入口，如 `src/index.js`，所以在引用某些模块的时候，如果模块功能要求在程序主入口添加代码时，你就可以写到这个文件。

#### 4.8.7 global.(css|less|sass|scss)

这个文件不走 css modules，自动被引入，可以写一些全局样式，它的引入位置很靠前，所以优先级相对较低；如果想覆盖三方依赖样式，推荐使用 `overrides.(css|less|sass|scss)`。

#### 4.8.8 overrides.(css|less|sass|scss)

这个文件不走 css modules，自动被引入，专用于覆盖三方依赖的样式；该文件中所有的 CSS 选择器都会被自动加上 `body` 前缀以确保优先级始终高于原有选择器，这样一来在页面切换时有异步 chunk 动态插入的情况下样式覆盖也能生效。

#### 4.8.9 loading.(tsx|jsx)

定义懒加载过程中要显示的加载动画。Umi 4 默认按页拆包，所以这近似等价于 Umi 3 中的 `dynamicImport.loading` 选项。

### 4.8.10 plugin.ts

存在这个文件，会被当前项目加载为 Umi 插件，你可以在这里实现一些插件级的功能。

```ts
import type { IApi } from 'umi';
 
export default (api: IApi) => {
  api.onDevCompileDone((opts) => {
    opts;
    // console.log('> onDevCompileDone', opts.isFirstCompile);
  });
  api.onBuildComplete((opts) => {
    opts;
    // console.log('> onBuildComplete', opts.isFirstCompile);
  });
  api.chainWebpack((memo) => {
    memo;
  });
};
```

### 4.8.11 favicon

约定如果存在 `src/favicon.(ico|gif|png|jpg|jpeg|svg|avif|webp)` 文件，将会使用它作为构建网页的 `shortcut icon`，如存在 `src/favicon.png` 则构建时会生成：

```html
<link rel="shortcut icon" href="/favicon.png">
```

支持多种文件后缀，按以下优先级匹配：

```js
const FAVICON_FILES = [
  'favicon.ico',
  'favicon.gif',
  'favicon.png',
  'favicon.jpg',
  'favicon.jpeg',
  'favicon.svg',
  'favicon.avif',
  'favicon.webp',
];
```

如果约定方式不满足你的需求，可以使用 [favicons](https://umijs.org/docs/api/config#favicons) 配置。

> 配置优先级会大于约定




# 24 【UmiJS基础】

## 1.常用配置项

### 1.1 title

- 类型：`string`
- 默认值：`null`

配置全局页面 title，暂时只支持静态的 Title。

 比如：

```js
export default {
  title: 'hi',
};
```

此外，你还可以针对路由配置标题，比如，

```js
export default {
  title: 'hi',
  routes: [
    { path: '/', title: 'Home' },
    { path: '/users', title: 'Users' },
    { path: '/foo' },
  ],
};
```

然后我们访问 `/` 标题是 `Home`，访问 `/users` 标题是 `Users`，访问 `/foo` 标题是默认的 `hi`。

注意：

- 默认不会在 HTML 里输出 `<title>` 标签，通过动态渲染得到
- 配 `exportStatic` 后会为每个 HTML 输出 `<title>` 标签
- 如果需要自行通过 react-helmet 等方式渲染 title，配 `title: false` 可禁用内置的 title 渲染机制

### 1.2 theme

- 类型：`object`
- 默认值：`{}`

配置 less 变量主题。

示例：

```js
export default {
  theme: {
    '@primary-color': '#1DA57A',
  },
};
```

### 1.3 favicons

- 类型：`string[]`
- 默认值：`null`

支持配置多个 favicon 文件。配置 favicons 路径，可以是绝对路径，也可以是基于项目根目录的相对路径。

比如：

```js
export default {
  favicon: [ '/assets/favicon.ico' ],
};
```

> 如果要使用本地的图片，图片请放到 `public` 目录

HTML 中会生成，

```html
<link rel="shortcut icon" type="image/x-icon" href="/assets/favicon.ico" />
```

> 放到`src`目录下会自动在html中生成

### 1.4 proxy

- 类型：`object`
- 默认值：`{}`

配置代理功能。

比如，

```js
proxy: {
  '/api': {
    'target': 'http://jsonplaceholder.typicode.com/',
    'changeOrigin': true,
    'pathRewrite': { '^/api' : '' },
  }
}
```

然后访问 `/api/users` 就能访问到 http://jsonplaceholder.typicode.com/users 的数据。

注意：proxy 功能仅在 dev 时有效。

### 1.5 alias

- 类型：`Record<string, string>`
- 默认值：`{}`

配置别名，对 import 语句的 source 做映射。

比如：

```js
{
  alias: {
    foo: '/tmp/to/foo',
  }
}
```

然后代码里 `import 'foo'` 实际上会 `import '/tmp/to/foo'`。

有几个 `Tip`。

1、alias 的值最好用绝对路径，尤其是指向依赖时，记得加 `require.resolve`，比如，

```js
// ⛔
{
  alias: {
    foo: 'foo',
  }
}
 
// ✅
{
  alias: {
    '@c': require('node:path').resolve(__dirname, './src/components'),
  }
}	
```

如果使用了`ts`，还需要配置

```json
{
	"compilerOptions": {
		"paths": {
			"@c/*": ["src/components/*"],
		}
	}
}
```

> 这样会把默认的配置覆盖，由于知识限制，暂时还未找到比较好的方法去解决。

2、如果不需要子路径也被映射，记得加 `$` 后缀，比如

```js
// import 'foo/bar' 会被映射到 import '/tmp/to/foo/bar'
{
  alias: {
    foo: '/tmp/to/foo',
  }
}
 
// import 'foo/bar' 还是 import 'foo/bar'，不会被修改
{
  alias: {
    foo$: '/tmp/to/foo',
  }
}
```

### 1.6 lessLoader

- 类型：`Object`
- 默认值：`{ modifyVars: userConfig.theme, javascriptEnabled: true }`

设置 less-loader 的 Options。具体参考参考 [less-loader 的 Options](https://github.com/webpack-contrib/less-loader#lessoptions)。

> 默认是用 less@4 版本，如果需要兼容 less@3 请配置使用[less-options-math](https://lesscss.org/usage/#less-options-math)。

`src/styles/index.less`

```less
@myBg: #bfa;

.myMixin(@mixinBg) {
  background-color: @mixinBg;
}
```

`.umirc.ts`

```ts
import { defineConfig } from 'umi'

export default defineConfig({
  lessLoader: {
    modifyVars: {
      hack: `true;@import '@/styles/index.less'`,
    },
  },
})
```

然后就可以使用全局`less`定义的变量、混合...

```less
// background-color: @myBg;
.myMixin(#bfa);
```

## 2.HTML 模板

### 2.1 修改默认模板

默认模板如下，

```html
<!DOCTYPE html>
<html>
<head><title><%= title %></title></head>
<body>
<div id="<%= mountElementId %>"></div>
</body>
</html>
```

如果自定义模板，请确保包含 `<%= title %>` 和 `<%= mountElementId %>`。

新建 `src/pages/document.ejs`，umi 约定如果这个文件存在，会作为默认模板，比如：

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Your App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

在 umi 里配置 title 请查看 [配置标题](https://v3.umijs.org/zh-CN/config#title)。

### 2.2 配置模板

模板里可通过 context 来获取到 umi 提供的变量，context 包含：

- `route`，路由信息，需要打包出多个静态 HTML 时（即配置了 exportStatic 时）有效
- `config`，用户配置信息

比如：

```html
<link rel="icon" type="image/x-icon" href="<%= context.config.publicPath %>favicon.png" />
```

## 3.样式

本文介绍各种在 Umi 项目中使用样式的方式。

### 3.1 使用 CSS 样式

你可以在 Umi 项目中使用 `.css` 文件声明各种样式，然后在 `.js` 文件中引入即可生效。

例如，在 `src/pages/index.css` 文件按照以下代码声明 `.title` 类的样式为红色：

```css
.title {
  color: red;
}
```

然后在 `src/pages/index.js` 文件中引入即可生效。

```jsx
// src/pages/index.js
 
import './index.css';
 
export default function () {
  return <div className="title">Hello World</div>;
}
```

按照此种引入方式的样式会在整个 Umi 项目中生效，即无论你从哪个 `.js` 文件引入，他声明的样式可以在任何页面和组件中使用。如果你想要避免这种情况，可以使用 [CSS Modules](https://umijs.org/docs/guides/styling#使用-css-modules) 的功能来限制样式的作用域。

### 3.2 使用 CSS Modules

在 `js` 文件中引入样式时，如果赋予他一个变量名，就可以将样式以 CSS Module 的形式引入。

```jsx
// src/pages/index.js
 
import styles from './index.css';
 
export default function () {
  return <div className={styles.title}>
    Hello World
  </div>;
}
```

上面的示例中，`index.css` 文件中声明的样式不会对全局样式造成影响，只会对从 `styles` 变量中使用的样式生效。

### 3.3 使用 CSS 预处理器

Umi 默认支持 LESS (推荐), SASS 和 SCSS 样式的导入，你可以直接按照引入 CSS 文件的方式引入并使用这些由 CSS 预处理器处理的样式。

> 💡
>
> 在 Umi 中使用 Sass(Scss) 需要额外安装预处理依赖 如: `npm add -D sass`

```jsx
// src/pages/index.js
 
import './index.less';
import './index.sass';
import './index.scss';
 
export default function () {
  return <div className="title">Hello World</div>;
}
```

同样也支持 CSS Module 的用法：

```jsx
// src/pages/index.js
 
import lessStyles from './index.less';
import sassStyles from './index.sass';
import scssStyles from './index.scss';
 
export default function () {
  return <div className={lessStyles.title}>
    Hello World
    <p className={sassStyles.blue}>I am blue</p>
    <p className={scssStyles.red}>I am red</p>
  </div>;
}
```

### 3.4 使用 Tailwindcss

为项目开启 [Tailwind CSS](https://tailwindcss.com/) 配置，命令执行后，`umi` 会生成 Tailwind CSS 和安装相应的的依赖。

```bash
$umi g tailwindcss
info  - Write package.json
set config:tailwindcss on /Users/umi/playground/.umirc.ts
set config:plugins on /Users/umi/playground/.umirc.ts
info  - Update .umirc.ts
info  - Write tailwind.config.js
info  - Write tailwind.css
```

生成的`tailwind.css`文件会有警告，可以通过`vscode`的配置来解决

`settings.json`

```json
{
  "css.lint.unknownAtRules": "ignore"
}
```

### 3.5 使用 UnoCSS

与 Tailwindcss 相同，Umi 也提供了内置的 [UnoCSS](https://github.com/unocss/unocss) 插件，可以按照相同方式开启。

1. 安装 `plugin-unocss`
2. 安装 `unocss` 及 `@unocss/cli`

```bash
pnpm i unocss @unocss/cli
```

3. 在 Umi 设置中启用插件，并声明会用到 `unocss` 的文件目录

```ts
// .umirc.ts
 
export default {
  plugins: [
    require.resolve('@umijs/plugins/dist/unocss')
  ],
  unocss: {
    // 检测 className 的文件范围，若项目不包含 src 目录，可使用 `pages/**/*.tsx`
    watch: ['src/**/*.tsx']
  },
};
```

4. 在项目目录下加入 `unocss.config.ts` 配置文件，并加入项目需要的 [UnoCSS Presets](https://github.com/unocss/unocss#presets)

```ts
// unocss.config.ts
 
import {defineConfig, presetAttributify, presetUno} from 'unocss';
 
export function createConfig({strict = true, dev = true} = {}) {
  return defineConfig({
    envMode: dev ? 'dev' : 'build', presets: [presetAttributify({strict}), presetUno()],
  });
}
 
export default createConfig(); 
```

5. 启动项目进行开发，插件会监听设置文件中的 `unocss.watch` 字段，动态生成样式文件并自动套用

## 4.使用图片

### 4.1 JS 里使用图片

通过 require 引用相对路径的图片。

比如：

```js
export default () => <img src={require('./foo.png')} />
```

支持别名，比如通过 `@` 指向 src 目录：

```js
export default () => <img src={require('@/foo.png')} />
```

### 4.2 JS 里使用svg

**组件式引入**

```js
import { ReactComponent as Logo } from './logo.svg'
function Analysis() {  return <Logo width={90} height={120} />}
```

**url式引入**

```js
import logoSrc from './logo.svg'
function Analysis() {  return <img src={logoSrc} alt="logo" />}
```

### 4.3 CSS 里使用图片

通过相对路径引用。

比如，

```css
.logo {
  background: url(./foo.png);
}
```

CSS 里也支持别名，但需要在前面加 `~` 前缀，

```css
.logo {
  background: url(~@/foo.png);
}
```

注意：

1. 这是 webpack 的规则，如果切到其他打包工具，可能会有变化
2. less 中同样适用

### 4.4 图片路径问题

项目中使用图片有两种方式，

1. 先把图片传到 cdn，然后在 JS 和 CSS 中使用图片的绝对路径
2. 把图片放在项目里，然后在 JS 和 CSS 中通过相对路径的方式使用

### 4.5 Base64 编译

通过相对路径引入图片的时候，如果图片小于 10K，会被编译为 Base64，否则会被构建为独立的图片文件。

10K 这个阈值可以通过 [inlineLimit 配置](https://v3.umijs.org/zh-CN/config#inlinelimit)修改。

## 5.编码规范

我们通常会在项目中使用 ESLint、Stylelint 来协助我们把控编码质量，为了实现低成本、高性能、更稳定地接入上述工具，Umi 提供了开箱即用的 Lint 能力，包含以下特性：

1. **推荐配置**：提供 ESLint 及 Stylelint 推荐配置，可以直接继承使用
2. **统一的 CLI**：提供 `umi lint` CLI，集成式调用 ESLint 和 Stylelint
3. **规则稳定**：始终确保规则的稳定性，不会出现上游配置更新导致存量项目 lint 失败的情况

其中，ESLint 配置具备如下特点：

1. **仅质量相关**：我们从数百条规则中筛选出数十条与编码质量相关的规则进行白名单开启，回归 Lint 本质，且不会与 Prettier 的规则冲突
2. **性能优先**：部分 TypeScript 的规则实用型低但项目全量编译的成本却很高，我们对这些规则进行禁用以提升性能
3. **内置常用插件**：包含 react、react-hooks、@typescript/eslint、jest，满足日常所需

另外，Stylelint 配置还内置 CSS-in-JS 支持，可以检测出 JS 文件中的样式表语法错误。听起来很有吸引力？来看看如何接入吧。

### 5.1 使用方式

#### 5.1.1 安装

为了节省安装体积，目前仅在 Umi Max 中内置了 Lint 模块，使用 `max lint` 来执行 lint 过程。**如果你使用的是 Umi，需要先安装 `@umijs/lint`**：

```bash
$ npm i @umijs/lint -D
# or
$ pnpm add @umijs/lint -D
```

然后安装 ESLint 及 Stylelint：

```bash
$ npm i eslint stylelint -D
# or
$ pnpm add eslint stylelint -D
```

#### 5.1.2 启用配置

在 `.eslintrc.js` 及 `.stylelintrc.js` 里继承 Umi 提供的配置：

```js
// .eslintrc.js
module.exports = {
  // Umi 项目
  extends: require.resolve('umi/eslint'),
 
  // Umi Max 项目
  extends: require.resolve('@umijs/max/eslint'),
  
   settings: {
    // 解决警告：Warning: React version was set to "detect" in eslint-plugin-react settings, but the "react" package is not installed. Assuming latest React version for linting.
    react: {
      version: '999.999.999',
    },
  },
}
 
// .stylelintrc.js
module.exports = {
  // Umi 项目
  // pnpm add stylelint-config-recess-order -D 
  extends: [require.resolve('umi/stylelint'), 'stylelint-config-recess-order'],
 
  // Umi Max 项目
  extends: [require.resolve('@umijs/max/stylelint'), 'stylelint-config-recess-order'],
}
```

编写检查忽略文件 `.eslintignore` 及 `.stylelintignore` 里继承 Umi 提供的配置：

```bash
# .eslintrc.js
/config
/dist
public
dist
.umi
mock
node_modules
 
# .stylelintrc.js
node_modules
.umi
.umi-production
tailwind.*
```

在配置文件创建完毕后，我们其实已经可以通过 `eslint`、`stylelint` 命令来执行 lint 了，但我们仍然推荐使用 `umi lint` 命令，以获得更便捷的体验。

#### 5.1.3 CLI

`umi lint` 命令的用法如下：

```bash
$ umi lint [glob] [--fix] [--eslint-only] [--stylelint-only] [--cssinjs]
```

参数说明：

```bash
 [glob]: 可选，指定要 lint 的文件，默认为 `{src,test}/**/*.{js,jsx,ts,tsx,css,less}`
  --quiet: 可选，禁用 `warn` 规则的报告，仅输出 `error`
  --fix: 可选，自动修复 lint 错误
  --eslint-only: 可选，仅执行 ESLint
  --stylelint-only: 可选，仅执行 Stylelint
  --cssinjs: 可选，为 Stylelint 启用 CSS-in-JS 支持
```

通常来说，直接执行 `umi lint` 应该就能满足大部分情况。

### 5.2 与 Git 工作流结合

### 5.3 Prettier

在启用 `umi lint` 的基础上，我们也建议与 [Prettier](https://prettier.io/docs/en/install.html) 一同使用，以确保团队的代码风格是基本一致的。

为项目生成 [prettier](https://prettier.io/) 配置，命令执行后，`umi` 会生成推荐的 prettier 配置和安装相应的依赖。

```bash
$umi g prettier
info  - Write package.json
info  - Write .prettierrc
info  - Write .prettierignore
```

我的`.prettierrc`配置

```json
{
  "printWidth": 120,
  "tabWidth": 2,
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all",
  "proseWrap": "never",
  "arrowParens": "avoid",
  "htmlWhitespaceSensitivity": "strict",
  "endOfLine": "auto",
  "bracketSpacing": true,
  "jsxSingleQuote": false,
  "bracketSameLine": false,
  "useTabs": false,
  "overrides": [{ "files": ".prettierrc", "options": { "parser": "json" } }],
  "plugins": ["prettier-plugin-organize-imports", "prettier-plugin-packagejson"]
}
```

可参考 Prettier 文档将其配置到 lint-staged 中：https://prettier.io/docs/en/install.html#git-hooks

## 6.路由

在 Umi 应用是[单页应用](https://en.wikipedia.org/wiki/Single-page_application)，页面地址的跳转都是在浏览器端完成的，不会重新请求服务端获取 html，html 只在应用初始化时加载一次。所有页面由不同的组件构成，页面的切换其实就是不同组件的切换，你只需要在配置中把不同的路由路径和对应的组件关联上。

### 6.1 配置路由

在配置文件中通过 `routes` 进行配置，格式为路由信息的数组。

比如：

```js
export default {
  routes: [
    { path: '/', component: 'index' },
    { path: '/user', component: 'user' },
  ],
}
```

Umi 4 默认根据路由来进行 JavaScript 模块按需加载。如果需要在路由组件加载的过程中配置自定义加载组件，在项目 `src` 目录下创建 `loading.tsx` 或者 `loading.jsx` 或者 `loading.js` 文件，默认导出的组件会在组件加载的时候渲染。

> 💡
>
> 你可以在 Chrome 的调试工具的网络 tab 中将网络设置成低速，然后切换路由查看动态加载中组件的展示。

#### 6.1.1 path

- Type: `string`

`path` 只支持两种占位符配置，第一种是动态参数 `:id` 的形式，第二种是 `*` 通配符，通配符只能出现路由字符串的最后。

✅ 以下是目前***支持***的路由路径配置形式：

```bash
/groups
/groups/admin
/users/:id
/users/:id/messages
/files/*
/files/:id/*
```

❌ 以下是目前***不支持***的路由路径配置形式：

```bash
/users/:id?
/tweets/:id(\d+)
/files/*/cat.jpg
/files-*
```

#### 6.1.2 component

- Type: `string`

配置 location 和 path 匹配后用于渲染的 React 组件路径。可以是绝对路径，也可以是相对路径，如果是相对路径，会从 `src/pages` 开始找起。

如果指向 `src` 目录的文件，可以用 `@`，也可以用 `../`。比如 `component: '@/layouts/basic'`，或者 `component: '../layouts/basic'`，推荐用前者。

#### 6.1.3 routes

配置子路由，通常在需要为多个路径增加 layout 组件时使用。

比如：

```ts
export default {
  routes: [
    { path: '/login', component: 'login' },
    {
      path: '/',
      component: '@/layouts/index',
      routes: [
        { path: '/list', component: 'list' },
        { path: '/admin', component: 'admin' },
      ],
    }, 
  ],
}
```

然后在 `src/layouts/index` 中通过 `<Outlet/>` 渲染子路由，

```tsx
import {Outlet} from 'umi'
 
export default (props) => {
  return <div style={{ padding: 20 }}> 
    <Outlet/> 
  </div>;
}
```

这样，访问 `/list` 和 `/admin` 就会带上 `src/layouts/index` 这个 layout 组件。

#### 6.1.4 redirect

- Type: `string`

配置路由跳转。

比如：

```ts
export default {
  routes: [
    { path: '/', redirect: '/list' },
    { path: '/list', component: 'list' },
  ],
}
```

访问 `/` 会跳转到 `/list`，并由 `src/pages/list` 文件进行渲染。

#### 6.1.5 wrappers

- Type: `string[]`

配置路由组件的包装组件，通过包装组件可以为当前的路由组件组合进更多的功能。 比如，可以用于路由级别的权限校验：

```ts
export default {
  routes: [
    { path: '/user', component: 'user',
      wrappers: [
        '@/wrappers/auth',
      ],
    },
    { path: '/login', component: 'login' },
  ]
}
```

然后在 `src/wrappers/auth` 中，

```tsx
import { Navigate, Outlet } from 'umi'
 
export default (props) => {
  const { isLogin } = useAuth();
  if (isLogin) {
    return <Outlet />;
  } else{
    return <Navigate to="/login" />;
  }
}
```

这样，访问 `/user`，就通过 `auth` 组件做权限校验，如果通过，渲染 `src/pages/user`，否则跳转到 `/login`。

> 🚨
>
> `wrappers` 中的每个组件会给当前的路由组件增加一层嵌套路由，如果你希望路由结构不发生变化，推荐使用高阶组件。先在高阶组件中实现 wrapper 中的逻辑，然后使用该高阶组件装饰对应的路由组件。

举例：

```tsx
// src/hocs/withAuth.jsx
import { Navigate } from 'umi'
 
const withAuth = (Component) => ()=>{
  const { isLogin } = useAuth();
  if (isLogin) {
    return <Component />;
  } else{
    return <Navigate to="/login" />;
  }
}
```

```tsx
// src/pages/user.jsx
 
const TheOldPage = ()=>{
  ...
}
 
export default withAuth(TheOldPage)
```

#### 6.1.6 title

- Type: `string`

配置路由的标题。

### 6.2 页面跳转

命令式跳转请使用 [`history`](https://umijs.org/docs/api/api#history) API

和 history 相关的操作，用于获取当前路由信息、执行路由跳转、监听路由变更。

```tsx
// 建议组件或 hooks 里用 useLocation 取
import { useLocation } from 'umi';
export default function Page() {
  let location = useLocation();
  return (
    <div>
     { location.pathname }
     { location.search }
     { location.hash }
    </div>
  );
}
```

如果在 React 组件和 Hooks 之外获取当前路由信息。

```js
// location 对象，包含 pathname、search 和 hash
window.location.pathname;
window.location.search;
window.location.hash;
```

命令式路由跳转。

```ts
import { history } from 'umi';
 
// 跳转到指定路由
history.push('/list');
 
// 带参数跳转到指定路由
history.push('/list?a=b&c=d#anchor', state);
history.push({
    pathname: '/list',
    search: '?a=b&c=d',
    hash: 'anchor',
  },
  {
    some: 'state-data',
  }
);
 
// 跳转当前路径，并刷新 state
history.push({}, state)
 
// 跳转到上一个路由
history.back();
history.go(-1);
```

> 🚨
>
> 注意：history.push 和 history.replace 需要使用 `state` 需将 `state` 作为这两个 API 的第二个参数传递

路由监听。

```ts
import { history } from 'umi';
 
const unlisten = history.listen(({ location, action }) => {
  console.log(location.pathname);
});
unlisten();
```

组件内还可以使用 [`useNavigate`](https://umijs.org/docs/api/api#usenavigate) hook

`useNavigate` 钩子函数返回一个可以控制跳转的函数；比如可以用在提交完表单后跳转到其他页面。

```ts
declare function useNavigate(): NavigateFunction;
 
interface NavigateFunction {
  (
    to: To,
    options?: { replace?: boolean; state?: any }
  ): void;
  (delta: number): void;
}
```

### 6.3 错误页配置

`src/404.tsx`

```tsx
export default function Error() {
  return <div>404</div>
}
```

`.umirc.ts`

```ts
routes: [
	...
  {
    path: '*',
    component: '@/404.tsx',
    title: '错误-404',
  },
],
```

示例：

- 跳转路径

```ts
import { useNavigate } from 'umi';
 
let navigate = useNavigate();
navigate("../success", { replace: true });
```

- 返回上一页

```ts
import { useNavigate } from 'umi';
 
let navigate = useNavigate();
navigate(-1);
```

### 6.4 路由组件参数

Umi4 使用 [react-router@6](https://reactrouter.com/docs/en/v6/api) 作为路由组件，路由参数的获取使其 hooks。

## 7.Mock

Umi 提供了开箱即用的 Mock 功能，能够用方便简单的方式来完成 Mock 数据的设置。

> 💡
>
> 什么是 Mock 数据：在前后端约定好 API 接口以后，前端可以使用 Mock 数据来在本地模拟出 API 应该要返回的数据，这样一来前后端开发就可以同时进行，不会因为后端 API 还在开发而导致前端的工作被阻塞。

### 7.1 目录约定

Umi 约定 `/mock` 目录下的所有文件为 [Mock 文件](https://umijs.org/docs/guides/mock#mock-文件)，例如这样的目录结构：

```bash
.
├── mock
    ├── todos.ts
    ├── items.ts
    └── users.ts
└── src
    └── pages
        └── index.tsx
```

则 `/mock` 目录中的 `todos.ts`, `items.ts` 和 `users.ts` 就会被 Umi 视为 [Mock 文件](https://umijs.org/docs/guides/mock#mock-文件) 来处理。

### 7.2 Mock 文件

Mock 文件默认导出一个对象，而对象的每个 Key 对应了一个 Mock 接口，值则是这个接口所对应的返回数据，例如这样的 Mock 文件：

```ts
// ./mock/users.ts
 
export default {
 
  // 返回值可以是数组形式
  'GET /api/users': [
    { id: 1, name: 'foo' },
    { id: 2, name: 'bar' }
  ],
 
  // 返回值也可以是对象形式
  'GET /api/users/1': { id: 1, name: 'foo' },
 
}
```

就声明了两个 Mock 接口，透过 `GET /api/users` 可以拿到一个带有两个用户数据的数组，透过 `GET /api/users/1` 可以拿到某个用户的模拟数据。

#### 7.2.1 请求方法

当 Http 的请求方法是 GET 时，可以省略方法部分，只需要路径即可，例如：

```ts
// ./mock/users.ts
 
export default {
 
  '/api/users': [
    { id: 1, name: 'foo' },
    { id: 2, name: 'bar' }
  ],
 
  '/api/users/1': { id: 1, name: 'foo' },
 
}
```

也可以用不同的请求方法，例如 `POST`，`PUT`，`DELETE`：

```ts
// ./mock/users.ts
 
export default {
 
  'POST /api/users': { result: 'true' },
 
  'PUT /api/users/1': { id: 1, name: 'new-foo' },
 
}
```

#### 7.2.3 自定义函数

除了直接静态声明返回值，也可以用函数的方式来声明如何计算返回值，例如：

```ts
export default {
 
  'POST /api/users/create': (req, res) => {
    // 添加跨域请求头
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.end('ok');
  }
 
}
```

关于 `req` 和 `res` 的 API 可参考 [Express@4 官方文档](https://expressjs.com/en/api.html) 来进一步了解。

####  7.2.4 defineMock

另外，也可以使用 `defineMock` 类型帮助函数来提供编写 mock 对象的代码提示，如：

```ts
import { defineMock } from "umi";
 
export default defineMock({
  "/api/users": [
    { id: 1, name: "foo" },
    { id: 2, name: "bar" },
  ],
  "/api/users/1": { id: 1, name: "foo" },
  "GET /api/users/2": (req, res) => {
    res.status(200).json({ id: 2, name: "bar" });
  },
});
```

`defineMock` 仅仅提供类型提示，入参与出参完全一致。

### 7.3 关闭 Mock

Umi 默认开启 Mock 功能，如果不需要的话可以从配置文件关闭：

```ts
// .umirc.ts
 
export default {
  mock: false,
};
```

或是用环境变量的方式关闭：

```
MOCK=none umi dev
```

### 7.4 引入 Mock.js

在 Mock 中我们经常使用 [Mock.js](http://mockjs.com/) 来帮我们方便的生成随机的模拟数据，如果你使用了 Umi 的 Mock 功能，建议你搭配这个库来提升模拟数据的真实性：

```ts
import mockjs from 'mockjs';
 
export default {
  // 使用 mockjs 等三方库
  'GET /api/tags': mockjs.mock({
    'list|100': [{ name: '@city', 'value|1-100': 50, 'type|0-2': 1 }],
  }),
};
```

