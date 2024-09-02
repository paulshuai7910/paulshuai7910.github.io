---
title: React Fiber架构的原理和工作模式
date: 2024-09-02 09:32:09
tags:
---

# What's Fiber？

React 的基本组成：当我们写 React 组件并使用 JSX 时，React 在底层会将 JSX 转换为元素的对象结构。例如：

```js
const element = <div>Hello, world!</div>
```

React 会将其转换为以下对象结构：

```js
const element = React.createElement("h1", null, "Hello, world")
```

为了将这个元素渲染到 DOM 上，React 需要创建一种内部实例，用来追踪该组件的所有信息和状态。在早期版本的 React 中，我们称之为“实例”或“虚拟 DOM 对象”。但在 Fiber 架构中，这个新的工作单元就叫做 Fiber。
所以，在本质上，Fiber 是一个 JavaScript 对象，代表 React 的一个工作单元，它包含了与组件相关的信息。一个简化的 Fiber 对象长这样：

```js
const fiber = {
  type: "div",
  props: {
    children: "Hello, world",
  },
  child: null,
  sibling: null,
  return: null,
  stateNode: null,
  alternate: null,
  pendingProps: null,
  memoizedProps: null,
  memoizedState: null,
  dependencies: null,
}
```

当 React 开始工作时，它会沿着 Fiber 树形结构进行，试图完成每个 Fiber 的工作（例如，比较新旧 props，确定是否需要更新组件等）。如果主线程有更重要的工作（例如，响应用户输入），则 React 可以中断当前工作并返回执行主线程上的任务。

因此，Fiber 不仅仅是代表组件的一个内部对象，它还是 React 的调度和更新机制的核心组成部分。

# Why do we need Fiber？

在 React 16 之前的版本中，是使用递归的方式处理组件树更新，称为堆栈调和（Stack Reconciliation），这种方法一旦开始就不能中断，直到整个组件树都被遍历完。这种机制在处理大量数据或复杂视图时可能导致主线程被阻塞，从而使应用无法及时响应用户的输入或其他高优先级任务。

Fiber 的引入改变了这一情况。Fiber 可以理解为是 React 自定义的一个带有链接关系的 DOM 树，每个 Fiber 都代表了一个工作单元，React 可以在处理任何 Fiber 之前判断是否有足够的时间完成该工作，并在必要时中断和恢复工作。

# Fiber structure

源码里 FiberNode 的结构：

```js
function FiberNode(
  this: $FlowFixMe,
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode
) {
  // 基本属性
  this.tag = tag // 描述此Fiber的启动模式的值（LegacyRoot = 0; ConcurrentRoot = 1）
  this.key = key // React key
  this.elementType = null // 描述React元素的类型。例如，对于JSX<App />，elementType是App
  this.type = null // 组件类型
  this.stateNode = null // 对于类组件，这是类的实例；对于DOM元素，它是对应的DOM节点。

  // Fiber链接
  this.return = null // 指向父Fiber
  this.child = null // 指向第一个子Fiber
  this.sibling = null // 指向其兄弟Fiber
  this.index = 0 // 子Fiber中的索引位置

  this.ref = null // 如果组件上有ref属性，则该属性指向它
  this.refCleanup = null // 如果组件上的ref属性在更新中被删除或更改，此字段会用于追踪需要清理的旧ref

  // Props & State
  this.pendingProps = pendingProps // 正在等待处理的新props
  this.memoizedProps = null // 上一次渲染时的props
  this.updateQueue = null // 一个队列，包含了该Fiber上的状态更新和副作用
  this.memoizedState = null // 上一次渲染时的state
  this.dependencies = null // 该Fiber订阅的上下文或其他资源的描述

  // 工作模式
  this.mode = mode // 描述Fiber工作模式的标志（例如Concurrent模式、Blocking模式等）。

  // Effects
  this.flags = NoFlags // 描述该Fiber发生的副作用的标志（十六进制的标识）
  this.subtreeFlags = NoFlags // 描述该Fiber子树中发生的副作用的标志（十六进制的标识）
  this.deletions = null // 在commit阶段要删除的子Fiber数组

  this.lanes = NoLanes // 与React的并发模式有关的调度概念。
  this.childLanes = NoLanes // 与React的并发模式有关的调度概念。

  this.alternate = null // Current Tree和Work-in-progress (WIP) Tree的互相指向对方tree里的对应单元

  // 如果启用了性能分析
  if (enableProfilerTimer) {
    // ……
  }

  // 开发模式中
  if (__DEV__) {
    // ……
  }
}
```

其实可以理解为是一个更强大的虚拟 DOM。

# Fiber 工作原理

Fiber 工作原理中最核心的点就是：可以中断和恢复，这个特性增强了 React 的并发性和响应性。

实现可中断和恢复的原因就在于：Fiber 的数据结构里提供的信息让 React 可以追踪工作进度、管理调度和同步更新到 DOM

Fiber 工作原理中的几个关键点：

- 单元工作：每个 Fiber 节点代表一个单元，所有 Fiber 节点共同组成一个 Fiber 链表树（有链接属性，同时又有树的结构），这种结构让 React 可以细粒度控制节点的行为。
- 链接属性：child、sibling 和 return 字段构成了 Fiber 之间的链接关系，使 React 能够遍历组件树并知道从哪里开始、继续或停止工作。
  ![fiber work](../../assets/img/fiber-link.webp "fiber")
