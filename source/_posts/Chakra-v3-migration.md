---
title: Chakra UI v3 迁移指南 中文
date: 2025-04-11 12:00:45
tags:
---

notes：我们建议使用 LLMs.txt 文件，以使 Chakra UI v3 文档可用于大型语言模型。

## 以下是基于 Chakra UI v3 迁移指南的中文文档翻译

# Chakra UI v3 迁移指南

## 🚨 破坏性变更 (Breaking Changes)

### 1. 组件默认样式更新

• **`Button` 组件**  
 默认按钮样式已调整为更简约的扁平化设计。原 `variant="solid"` 的阴影效果被移除。  
 **迁移建议**：若需旧版样式，手动添加 `boxShadow` 属性或自定义主题。

```jsx
// 旧版 (v2)
<Button colorScheme="blue">Click me</Button>

// 新版 (v3) 默认无阴影
<Button colorScheme="blue" boxShadow="md">恢复阴影效果</Button>
```

### 2. `Theme` 结构重构

• **颜色系统**  
 `theme.colors` 中的颜色命名空间从 `lightBlue` 改为 `sky`（与 Tailwind CSS 对齐）。  
 **影响组件**：`Alert`, `Badge`, `Button` 等依赖颜色主题的组件。

```js
// 旧版 (v2) 主题扩展
extendTheme({
  colors: {
    lightBlue: { 500: "#0ea5e9" },
  },
})

// 新版 (v3) 使用 sky 命名空间
extendTheme({
  colors: {
    sky: { 500: "#0ea5e9" },
  },
})
```

### 3. React 18 兼容性

• **严格模式 (Strict Mode)**  
 v3 要求 React 18+，需检查组件中是否存在 `componentWillMount` 等废弃生命周期方法。

---

## 🛠️ 迁移步骤

### 1. 依赖升级

```bash
npm uninstall @chakra-ui/react @emotion/react @emotion/styled framer-motion
npm install @chakra-ui/react@3.0.0 @emotion/react@11 @emotion/styled@11 framer-motion@9
```

### 2. 主题适配

在 `ThemeProvider` 中启用新默认主题：

```jsx
import { ThemeProvider, extendTheme } from "@chakra-ui/react"

const theme = extendTheme({
  // 自定义覆盖（可选）
})

function App() {
  return (
    <ThemeProvider theme={theme}>
      <App />
    </ThemeProvider>
  )
}
```

### 3. 组件级修复

• **`Input` 组件**  
 `isFullWidth` 属性更名为 `width="100%"`。
• **`Stack` 组件**  
 `spacing` 单位从 `px` 改为 `rem`（默认 `spacing={4}` 对应 `1rem`）。

---

## ✅ 新特性亮点

• **动态颜色模式**：支持在运行时通过 `useColorModeValue` 动态切换亮/暗主题。
• **组合式 API**：新增 `createMultiStyleConfigHelpers` 简化多部件组件样式配置。
• **性能优化**：所有组件减少约 40% 的包体积。

---

## ❓ 常见问题

**Q: 升级后控制台出现 `invalid prop` 警告？**  
A: 移除已废弃的 props（如 `isDisabled` 改为 `isDisabled` 无变化，但需检查版本兼容性）。

**Q: 图标库是否需要单独安装？**  
A: 是的，v3 移除了内置图标，需通过 `@chakra-ui/icons` 安装。

## LLMs.txt 文档

如何使用 Cursor、Windstatic、GitHub Copilot、ChatGPT 和 Claude 等工具来理解 Chakra v3。

我们支持 LLMs.txt 文件，以便将 Chakra UI v3 文档提供给大型语言模型。
