---
title: nextjs
date: 2025-03-05 21:11:09
tags:
---

# getServerSideProps（SSR）
📌 服务器端渲染（Server-Side Rendering, SSR）

每次请求 都会 运行在服务器端
适用于 动态数据（如用户定制内容）

# getStaticProps（SSG）
📌 静态生成（Static Site Generation, SSG）

生成 HTML 时 运行，仅在 构建时 执行
适用于 静态数据（如博客文章）
# getInitialProps 是什么？
📌 getInitialProps 是 Next.js 早期的数据获取方法，可用于 页面级组件，但 官方不推荐在新项目中使用，建议改用 getServerSideProps 或 getStaticProps。

### **Next.js 面试题 3**  
**问题：Next.js 如何处理 API 路由？API 路由与传统 Express/Koa 的区别是什么？**  

---

### **1️⃣ Next.js API 路由是什么？**  
📌 **Next.js 提供内置的 API 路由**，可以在 `pages/api/` 目录下创建 API 处理请求，类似于 Express/Koa 后端。  

✅ **示例**（`pages/api/hello.js`）：  
```javascript
export default function handler(req, res) {
  res.status(200).json({ message: "Hello, Next.js API!" });
}
```
📌 **特点**：  
- **运行在服务器端**（不会被前端打包）  
- **自动处理 JSON 解析**  
- **支持中间件、CORS 等**  

---

### **2️⃣ API 路由 vs 传统 Express/Koa**
| **特性** | **Next.js API 路由** | **Express/Koa** |
|---------|-----------------|-----------------|
| **创建方式** | `pages/api/*.js` | 需要手动创建 `server.js` |
| **服务器环境** | **自动管理**（Vercel 无需额外配置） | 需要 **自己配置服务器** |
| **性能** | 适用于 **小型 API** | 更适合 **复杂 API** |
| **路由** | **基于文件**（自动映射） | 需要手动 `app.get('/api')` |

📌 **适用场景**：
✅ 适合 **小型项目**（无需单独后端）  
✅ **前后端一体化**（如 SSR、静态生成 + API）  
❌ **不适合复杂 API**（如 WebSocket、流式数据）  

---

### **3️⃣ 处理 `POST` 请求**
✅ **示例**（`pages/api/user.js`）：  
```javascript
export default function handler(req, res) {
  if (req.method === "POST") {
    const { name } = req.body;
    return res.status(201).json({ message: `User ${name} created` });
  }
  res.status(405).json({ message: "Method Not Allowed" });
}
```
📌 **特点**：
- **支持 `req.body` 解析**（Next.js 自动解析 JSON）
- **需要手动检查 `req.method`**

---

### **4️⃣ 在前端调用 API**
✅ **使用 `fetch`：**
```javascript
async function fetchData() {
  const res = await fetch("/api/hello");
  const data = await res.json();
  console.log(data); // { message: "Hello, Next.js API!" }
}
```
---

### **5️⃣ API 路由支持动态参数**
✅ **动态 API 路由（`pages/api/user/[id].js`）：**
```javascript
export default function handler(req, res) {
  const { id } = req.query;
  res.status(200).json({ message: `User ID: ${id}` });
}
```
📌 **访问方式**：
```bash
GET /api/user/123
```
📌 **返回**：
```json
{ "message": "User ID: 123" }
```

---

### **6️⃣ 面试官可能的追问**
1. **Next.js API 适合做什么？**
   - ✅ 适合 **小型 API**（如身份验证、数据处理）  
   - ❌ 不适合 **长连接**（如 WebSocket），可以使用 **外部 API** 或 `next.config.js` 配置代理  

2. **如何在 API 路由中连接数据库？**
   - 可以使用 **Prisma / Mongoose / Knex.js**：
   ```javascript
   import { prisma } from "@/lib/prisma";

   export default async function handler(req, res) {
     const users = await prisma.user.findMany();
     res.status(200).json(users);
   }
   ```

3. **如何处理 CORS 跨域？**
   - **简单方式**：在 `res.setHeader` 添加 CORS 头：
   ```javascript
   res.setHeader("Access-Control-Allow-Origin", "*");
   res.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
   ```
   - **更好的方式**：使用 `next.config.js` 配置 API 代理，避免 CORS 问题：
   ```javascript
   module.exports = {
     async rewrites() {
       return [
         { source: "/api/:path*", destination: "https://external-api.com/:path*" },
       ];
     },
   };
   ```

---
### 动态路由
📌 动态路由 允许根据 URL 中的动态部分生成不同的页面。例如，URL /posts/[id] 可以匹配 /posts/1、/posts/2 等不同的页面。

Next.js 支持 文件系统路由，通过在 pages 文件夹中创建带有 方括号 的文件来定义动态路由。
```bash
/pages
  /posts
    [id].js
```
// 在这个例子中，[id].js 会匹配 /posts/1、/posts/2 等路径。动态部分是 id，它会成为 query 参数传递给页面组件。

### 动态路由的实现方法
📌 在 pages/posts/[id].js 中访问动态路由参数：
```javascript
import { useRouter } from 'next/router';

export default function Post() {
  const router = useRouter();
  const { id } = router.query;

  return <div>Post ID: {id}</div>;
}

```

### 实现带多个动态参数的路由？
📌 多个动态参数：你可以在文件名中定义多个动态部分，Next.js 会根据路由中不同的部分匹配它们。

### 获取动态路由数据
在动态路由中，你可能需要 获取远程数据。可以结合 getServerSideProps 或 getStaticProps 使用，以在请求页面时获取数据。
```javascript
// pages/posts/[id].js
export async function getServerSideProps(context) {
  const { id } = context.params;
  const res = await fetch(`https://api.example.com/posts/${id}`);
  const data = await res.json();

  return {
    props: { post: data },
  };
}

export default function Post({ post }) {
  return <div>{post.title}</div>;
}
// context.params 包含路由参数（如 { id: '1' }）。
// getServerSideProps 在每次请求时运行，适用于需要实时数据的动态页面。
```
# Incremental Static Regeneration (ISR)？
📌 Incremental Static Regeneration (ISR) 是 Next.js 的一项强大功能，它允许你在构建完成之后，更新静态页面，而无需重新构建整个站点。这意味着即使在应用运行后，静态页面也可以根据需要被重新生成，并且可以 增量更新，确保静态内容保持最新。

### 工作原理：
首次构建时：Next.js 会生成静态页面并将其保存下来。
增量更新：当一个页面请求到达时，Next.js 会检查该页面是否需要更新。如果页面需要更新，它会重新生成页面并替换旧的页面，同时在后台处理。
ISR 的关键点是：通过增量的方式更新静态页面，而无需重新构建整个站点。

### 启用 ISR？
ISR 是通过在 getStaticProps 方法中配置 revalidate 参数来启用的。revalidate 指定了静态页面重新生成的间隔时间（以秒为单位）。如果在 revalidate 时间内有用户请求访问该页面，Next.js 将重新生成该页面。
```javascript
// pages/index.js
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts },
    revalidate: 60,  // 每60秒重新生成一次页面
  };
}

export default function HomePage({ posts }) {
  return (
    <div>
      {posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}

```