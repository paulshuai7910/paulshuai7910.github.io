---
title: node++
date: 2024-05-04 13:48:19
tags:
---

# 如何增加线程

默认情况下所有的 JavaScript 代码都是在单线程环境中执行的。然而，为了处理高并发和 CPU 密集型任务，Node.js 提供了几种方式来增加线程来执行并行任务

- Worker Threads：Node.js 原生支持的多线程机制。
- 线程池（Thread Pool）：通过 libuv 提供的 I/O 密集型任务多线程支持。
- 子进程（Child Processes）：虽然不是线程，但可以通过 child_process 模块生成新的进程来执行任务。

## 使用 Worker Threads

Node.js 从 v10.5.0 开始引入了 Worker Threads 模块，允许你在多个线程中并行执行 JavaScript 代码。Worker Threads 非常适合处理计算密集型任务，因为它们可以利用多核 CPU 来并行处理任务。

```js
// main.js
const { Worker } = require("worker_threads")

const worker = new Worker("./worker.js")

worker.on("message", (message) => {
  console.log(`Main thread received message: ${message}`)
})

worker.on("error", (error) => {
  console.error(`Worker thread error: ${error}`)
})

worker.on("exit", (code) => {
  console.log(`Worker thread exited with code: ${code}`)
})
```

```js
// worker.js
const { parentPort } = require("worker_threads")

// 执行计算密集型任务
const heavyTask = () => {
  let sum = 0
  for (let i = 0; i < 1e9; i++) {
    sum += i
  }
  return sum
}

const result = heavyTask()
parentPort.postMessage(result) // 将结果传回主线程
```

# 控制事件池的大小(使用线程池（Thread Pool）)

Node.js 默认分配了一个包含 4 个线程的线程池，开发者可以通过设置环境变量 UV_THREADPOOL_SIZE 来调整线程池的大小。例如：

```bash
export UV_THREADPOOL_SIZE=8
```

这样可以增加并行执行异步操作的线程数量，但需要注意，如果线程池大小设置过大，可能会导致过多的线程上下文切换，反而降低性能。

# 使用子进程（Child Processes）

如果你希望以独立的进程方式增加并行性，可以使用 child_process 模块创建新的子进程。子进程与父进程是完全独立的，因此它们不会共享内存空间，但可以通过消息传递进行通信。

```js
const { fork } = require("child_process")

// 创建一个子进程来处理任务
const child = fork("./child.js")

child.on("message", (message) => {
  console.log(`Received from child process: ${message}`)
})

child.send("start")
```

# 事件池的局限性

尽管事件池能够帮助 Node.js 实现异步 I/O 操作，但它也有一定的局限性：

- 线程竞争：由于线程池的线程数是有限的，当大量 I/O 操作同时进行时，线程池中的线程可能会成为瓶颈，导致操作被排队等待执行。
- 计算密集型任务：尽管事件池可以执行计算密集型任务，但这些任务仍然会占用线程池中的线程，可能导致其他 I/O 操作的延迟。因此，对于非常耗时的计算任务，使用外部的工作线程或集群来处理可能是更好的选择。

# 常用模块

- fs：文件系统模块，用于读取和写入文件。
- http：HTTP 模块，用于创建和处理 HTTP 请求和响应。
- path：路径模块，用于处理文件路径。
- os：操作系统模块，用于获取操作系统信息和操作。
- crypto：加密模块，用于加密和解密数据。
- zlib：压缩模块，用于压缩和解压缩数据。
- stream：流模块，用于处理数据流。
- events：事件模块，用于处理事件和事件监听器。
- dgram：UDP 模块，用于创建和处理 UDP 数据包。
- net：网络模块，用于创建和处理网络连接。
- dns：DNS 模块，用于解析域名和 IP 地址。
- tls：TLS/SSL 模块，用于创建和处理 TLS/SSL 连接。
- child_process：子进程模块，用于创建和管理子进程。
- cluster：集群模块，用于创建和管理集群。
- module：模块加载器模块，用于加载和管理模块。
- querystring：查询字符串模块，用于解析和生成 URL 查询字符串。

# express 模块

- express 是一个流行的 Node.js 框架，它提供了一系列的功能来简化 Web 开发。其中，express 模块是一个核心模块，用于创建和配置 Express 应用。
- nodemon：nodemon 是一个用于监视和重启 Node.js 应用的工具。它可以自动检测应用代码的更改，并在代码更改后自动重启应用，从而提高开发效率。
- body-parser：用于解析 HTTP 请求体，特别是处理 POST 请求的数据。可以解析 JSON、URL 编码等格式的数据。
- cors：用于处理跨域请求。可以通过设置 Access-Control-Allow-Origin 头来允许跨域请求。

# 使用 worker thread 与 child_process 相比有什么优势？

child_process 用自己的内存空间运行自己的进程，而 worker thread 则是一个进程中的线程，可以与主线程共享内存，这有助于避免来回昂贵的数据序列化。
通过 HTTP 与客户端建立双向实时连接
我们可以使用 WebSockets 或者长轮询，有像http://soket.io和SignalR这样的库可以为我们简化这个过程。如果WebSockets在浏览器中不可用，它们甚至可以为客户端提供长时间的轮询功能。
