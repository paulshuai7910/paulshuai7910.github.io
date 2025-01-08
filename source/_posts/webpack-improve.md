---
title: webpack 优化方案
date: 2024-04-12 20:09:55
tags:
---

# 代码分割（Code Splitting）

## import 动态导入和代码拆分

import() 是 JavaScript 的一个动态模块加载语法，它允许在运行时动态地加载模块。与传统的 import 语法不同，import() 是一个 异步 操作，并返回一个 Promise。这样，可以在需要时加载模块，而不是在应用启动时加载所有模块。

```js
import(moduleSpecifier)
  .then((module) => {
    // 使用动态加载的模块
  })
  .catch((error) => {
    // 错误处理
  })
```

- 按需加载

```js
import React, { Suspense, lazy } from "react"

// 使用 lazy 函数动态加载组件
const MyComponent = lazy(() => import("./MyComponent"))

function App() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <MyComponent />
      </Suspense>
    </div>
  )
}

export default App
```

- 代码拆分
  Webpack 内部会自动处理动态导入的代码拆分。每当 import() 被调用时，Webpack 会：

1. 生成一个新的 JavaScript 文件（即代码块）。
2. 在加载该模块时，异步地加载这个新生成的文件。
3. 如果该模块已经被加载过，则不会重复加载，而是使用缓存的模块。

```js
// index.js
import("./moduleA").then((module) => {
  console.log(module)
})
```

Webpack 会将 moduleA 拆分成一个独立的文件，假设叫做 moduleA.bundle.js。这样，moduleA.bundle.js 文件会在需要时被加载，而不是在页面加载时就被加载。

## 使用 Webpack 配置优化代码拆分

Webpack 提供了一些配置来优化动态导入的代码拆分，比如使用 splitChunks 插件来更细粒度地控制代码拆分。

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: "all", // 对所有的模块进行代码拆分
    },
  },
}
```

- chunks: 'all'：表示对所有模块进行代码拆分，无论是动态导入还是普通导入的模块都可以拆分。
- 通过合理配置 splitChunks，你可以确保常用的第三方库（比如 lodash、react）被提取成一个共享模块，避免在多个入口文件中重复加载。

# Tree Shaking

- 原理：基于 ES6 模块静态结构的特性，通过静态分析找出并删除未引用的代码。
- 配置：在生产模式下（mode: 'production'），Webpack 默认启用 Tree Shaking。确保使用 ES6 模块语法。

# 增加入口文件，共享依赖,代码分割

防止重复引用：

```js
const path = require("path")

module.exports = {
  mode: "development",
  entry: {
    index: {
      import: "./src/index.js",
      dependOn: "shared",
    },
    main: {
      import: "./src/main.js",
      dependOn: "shared",
    },
    shared: "lodash",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
}
```

# 线程加速器

多线程可以提高程序的效率，我们也可以在 Webpack 中使用。而 thread-loader 是一个可以在 Webpack 中启用多线程的加载器
`npm i thread-loader -D`

```js
{
        test: /\.js$/,
        use: [
          'thread-loader',
          'babel-loader'
        ],
}
```

# 缓存加载器

`npm i cache-loader -D`

# 代码压缩

css-minimizer-webpack-plugin 可以压缩和去重 CSS 代码。
terser-webpack-plugin 可以压缩和去重 JavaScript 代码。

# 方案：懒加载与路由结合

动态导入通常和路由结合使用，这样你可以在用户访问某个页面时再加载相关模块。这种方法常见于单页面应用（SPA）。

```js
// React 示例：使用 React Router 和动态导入进行按需加载
import React, { Suspense, lazy } from "react"
import { BrowserRouter as Router, Route, Switch } from "react-router-dom"

const Home = lazy(() => import("./Home"))
const About = lazy(() => import("./About"))

function App() {
  return (
    <Router>
      <Suspense fallback={<div>Loading...</div>}>
        <Switch>
          <Route path="/home" component={Home} />
          <Route path="/about" component={About} />
        </Switch>
      </Suspense>
    </Router>
  )
}

export default App
```

# 监控和性能分析

- 使用 webpack-bundle-analyzer：该插件可以帮助你分析打包后的文件大小，找出可以优化的地方。

- 使用 speed-measure-webpack-plugin：分析 Webpack 打包过程中 Loader 和 Plugin 的耗时，有助于找到构建过程中的性能瓶颈。
