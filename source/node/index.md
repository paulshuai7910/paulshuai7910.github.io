---
title: Nodejs 启动过程
date: 2024-09-04 22:23:59
tags:
---

**Nodejs 是一个开源和跨平台的 js 运行时环境！！！**

![http cache](node/index/iopoll.jpg)

js 代码<-->V8 引擎<-->Nodejs Bindings(node api)<---->Libuv

nodejs 启动--> 注册 C++模块 ---> 初始化模块加载器 ---> 初始化 V8 ---> 初始化 libuv ---> 初始化 libuv 事件循环 ---> 启动事件循环 ---> 启动 V8 ---> 启动 libuv ---> 启动 libuv 事件循环 ---> 启动事件循环 ---> 启动 V8 ---> 启动 libuv ---> 启动 libuv 事件循环 --->启动事件循环 ---> 启动 V8 ---> 启动 libuv ---> 启动 libuv 事件循环 --->

# 一、注册 C++模块

首先 Node.js 会调用 registerBuiltinModules 函数注册 C++模块，这个函数会调用一系列 registerxxx 的函数
Node.js-->-->registerModule

# 二、创建 Environment 对象，并绑定到 Context

注册完 C++模块后就开始创建 Environment 对象，Environment 是 Node.js 执行时的环境对象，类似一个全局变量的作用，他记录了 Node.js 在运行时的一些公共数据。创建完 Environment 后，Node.js 会把该对象绑定到 V8 的 Context 中，为什么要这样做呢？主要是为了在 V8 的执行上下文里拿到 env 对象，因为 V8 中只有 Isolate、Context 这些对象。如果我们想在 V8 的执行环境中获取 Environment 对象的内容，就可以通过 Context 获取 Environment 对象。
Node.js-->-->registerModule-->-->createEnvironment-->-->createContext

# 三、初始化模块加载器

1. Node.js 首先传入 c++模块加载器，执行 loader.js，loader.js 主要是封装了 c++模块加载器和原生 js 模块加载器。并保存到 env 对象中。
2. 接着传入 c++和原生 js 模块加载器，执行 run_main_module.js。
3. 在 run_main_module.js 中传入 js 和原生 js 模块加载器，执行用户的 js。
   假设用户 js 如下

```js
require("net")
require("./myModule")
```

分别加载了一个用户模块和原生 js 模块，我们看看加载过程，执行 require 的时候。

1. Node.js 首先会判断是否是原生 js 模块，如果不是则直接加载用户模块，否则，会使用原生模块加载器加载原生 js 模块。
2. 加载原生 js 模块的时候，如果用到了 c++模块，则使用 internalBinding 去加载。

# 四、执行用户 JS 代码，然后进入 Libuv 事件循环

接着 Node.js 就会执行用户的 js，通常用户的 js 会给事件循环生产任务，然后就进入了事件循环系统，比如我们 listen 一个服务器的时候，就会在事件循环中新建一个 tcp handle。Node.js 就会在这个事件循环中一直运行。

```js
net.createServer(() => {}).listen(80)
```

# 五、事件循环

主模块代码执行完毕后，Node.js 不会立即退出。相反，它会进入事件循环，以等待和处理任何异步操作。事件循环是 Node.js 处理非阻塞 I/O 操作的核心机制。
事件循环主要分为 7 个阶段。

- timer 阶段主要是处理定时器相关的任务，
- pending, I/O Callbacks 阶段主要是处理 Poll I/O 阶段回调里产生的回调。
- check、prepare、idle 阶段是自定义的阶段，这三个阶段的任务每次事件序循环都会被执行。 - Poll I/O 阶段主要是处理网络 I?O、信号、线程池等等任务。
- Closing 阶段主要是处理关闭的 Handle，比如停止关闭服务器。
  ![alt text](nodejs/v2-5c5d17777ad3b75ca07be71bef5bb305_b.jpg)

1. Timer 阶段: 执行 setTimeout 和 setInterval 回调。用二叉堆实现，最快过期的在根节点。
2. Pending Callbacks 阶段：执行一些延迟到下一轮的 I/O 回调
3. Check、prepare、idle 阶段：每次事件循环都会被执行。内部操作，处理一些系统准备工作。
4. Poll I/O 阶段：处理新的 I/O 事件，执行 I/O 相关的回调函数。
5. Check Handles 阶段：执行 setImmediate 的回调。
6. Close Callbacks 阶段：处理 close 事件的回调，如 socket.on('close', ...)。

# 定时器阶段

定时器的底层数据结构是二叉堆，最快到期的节点在最上面。在定时器阶段的时候，就会逐个节点遍历，如果节点超时了，那么就执行他的回调，如果没有超时，那么后面的节点也不用判断了，因为当前节点是最快过期的，如果他都没有过期，说明其他节点也没有过期。节点的回调被执行后，就会被删除，为了支持 setInterval 的场景，如果设置 repeat 标记，那么这个节点会被重新插入到二叉堆。

## check、idle、prepare 阶段

check、idle、prepare 阶段相对比较简单，每个阶段维护一个队列，然后在处理对应阶段的时候，执行队列中每个节点的回调，不过这三个阶段比较特殊的是，队列中的节点被执行后不会被删除，而是虎一直在队列里，除非显式删除。

## pending、closing 阶段

pending 阶段：在 poll io 回调里产生的回调。 closing 阶段：执行关闭 handle 的回调。 pending 和 closing 阶段也是维护了一个队列，然后在对应阶段的时候执行每个节点的回调，最后删除对应的节点。

## Poll io 阶段

Poll io 阶段是最重要和复杂的一个阶段，下面我们看一下实现。首先我们看一下 poll io 阶段核心的数据结构：io 观察者。io 观察者是对文件描述符、感兴趣事件和回调的封装。主要是用在 epoll 中。

# 进程和进程间通信

## 创建进程

Node.js 中的进程是使用 fork+exec 模式创建的，fork 就是复制主进程的数据，exec 是加载新的程序执行。Node.js 提供了异步和同步创建进程两种模式。

## 进程间通信

接下来我们看一下父子进程间怎么通信呢？在操作系统中，进程间的虚拟地址是独立的，所以没有办法基于进程内存直接通信，这时候需要借助内核提供的内存。进程间通信的方式有很多种，管道、信号、共享内存等等。

# 线程和线程间通信

线程架构
Node.js 是单线程的，为了方便用户处理耗时的操作，Node.js 在支持多进程之后，又支持了多线程。Node.js 中多线程的架构如下图所示。每个子线程本质上是一个独立的事件循环，但是所有的线程会共享底层的 Libuv 线程池。

# Libuv 线程池

为什么需要使用线程池？文件 IO、DNS、CPU 密集型不适合在 Node.js 主线程处理，需要把这些任务放到子线程处理。
1 Libuv 内部维护了一个异步通信的队列，需要异步通信的时候，就往里面插入一个 async 节点
2 同时 Libuv 还维护了一个异步通信相关的 io 观察者
3 当有异步任务完成的时候，就会设置对应 async 节点的 pending 字段为 1，说明任务完成了。并且通知主线程。
4 主线程在 poll io 阶段就会执行处理异步通信的回调，在回调里会执行 pending 为 1 的节点的回调。
下面我们来看一下线程池的实现。
1 线程池维护了一个待处理任务队列，多个线程互斥地从队列中摘下任务进行处理。
2 当给线程池提交一个任务的时候，就是往这个队列里插入一个节点。
3 当子线程处理完任务后，就会把这个任务插入到事件循环本身维护到一个已完成任务队列中，并且通过异步通信的机制通知主线程。
4 主线程在 poll io 阶段就会执行任务对应的回调。

[参考来源](https://zhuanlan.zhihu.com/p/375276722,"nodejs")
