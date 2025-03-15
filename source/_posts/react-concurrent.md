---
title: react 并发模式
date: 2025-03-02 12:11:44
tags: react
---

### **1. 并发模式的基础概念**

- **并发渲染**：将渲染任务拆分为小单元，在浏览器空闲时执行。
- **可中断渲染**：渲染过程中可以中断当前任务，优先处理高优先级更新。
- **自动批处理**：将多个状态更新合并为一个渲染任务，减少渲染次数。

---

### **2. 并发模式的核心特性**

- **优先级调度**：
  - 高优先级更新（如用户交互）优先处理。
  - 低优先级更新（如数据渲染）延迟处理。
- **过渡更新（Transitions）**：
  - 使用 `startTransition` 将某些更新标记为低优先级。
- **延迟值（Deferred Value）**：
  - 使用 `useDeferredValue` 延迟某个值的更新。
- **Suspense**：
  - 在组件加载时显示 fallback UI，提升用户体验。

---

### **3. 并发模式的 API**

- **`startTransition`**：
  - 将更新标记为低优先级，确保高优先级更新不被阻塞。
  ```javascript
  startTransition(() => {
    setList(newList) // 低优先级更新
  })
  ```
- **`useDeferredValue`**：
  - 延迟某个值的更新，直到浏览器空闲时处理。
  ```javascript
  const deferredInput = useDeferredValue(input)
  ```
- **`Suspense`**：
  - 在组件加载时显示 fallback UI。
  ```javascript
  <Suspense fallback={<div>Loading...</div>}>
    <LazyComponent />
  </Suspense>
  ```

---

### **4. 启用并发模式的步骤**

1. 使用 `ReactDOM.createRoot` 渲染应用：
   ```javascript
   const root = ReactDOM.createRoot(document.getElementById("root"))
   root.render(<App />)
   ```
2. 使用并发模式特性（如 `startTransition`、`useDeferredValue`、`Suspense`）。

---

### **5. 并发模式的优势**

- **更流畅的用户体验**：优先处理用户交互，避免卡顿。
- **更好的性能**：减少不必要的渲染，提升性能。
- **更灵活的更新控制**：通过优先级调度精细控制更新。
- **更好的加载体验**：`Suspense` 使加载过程更平滑。

---

### **6. 注意事项**

- **兼容性**：确保应用升级到 React 18。
- **调试工具**：使用 React DevTools 调试并发模式。
- **潜在问题**：并发模式可能暴露竞态条件或副作用，需仔细测试。

---

### **7. 知识总结**

- **核心目标**：提升应用的响应性和性能。
- **关键机制**：并发渲染、优先级调度、可中断渲染。
- **主要 API**：`startTransition`、`useDeferredValue`、`Suspense`。
- **适用场景**：复杂 UI、高优先级更新、懒加载组件。

---

### **8. 记忆口诀**

- **并发渲染**：拆分任务，空闲执行。
- **优先级调度**：用户优先，数据延迟。
- **过渡更新**：`startTransition`，标记低优。
- **延迟值**：`useDeferredValue`，空闲更新。
- **Suspense**：懒加载，fallback UI。

---

通过这种线性结构，你可以逐步掌握并发模式的核心概念、特性、API 和应用场景，加深记忆并灵活运用。
