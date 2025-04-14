---
title: next 渲染模式
date: 2025-04-07 11:22:18
tags: nextjs render
---

## 服务端渲染 Server side rendering

工作原理：
每次页面请求时，Next.js 在服务器端实时生成完整的 HTML，发送给客户端。

```javascript
export async function getServerSideProps(context) {
  const res = await fetch("https://api.example.com/data")
  const data = await res.json()
  return { props: { data } } // 传递给页面组件
}

export default function pages({ data }) {
  return <div>{data.title}</div>
}
```

## 静态生成 Static site rendering

## 客户端渲染 Client side rendering

## Incremental Static Regeneration (ISR)
