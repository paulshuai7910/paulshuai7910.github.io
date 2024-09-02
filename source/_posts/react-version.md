---
title: react-version info
date: 2024-04-13 11:34:12
tags:
---

# react 16

引入 Fiber 架构
异步渲染的支持
支持错误边界，更好地处理组件内的错误。
render 方法支持返回数组和字符串
Portals：将元素渲染到组件之外的 dom 节点
自定义 DOM 属性
服务端渲染优化

# React 16.3

新增 getDerivedStateFromProps 生命周期方法，用于派生状态

# react16.8

react hooks,建议启用 eslint-plugin-react-hooks 这个 ESLint 插件来强制执行 Hooks 的最佳实践
react devtool 支持 Hooks
typescript 和 Flow 的支持
性能优化
ReactTestUtils.act

还添加了一个名为 ReactTestUtils.act()的 API，它可以确保测试中的行为与在浏览器中的行为更加接近。这个 API 主要用于处理异步操作和副作用的测试。

# react 17

- 渐进式更新
- 新的 API 和特性
- 时间系统改进

# React 18

### 并发渲染器

- transitions：引入 transitions 概念用于区分 urgent 和 non-urgent 的更新，有助于开发者更好的控制 UI 渲染的优先级，减少卡顿
- startTransition API： 新的 API 允许开发者将某些更新状态标记为非紧急（non-urgent），这些个更新会在当前屏幕帧渲染完成后进行异步
- Suspense 扩展
- 流式服务器端渲染
- 更细颗粒度的可选择性
- 自动批处理
- react 18 引入了自动批处理功能，这可以自动的将多个状态合并为一个更新批次，从而提高渲染性能，在 react17 以及以前的版本中，需要手动使用 unstable_bantchUpdates 来实现类似新功能

## new hooks

useId:
useTransition:
useDeferredValue:
useSyncExternalStore
useInsertionEffect:
ReactDom API 的更新:
client\server 的分离
废弃旧的 API
stric modal 的增强
其他改进：
一致的 uesEffect 计时
更严格的补水错误

# 基础信息

## react 节点类型

- Boolean (实际渲染会被忽略)
- null or undefined (实际渲染会被忽略)
- Number
- String
- React 元素 (JSX 语句的返回值)
- 由以上类型数据组成的数组（还可以嵌套）

## How to upgrade react 18

install
npm install react@18 react-dom@18
升级后代码不用做任何更改，不影响项目正常启动和运行
在 legacy 模式下，会有 console warning 旧的 api 已经弃用
用户更新更加丝滑，不会出现 upgrade crash 的情况。方便用户做 ab test。
引入：

```js
// Before
import { render } from "react-dom"
const container = document.getElementById("app")
render(<App tab="home" />, container)
```

```js
// After
import { createRoot } from "react-dom/client"
const container = document.getElementById("app")
const root = createRoot(container) // createRoot(container!) if you use TypeScript
root.render(<App tab="home" />)
```

弃用 API：

```js
// Before
unmountComponentAtNode(container);
// After
root.unmount();
callback render：
// Before
const container = document.getElementById('app');
render(<App tab="home" />, container, () => {
  console.log('rendered');
});
// After
function AppWithCallbackAfterRender() {
  useEffect(() => {
    console.log('rendered');
  });
  return <App tab="home" />
}
const container = document.getElementById('app');
const root = createRoot(container);
root.render(<AppWithCallbackAfterRender />);
```

- hydration
  server-side rendering with hydration,
  ……
- Updates to server rendering apis
  ……
- typescript
  Update to typescript definitions
  update @types/react and @types/react-dom dependencies to the latest versions.

### Automatic batching

Before:除了在 react event handler 中 其他所有在 promise、settimeout 中产生的 state update 都是不能自动批量更新。也就是在这些事件中没调用一次 state 就会触发一次渲染，新版本支持更多地方批量更新：

```js
react18：React event hander\Promise\setTimeout\native event handler (or any other event are batched.--form react)
// Before React 18 only React events were batched
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will render twice, once for each state update (no batching)
}, 1000);
// -------------------------------
// After React 18 updates inside of timeouts, promises,
// native event handlers or any other event are batched.
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

备注：automatic batching 默认开启，但是可以使用 flushsync

```js
import { flushSync } from "react-dom"
function handleClick() {
  flushSync(() => {
    setCounter((c) => c + 1)
  })
  // React has updated the DOM by now
  flushSync(() => {
    setFlag((f) => !f)
  })
  // React has updated the DOM by now
}
```

### New apis for Library

- useSyncExternalStore
  ...
- useInsertionEffect
  Concurrent mode
- startTransition
  Concurrent mode
- useDeferredValue
  ...
- useId
  useId is a hook for generating unique IDs that are stable across the server and client, while avoiding hydration mismatches

```js
const id = useId()
// --------
function Checkbox() {
  const id = useId()
  return (
    <>
      <label htmlFor={id}>Do you like React?</label>
      <input id={id} type="checkbox" name="react" />
    </>
  )
}
//  -----For multiple IDs in same component, append a suffix using the same id
function NameFields() {
  const id = useId()
  return (
    <div>
      <label htmlFor={id + "-firstName"}>First Name</label>
      <div>
        <input id={id + "-firstName"} type="text" />
      </div>
      <label htmlFor={id + "-lastName"}>Last Name</label>
      <div>
        <input id={id + "-lastName"} type="text" />
      </div>
    </div>
  )
}
```

## Concurrent mode

不属于新功能，是一种新的底层机制,成为众多新功能的基础。
Before：线型渲染，一个接着一个被触发，只要触发就无法终止。
After：本质上的变化，并发渲染，所有渲染都是可中断、继续、终止
可以在后台进行
可以有优先级（开发者可以决定渲染顺序）
Trasition（新功能）
控制渲染优先级（分为高低两种优先级），useTransition\starttransition
默认都是高优先级（正常的 state 更新）
非 hooks 场景下你用 starttransition，使用该 API 进行包裹
Updates wrapped in startTransition are handled as non-urgent and will be interrupted if more urgent updates like clicks or key presses come in. If a transition gets interrupted by the user (for example, by typing multiple characters in a row), React will throw out the stale rendering work that wasn’t finished and render only the latest update.

```js
import { stratTransition, useTransition } from "react"

// urgent: show what was type
setInputValue(input)
// Mark any state updates inside as transition.
startTransitiono(() => {
  // Transition: Show the results
  setSearchQuery(input)
})
// hooks
```

useTransition: a hook to start transitions, including a value to track the pending state.
startTransition: a method to start transitions when the hook cannot be used.

### Suspense

fetch on render

```js
// component1
useEffect(() => {
  fetchUser().then((u) => {
    setUser(u)
  })
})
```

fetch then reder

```js
// fetch then render
// 使用 promise.all() 进行解决，但是单独一个请求完成不能直接渲染，必须要等两个都请求结束才能 rendering
// 称为 waterfall
```

render as your fetch

```js
import React, { Suspense } from "react"
// solution：render as you fetch
// fallback :传入获取数据之前的 loading 组件
;<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

原理，在 Suspense 包裹的组件内部，如果 react 得到一个被 throw 出来的 promise，react 会进行 catch，去找到最近的 suspense 组件，这样 suspense 组件就只要他要等待 promise 完成，接着就去渲染 fallback 的内容。在 promise resolve 之后，suspense 就会渲染内部组件，所以为了使用 suspense 方法，data query 需要符合某种规定
需要网络请求成去适配 react Suspense 这种约定
现阶段 swr 支持该 Suspense

### Updates to strict mode

```js
//strictMode
import React from "react"
function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  )
}
```

// Before

- React mounts the component.
  _ Layout effects are created.
  _ Effect effects are created.
  // After:
- React mounts the component.
  - Layout effects are created.
  - Effect effects are created.
- React simulates unmounting the component.
  - Layout effects are destroyed.
  - Effects are destroyed.
- React simulates mounting the component with the previous state.
  _ Layout effect setup code runs
  _ Effect setup code runs
  configuring you Testing Environment
  before running test:
  // In your test setup file
  globalThis.IS_REACT_ACT_ENVIRONMENT = true;
  Deprecations
  react-dom: ReactDOM.render has been deprecated. Using it will warn and run your app in React 17 mode.
  react-dom: ReactDOM.hydrate has been deprecated. Using it will warn and run your app in React 17 mode.
  react-dom: ReactDOM.unmountComponentAtNode has been deprecated.
  react-dom: ReactDOM.renderSubtreeIntoContainer has been deprecated.
  react-dom/server: ReactDOMServer.renderToNodeStream has been deprecated.

### About IE

dropping support for internet explorer,
if you need to support Internet Explorer we recommend you stay with react 17;

### React DOM Server

renderToString
renderToStaticMarkup
