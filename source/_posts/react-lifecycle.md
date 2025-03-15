---
title: react-lifecycle

date: 2024-01-22 15:23:43
tags: react
---

constructor：不是 react 的方法，是 es6 的方法,可以在 react 中使用，调用当前组件的父级，也可以在函数中设置当前组件所需要的状态 state；

# 挂载阶段

- constructor()：在组件实例被创建和初始化的时候调用。通常用于初始化 state 和绑定事件处理函数。
- static getDerivedStateFromProps(nextProps, prevState)：在组件实例被创建以及接收到新的 props 时被调用。用于根据 props 来更新 state。
- render()：用于渲染 UI，必须是一个纯函数，不能有任何副作用操作。
- componentDidMount()：在组件挂载到 DOM 后立即调用。适合在这个阶段进行网络请求、订阅事件等操作。

# 更新阶段

- static getDerivedStateFromProps(nextProps, prevState)：当组件接收到新的 props 或者 state 发生变化时调用，用于根据新的 props 更新 state。
- shouldComponentUpdate(nextProps, nextState)：在接收到新的 props 或 state 时被调用。可以通过返回 false 来阻止组件的重新渲染，以优化性能。
- render()：在组件需要更新时被调用以重新渲染 UI。
- getSnapshotBeforeUpdate(prevProps, prevState)：在更新发生之前被调用，返回值会作为 componentDidUpdate 的第三个参数。
  -componentDidUpdate(prevProps, prevState, snapshot)：在组件更新后被调用，可以在这里进行 DOM 操作或者数据更新后的处理。

# 卸载阶段

componentWillUnmount()：在组件即将被卸载和销毁时调用。适合在这里进行清理操作，如取消网络请求、取消定时器等。
