---
title: EventLoop 事件循环
date: 2024-06-03 12:39:13
tags: eventloop nodejs
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

### **浏览器事件循环（Event Loop）**

**事件循环**是 JavaScript 运行时的一个核心机制，主要用于处理异步任务。浏览器中的 JavaScript 是单线程执行的，这意味着同一时刻只有一个任务在执行，但通过事件循环，浏览器能够高效地处理异步操作（如用户交互、定时器、I/O 操作等）并保持响应性。

**事件循环**的工作原理是协调同步代码和异步代码（例如：`setTimeout`、`Promise`、`XHR` 等）在队列中的执行顺序。

---

### **事件循环的执行流程**

浏览器的事件循环由以下几个关键部分组成：

1. **调用栈（Call Stack）**：

   - 调用栈用于存储当前正在执行的所有函数调用。
   - JavaScript 的代码按顺序从上到下执行。当遇到函数调用时，该函数会被压入栈中执行，执行完后会从栈中弹出。
   - 这是 JavaScript 的同步执行模型。

2. **任务队列（Task Queue）/消息队列（Message Queue）**：

   - 当一个异步操作（如 `setTimeout`、`Promise`）完成时，相应的回调函数会被推送到任务队列中。
   - 任务队列存放着待执行的异步任务，它们将在调用栈空闲时依次执行。

3. **事件循环（Event Loop）**：

   - 事件循环的作用是不断检查调用栈是否为空。
   - 如果调用栈为空，事件循环就会将任务队列中的一个任务（也叫消息）取出来并执行。
   - 任务队列中的任务必须等待调用栈清空后才能执行。

4. **Web APIs**：
   - 浏览器环境提供的 Web APIs 如 `setTimeout`、`fetch`、`DOM` 事件等，执行异步操作时，相关回调会在 Web APIs 中运行，然后将回调函数添加到任务队列中。

---

### **事件循环的执行顺序**

浏览器的事件循环通常按以下顺序执行：

1. **执行同步代码**：

   - 首先，浏览器会执行所有的同步代码。所有的同步任务都会被推入调用栈中执行，直到调用栈为空。

2. **执行微任务队列（Microtasks）**：

   - 微任务（例如：`Promise.then`、`MutationObserver` 等）会在调用栈清空后立即执行，并且会优先于任务队列中的其他任务执行。
   - 微任务队列在任务队列之前执行，这意味着如果有多个微任务，它们会按顺序依次执行，直到微任务队列清空。

3. **执行任务队列中的任务（宏任务）**：

   - 微任务执行完毕后，事件循环会从任务队列中取出一个任务并执行。
   - 宏任务（例如：`setTimeout`、`setInterval`、`DOM` 事件等）会按顺序执行。

4. **渲染（Repaint & Reflow）**：
   - 在任务执行后，浏览器会检查页面是否需要重新渲染（例如，页面布局或样式变化）。
   - 但浏览器并不会在每个事件循环的每个周期都执行渲染，它会根据需求进行渲染操作。

---

### **微任务与宏任务**

在事件循环中，我们通常会听到 **微任务** 和 **宏任务** 的概念，它们是浏览器如何处理任务队列的不同方式。

- **微任务（Microtasks）**：

  - 微任务具有更高的优先级，在每次执行完当前执行栈中的代码后，事件循环会立即执行所有待处理的微任务。
  - **典型的微任务**：`Promise.then`、`Promise.catch`、`MutationObserver`。

- **宏任务（Macrotasks）**：
  - 宏任务包括 `setTimeout`、`setInterval`、I/O 操作、DOM 事件等。它们会在微任务执行完后再执行。
  - **典型的宏任务**：`setTimeout`、`setInterval`、UI 渲染、`click` 事件等。

#### **微任务和宏任务的执行顺序**：

- 每次执行一个宏任务（例如 `setTimeout`），事件循环会检查并执行所有排队的微任务（`Promise`）。
- 这样微任务总是会在宏任务之前执行。

---

### **示例：**

```javascript
console.log("1") // 同步任务
setTimeout(() => {
  console.log("2") // 宏任务
}, 0)
Promise.resolve().then(() => {
  console.log("3") // 微任务
})
console.log("4") // 同步任务
```

**执行顺序**：

1. **同步任务**：
   - `console.log('1')` → 打印 1
   - `console.log('4')` → 打印 4
2. **微任务**：
   - `Promise.resolve().then()` → 打印 3
3. **宏任务**：
   - `setTimeout()` → 打印 2

**输出顺序**：

```
1
4
3
2
```

### **总结**

1. **同步代码**先执行，逐个压入调用栈并按顺序执行。
2. **微任务**在同步代码执行完后立即执行，优先于宏任务。
3. **宏任务**（例如 `setTimeout`）排队执行，等待微任务队列清空。
4. 在每个事件循环周期结束时，浏览器会进行渲染操作（如重排和重绘）。

事件循环是 JavaScript 异步执行的关键，它让 JavaScript 在单线程的情况下能够处理大量的异步任务，保证了页面的响应性和流畅性。

# Node 中的 Event Loop

### **Node.js 的事件循环（Event Loop）**

Node.js 的事件循环是 Node.js 中的核心概念之一，它是 Node.js 能够处理高并发、非阻塞 I/O 操作的基础。与浏览器的事件循环类似，Node.js 的事件循环允许 Node.js 在单线程上执行异步任务，而不会阻塞程序的执行。

### **Node.js 事件循环的工作原理**

Node.js 的事件循环机制基于 **libuv** 库，它提供了异步 I/O 操作的支持。在 Node.js 中，事件循环的任务是从队列中拉取任务并依次执行。这个过程会不断地循环，确保 Node.js 能够高效地处理 I/O 操作（如网络请求、文件系统操作、定时器等）。

### **Node.js 事件循环的生命周期**

Node.js 的事件循环分为多个阶段，每个阶段都有特定的任务队列。事件循环会在每个阶段中依次执行这些任务。下面是 Node.js 事件循环的详细阶段（从 Node.js v12+ 起，阶段和顺序保持一致）：

#### 1. **Timers（定时器）**

- 这个阶段执行所有准备好的 `setTimeout` 和 `setInterval` 的回调函数。定时器的回调函数在其指定的时间之后执行。
- **注意**：`setTimeout` 和 `setInterval` 的回调并不保证在指定的时间点立即执行，它们的回调会在事件循环的下一轮执行时被调用。

#### 2. **I/O callbacks（I/O 回调）**

- 在这个阶段，Node.js 会处理大部分 I/O 操作的回调，例如文件操作、网络请求等。比如读取文件、数据库操作等。
- 该阶段的回调函数会尽可能快地执行，以便尽早处理 I/O 请求。

#### 3. **idle, prepare（空闲准备阶段）**

- 这个阶段主要是为了执行内部的系统操作，通常应用程序不会在此阶段做任何事情。Node.js 主要会做一些准备工作，以确保事件循环的顺利进行。

#### 4. **Poll（轮询）**

- 这是事件循环的核心阶段。Node.js 会检查是否有待处理的事件或回调（例如 `setTimeout`、`setImmediate` 等）。
- 如果队列中有回调，Node.js 会处理它们。这个阶段可以阻塞直到某个 I/O 操作完成，或者任务队列为空。
- 如果没有事件需要处理，Node.js 会在此阶段等待新的 I/O 事件。

#### 5. **Check（检查阶段）**

- 在 `poll` 阶段结束后，`setImmediate` 的回调会在这个阶段执行。`setImmediate` 用于在事件循环的下一轮中立即执行回调。

#### 6. **Close callbacks（关闭回调）**

- 在这个阶段，Node.js 会执行一些清理工作，如关闭事件监听器、清理定时器等。

---

### **Node.js 事件循环的详细流程**

1. **执行同步代码**：

   - 事件循环首先会执行程序中所有的同步代码（如函数调用），这些同步任务会在调用栈中顺序执行。

2. **定时器回调执行**（`setTimeout` / `setInterval`）：

   - 如果有定时器已经到期，`setTimeout` 和 `setInterval` 的回调会在定时器阶段执行。

3. **执行 I/O 回调**：

   - 在 I/O 回调阶段，Node.js 会执行一些基于 I/O 操作（如文件操作、网络请求等）的回调函数。

4. **轮询阶段**：

   - 在轮询阶段，Node.js 会检查是否有新的事件需要处理（例如 I/O 事件、定时器事件等）。如果没有事件发生，Node.js 会进入等待状态。

5. **立即执行回调**（`setImmediate`）：

   - 如果在 `poll` 阶段没有更多事件处理，Node.js 会进入 `check` 阶段并执行通过 `setImmediate` 设置的回调。

6. **关闭回调**：
   - 如果有未处理的关闭回调（如 WebSocket、TCP 连接关闭等），这些回调将在此阶段执行。

---

### **事件循环的示例：**

```javascript
const fs = require("fs")

// 1. 执行同步代码
console.log("start")

// 2. setTimeout 在 Timers 阶段执行
setTimeout(() => {
  console.log("setTimeout")
}, 0)

// 3. I/O 回调
fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) throw err
  console.log("I/O callback")
})

// 4. setImmediate 在 Check 阶段执行
setImmediate(() => {
  console.log("setImmediate")
})

console.log("end")
```

**输出顺序**：

```
start
end
setImmediate
I/O callback
setTimeout
```

**解释**：

1. `start` 和 `end` 是同步代码，按顺序执行。
2. `setTimeout` 设置的是一个定时器，它的回调会在 `Timers` 阶段执行。
3. `setImmediate` 设置的是一个立即执行回调，它会在 `Check` 阶段执行。
4. `fs.readFile` 是异步 I/O 操作，它的回调会在 `I/O callbacks` 阶段执行。

### **微任务与宏任务**

Node.js 和浏览器类似，也有微任务和宏任务的区别：

- **宏任务**：包含 `setTimeout`、`setInterval`、I/O 操作回调等。
- **微任务**：主要是 Promise 的回调（`then`、`catch`），`process.nextTick` 等。

#### **微任务与宏任务的执行顺序**：

1. 执行所有同步代码。
2. 执行微任务队列中的所有任务（例如 `Promise.then()`）。
3. 执行宏任务队列中的一个任务（例如 `setTimeout`）。
4. 在执行宏任务后，会再次检查微任务队列是否有待执行的任务，如果有，继续执行所有微任务。

---

### **总结：**

- **Node.js 事件循环**是单线程的，它通过异步 I/O 操作和回调队列的机制，实现了非阻塞的高效 I/O 处理。
- Node.js 的事件循环通过多个阶段处理任务，每个阶段有不同类型的回调（同步、异步等）待执行。
- 事件循环的主要优点是能使 Node.js 在单线程中高效处理并发任务，尤其适用于 I/O 密集型任务。

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
