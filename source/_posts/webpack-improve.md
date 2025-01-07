---
title: webpack 优化方案
date: 2024-04-12 20:09:55
tags:
---

# 代码分割（Code Splitting）

import() 是 JavaScript 的一个动态模块加载语法，它允许在运行时动态地加载模块。与传统的 import 语法不同，import() 是一个 异步 操作，并返回一个 Promise。这样，可以在需要时加载模块，而不是在应用启动时加载所有模块。

- 动态导入：使用动态导入（import()）来按需加载代码块，减少初始加载时间。

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

- 配置 SplitChunksPlugin：在 Webpack 配置中使用 optimization.splitChunks 选项来分割代码，提取公共模块和第三方库到单独的 chunk 中。

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

# 模块延迟加载 (动态加载)

webpack 提供了两个类似的技术实现动态代码分离。第一种，也是推荐选择的方式，是使用符合 ECMAScript 提案的 import() 语法实现动态导入。第二种则是 webpack 的遗留功能，使用 webpack 特定的 require.ensure. 当 webpack 识别到这两个语法时，会自动将被导入的模块分割成一个单独的文件，然后在需要的时候再去加载这个文件。

模块延迟加载后，大 js 文件会被分割成多个小 js 文件，加载时网页按需加载，大大提高了首屏的加载速度。

```js
// src/router/index.js
const routes = [
  {
    path: "/login",
    name: "login",
    component: login,
  },
  {
    path: "/home",
    name: "home",
    // lazy-load
    component: () => import("../views/home/home.vue"),
  },
]
```

- 监控和性能分析
  使用 webpack-bundle-analyzer：该插件可以帮助你分析打包后的文件大小，找出可以优化的地方。
- 使用 speed-measure-webpack-plugin：分析 Webpack 打包过程中 Loader 和 Plugin 的耗时，有助于找到构建过程中的性能瓶颈。
