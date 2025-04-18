---
title: ts in node
date: 2024-05-04 09:55:09
tags: typescript nodejs
---

Node.js 无法原生运行 TypeScript。你无法直接从命令行调用 node example.ts。但是这个问题有三种解决方案

# 在 Node.js 中运行 TypeScript 代码

npx 代表 Node Package Execute。此工具允许我们运行 TypeScript 的编译器而无需全局安装它。

## 编译 TypeScript 到 JavaScript

```bash
npx tsc example.ts
node example.js

```

## 使用 ts-node 运行 TypeScript 代码

要使用 ts-node，你需要先安装它：

```bash
npm install -g ts-node
```

然后你可以像这样运行你的 TypeScript 代码：

```bash
npx ts-node example.ts
```

# 使用 tsx 运行 TypeScript 代码

你可以使用 tsx 直接在 Node.js 中运行 TypeScript 代码，而无需先编译它。但是它不是对你的代码进行类型检查。因此，我们建议在交付之前先使用 tsc 对代码进行类型检查，然后使用 tsx 运行它。

```bash
npm i -D tsx
npx tsx example.ts
```

- 同时需要修改配置：
  tsconfig.json

```json
    "ts-node": {
      "esm": true
    },
```

# 通过 node 使用 tsx，你可以通过 --import 注册 tsx：

```bash
node --import=tsx example.ts
```
