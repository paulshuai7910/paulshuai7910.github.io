---
title: react-hooks
date: 2024-05-22 22:03:38
tags: react
---

- createContext API 可以创建一个 context，你可以将其提供给子组件，通常会与 useContext 一起配合使用。
- forwardRef 允许组件将 DOM 节点作为 ref 暴露给父组件。
- lazy 允许你延迟加载组件，直到该组件需要第一次被渲染。
- memo 允许你在 props 没有变化的情况下跳过组件的重渲染。通常 useMemo 与 useCallback 会一起配合使用。
- startTransition 允许你可以标记一个状态更新是不紧急的。类似于 useTransition。
- act 允许你在测试中包装渲染和交互，以确保在断言之前已完成更新。

# State Hook

useState

- 使用 useState 声明可以直接更新的状态变量。

# Context Hook

useContext -帮助组件 从祖先组件接收信息，而无需将其作为 props 传递。

```js
const theme = useContext(ThemeContext)
```

# Ref Hook

useRef
ref 允许组件 保存一些不用于渲染的信息，比如 DOM 节点或 timeout ID。与状态不同，更新 ref 不会重新渲染组件。ref 是从 React 范例中的“脱围机制”。当需要与非 React 系统如浏览器内置 API 一同工作时，ref 将会非常有用。

# Effect Hook

useEffect
useEffect 有两个很少使用的变换形式，它们在执行时机有所不同：

- useLayoutEffect 在浏览器重新绘制屏幕前执行，可以在此处测量布局。
- useInsertionEffect 在 React 对 DOM 进行更改之前触发，库可以在此处插入动态 CSS。

# 性能 Hook

- 使用 useMemo 缓存计算代价昂贵的计算结果。
- 使用 useCallback 将函数传递给优化组件之前缓存函数定义。
- useTransition 允许将状态转换标记为非阻塞，并允许其他更新中断它。

```js
const [isPending, startTransition] = useTransition()
```

- useDeferredValue 允许延迟更新 UI 的非关键部分，以让其他部分先更新。
