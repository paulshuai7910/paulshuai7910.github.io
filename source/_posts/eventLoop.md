---
title: EventLoop 事件循环
date: 2024-06-03 12:39:13
tags:
---

# 进程和线程的定义：

[EventLoop](https://segmentfault.com/a/1190000015832962 "eventLoop")。
进程是 cpu 资源分配的最小单位（是能拥有资源和独立运行的最小单位）
线程是 cpu 调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）
补充：

我们所说的单线程和多线程，是指一个进程内是单一线程还是多线程。
进程间的通信方式包括： 管道 pipe、 命名管道 FIFO、消息队列 MessageQueue、共享存储 SharedMemory、信号量 Semaphore、套接字 Socket、信号。

# 浏览器是多进程

- 浏览器是多进程的。
- 浏览器之所以能够运行，是因为系统给它的进程分配了资源（cpu、内存）。
- 简单点理解，每打开一个 Tab 页，就相当于创建了一个独立的浏览器进程。

## 渲染进程显然是多线程的，它主要包括以下 5 个常驻线程：

- GUI 渲染线程，负责渲染浏览器界面，解析 HTML，CSS，构建 DOM 树和 RenderObject 树，布局和绘制等。
- JS 引擎线程，也称为 JS 内核，负责处理 Javascript 脚本程序，（例如 V8 引擎）。
- 事件触发线程，用来控制事件循环（可以理解为，JS 引擎线程自己都忙不过来，需要浏览器另开线程协助）。
- 定时触发器线程，浏览器定时计数器并不是由 JavaScript 引擎计数的,（因为 JavaScript 引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确），JS 中常用的 setInterval 和 setTimeout 就归这个线程管理。
- 异步 http 请求线程，也就是 ajax 发出 http 请求后，接收响应、检测状态变更等都是这个线程管理的。

我们常说的 JavaScript 是单线程的，其实就是说的 JS 引擎是单线程的，它仅仅是浏览器渲染进程种的一个线程。为什么呢？因为 JavaScript 的主要作用是与用户互动，以及操作 DOM，如果 JavaScript 有两个线程，一个线程对一个 DOM 节点执行 A 操作，另一个线程这个 DOM 节点执行 B 操作，那么就会起冲突，所以 JavaScript 在前端的应用就注定了它是单线程的。

然而 JavaScript 的单线程特性就注定我们不用它去完成密集的 cpu 运算，因为密集 cpu 运算耗时过长，阻塞页面渲染。为了解决这个问题，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不得操作 DOM。

# 浏览器中的 Event Loop

# Node 中的 Event Loop

# Difference between Event Loop and Promise

Node.js 与浏览器中的 Event Loop 存在多个显著的不同点，这些差异主要源于它们各自的设计目标、运行环境以及处理任务的方式。以下是对这些不同点的详细分析：

1. 执行环境与任务类型
   Node.js：Node.js 的 Event Loop 运行在 Node.js 环境中，它主要处理 I/O 操作（如文件读写、网络请求等）和服务器端的逻辑处理。Node.js 的 Event Loop 允许它在不创建额外线程的情况下处理并发操作，提高了 I/O 密集型任务的效率。
   浏览器：浏览器的 Event Loop 运行在浏览器环境中，负责处理浏览器事件（如点击、滚动等）、用户交互、页面渲染等。它确保页面的响应性和动画的流畅性。
2. 宏任务与微任务
   宏任务（Macro Task）：在 Node.js 和浏览器中，宏任务都是指那些需要较长时间运行的任务，如 I/O 操作、定时器的回调等。然而，在 Node.js 中，宏任务被细分为多种类型，如 Timers（定时器）、I/O Polling（I/O 轮询）、Check（检查，用于 setImmediate 回调）、Close Callbacks（关闭回调）等，每种类型都有其特定的执行顺序和优先级。而在浏览器中，宏任务主要包括整体脚本（script）、setTimeout、setInterval、I/O 操作、UI 渲染等。
   微任务（Micro Task）：微任务是指那些需要快速执行的任务，它们会在当前宏任务执行完毕后立即执行。在 Node.js 中，微任务主要包括 Promise.then()和 process.nextTick()（注意，process.nextTick()的优先级高于 Promise.then()）。而在浏览器中，微任务主要包括 Promise.then()、MutationObserver 和 queueMicrotask 等。
3. 执行顺序与优先级
   Node.js：Node.js 的 Event Loop 在执行顺序上更加复杂，因为它有多种类型的宏任务队列。Node.js 会按照特定顺序（如先 Timers，再 I/O Polling，然后是 Check 等）处理宏任务，并在每次宏任务执行完毕后清空微任务队列。如果有多个微任务同时满足执行条件，Node.js 会按照优先级（如 process.nextTick()的优先级高于 Promise.then()）来执行它们。
   浏览器：浏览器的 Event Loop 则相对简单，它会在每次执行完一个宏任务后清空微任务队列。如果有多个微任务同时满足执行条件，浏览器会按照它们被添加到队列中的顺序（先进先出）来执行它们。
4. 并发处理机制
   Node.js：Node.js 的 Event Loop 是基于 Libuv 库实现的，它利用了底层操作系统提供的多线程特性来优化 I/O 操作和网络请求等任务的并发处理。这使得 Node.js 能够处理更高的并发请求。
   浏览器：浏览器的 Event Loop 则是单线程的，它通过异步回调函数和事件触发来实现非阻塞的异步操作。虽然浏览器的 JavaScript 引擎（如 V8）可能会使用多线程来优化某些操作（如垃圾回收），但事件循环本身是单线程的。
   综上所述，Node.js 与浏览器中的 Event Loop 在执行环境、任务类型、执行顺序与优先级以及并发处理机制等方面都存在显著差异。了解这些差异有助于开发者更好地理解和优化各自的异步操作。
