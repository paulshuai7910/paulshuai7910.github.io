---
title: node 调试工具
date: 2024-10-09 12:49:11
tags:
---

### 1. **使用 `console` 进行调试**

使用 `console.log()`、`console.error()` 等打印输出调试信息。这种方法适用于快速查看变量值、函数执行情况等。

```javascript
const a = 5
console.log(a) // 输出变量 a 的值
```

#### 优点：

- 简单直接，适用于快速调试。
- 不需要任何额外的工具或配置。

#### 缺点：

- 不适用于复杂的调试需求，尤其是在调试异步代码时，可能很难跟踪。
- 需要手动清除 `console` 语句，可能导致代码污染。

### 2. **Node.js 内置调试器 (`--inspect` 或 `--inspect-brk`)**

Node.js 提供了内置的调试工具，可以通过命令行参数启用。这种方法非常强大，可以配合 Chrome DevTools 或 Visual Studio Code 使用。

#### 启用调试：

- **`--inspect`**：启动调试器，但程序会继续运行。
- **`--inspect-brk`**：启动调试器并在第一行代码处暂停，适用于需要在代码开始运行时调试的场景。

```bash
node --inspect app.js
# 或者
node --inspect-brk app.js
```

启用调试后，Node.js 会在默认端口 `9229` 启动调试服务。你可以使用 Chrome 浏览器或其他支持的调试工具连接到该端口进行调试。

#### 使用 Chrome DevTools：

1. 打开 Chrome 浏览器，访问 `chrome://inspect`。
2. 点击 "Configure" 并确保 "localhost:9229" 已经添加。
3. 在 "Remote Targets" 下找到你的 Node.js 程序，点击 "inspect" 进行调试。

#### 使用 Visual Studio Code 调试：

1. 在 VS Code 中，打开你的项目。
2. 在 `.vscode` 目录中创建一个 `launch.json` 配置文件：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/app.js"
    }
  ]
}
```

3. 使用 VS Code 的调试面板启动调试。

#### 优点：

- 通过 Chrome DevTools 或 VS Code 提供的强大功能，可以查看堆栈、变量、设置断点、逐步执行等。
- 支持异步代码调试，能够方便地查看回调函数、`Promise` 等执行情况。

#### 缺点：

- 需要手动启动调试模式，不适合频繁使用。

### 3. **使用 `debug` 模块**

[`debug`](https://www.npmjs.com/package/debug) 是一个非常流行的第三方库，可以帮助开发者在 Node.js 应用中轻松启用和禁用调试输出。它允许你通过命名空间控制日志输出，非常适用于复杂的应用。

#### 安装：

```bash
npm install debug
```

#### 使用：

```javascript
const debug = require("debug")("myapp:server")
const http = require("http")

http
  .createServer((req, res) => {
    debug("Request received")
    res.end("Hello, world!")
  })
  .listen(3000)

debug("Server is running on port 3000")
```

通过设置 `DEBUG` 环境变量，你可以控制哪些命名空间的调试信息会被打印出来。

```bash
DEBUG=myapp:* node app.js
```

#### 优点：

- 能够轻松控制日志输出，适合调试多个模块。
- 可以启用不同的命名空间，帮助开发者更好地跟踪不同部分的执行情况。

#### 缺点：

- 需要手动引入 `debug` 库，可能会增加额外的依赖。

### 4. **使用 `node-inspect`（CLI 调试）**

Node.js 自带的命令行调试工具 `node-inspect` 提供了一个交互式的调试体验。你可以通过它在命令行中调试 Node.js 程序。

#### 启动调试：

```bash
node-inspect app.js
```

这将启动一个交互式调试器，你可以输入调试命令来逐步执行代码，检查变量等。

#### 优点：

- 适用于命令行界面中进行简单调试。
- 不需要依赖其他工具或图形化界面。

#### 缺点：

- 命令行调试体验较差，不如图形化工具（如 Chrome DevTools）直观。

### 5. **使用 `nodemon` 自动重启**

[`nodemon`](https://www.npmjs.com/package/nodemon) 是一个工具，用于自动检测代码更改并重启应用程序，特别适用于开发过程中的频繁调试。通过结合 `nodemon` 和调试工具（如 `--inspect`），可以让调试过程更加高效。

#### 安装：

```bash
npm install -g nodemon
```

#### 使用：

```bash
nodemon --inspect app.js
```

这样，`nodemon` 会在你修改代码时自动重启 Node.js 程序，并且仍然保持调试功能。

#### 优点：

- 自动检测文件变动，适合频繁修改代码时使用。
- 能与 `--inspect` 一起工作，提供更高效的调试体验。

#### 缺点：

- 可能对性能有一定影响，尤其是在大型项目中。

### 6. **使用断点调试**

在 Visual Studio Code 中，你可以通过设置断点来暂停代码的执行，检查当前堆栈、变量等信息。这是通过 `launch.json` 配置文件进行设置的。

#### 设置断点：

1. 在 VS Code 中打开文件。
2. 在代码的行号上点击，设置断点。
3. 启动调试，执行代码时会在断点处暂停。

#### 优点：

- 通过图形化界面管理断点，可以更直观地跟踪代码执行。
- 适合复杂的调试需求，可以检查堆栈、变量、对象等。

#### 缺点：

- 需要使用专门的 IDE，如 Visual Studio Code。

### 7. **使用性能分析工具（例如 `clinic.js`）**

如果你需要分析 Node.js 应用的性能，`clinic.js` 是一个非常强大的工具，它可以帮助你分析 CPU 使用情况、内存使用情况、事件循环等。

#### 安装：

```bash
npm install -g clinic
```

#### 使用：

```bash
clinic doctor -- node app.js
```

这将启动性能分析，`clinic` 会生成一个报告，帮助你发现性能瓶颈。

#### 优点：

- 强大的性能分析工具，适用于大型项目的性能调优。
- 可以帮助开发者了解 Node.js 应用的性能瓶颈。

#### 缺点：

- 主要用于性能分析，调试方面的功能较弱。

---

### 总结

在 Node.js 开发中，调试是一个非常重要的步骤，常用的调试方式有：

1. 使用 `console.log()` 打印输出（适用于简单调试）。
2. 使用 `--inspect` 或 `--inspect-brk` 启动内置调试器（结合 Chrome DevTools 或 VS Code 使用）。
3. 使用 `debug` 模块来控制日志输出。
4. 使用命令行调试工具 `node-inspect` 进行交互式调试。
5. 使用 `nodemon` 来实现代码更改时自动重启并保持调试功能。
6. 在 IDE（如 VS Code）中设置断点进行调试。
7. 使用 `clinic.js` 等工具进行性能分析。

选择合适的调试工具和方法，能够提高开发效率、帮助排查问题。
