---
title: node 调试工具
date: 2024-10-09 12:49:11
tags:
---

# 1. Node.js 基础
   - **事件驱动**
     - 事件循环（Event Loop）
     - 非阻塞 I/O
   - **模块系统**
     - CommonJS 模块
     - `require` 和 `module.exports`
     - ES 模块（`import/export`）
   - **全局对象**
     - `global`
     - `process`
     - `Buffer`
   - **核心模块**
     - `fs`：文件系统
     - `http`/`https`：HTTP 服务器
     - `path`：路径处理
     - `stream`：流处理
     - `events`：事件触发器
     - `child_process`：子进程
     - `cluster`：集群

# 2. 异步编程
   - **回调函数**
     - 回调地狱（Callback Hell）
     - 错误优先回调（Error-first Callback）
   - **Promise**
     - `Promise` 链
     - `Promise.all`/`Promise.race`
   - **Async/Await**
     - 异步函数
     - 错误处理（`try/catch`）
   - **事件驱动**
     - `EventEmitter`
     - 自定义事件

# 3. 网络编程
   - **HTTP/HTTPS**
     - 创建 HTTP 服务器
     - 处理请求和响应
     - RESTful API 设计
   - **WebSocket**
     - 实时通信
     - `ws` 库
   - **TCP/UDP**
     - `net` 模块
     - `dgram` 模块

# 4. 数据库
   - **SQL 数据库**
     - MySQL
     - PostgreSQL
   - **NoSQL 数据库**
     - MongoDB
     - Redis
   - **ORM/ODM**
     - Sequelize（SQL）
     - Mongoose（MongoDB）

# 5. 框架与工具
   - **Express.js**
     - 路由
     - 中间件
     - 错误处理
   - **Koa.js**
     - 中间件机制
     - 上下文（Context）
   - **NestJS**
     - 模块化
     - 依赖注入
   - **工具**
     - NPM/Yarn
     - Nodemon
     - PM2

# 6. 性能优化
   - **事件循环优化**
     - 避免阻塞操作
     - 使用 `setImmediate` 和 `process.nextTick`
   - **集群与负载均衡**
     - `cluster` 模块
     - 负载均衡策略
   - **缓存**
     - Redis 缓存
     - 内存缓存（如 `node-cache`）
   - **流处理**
     - 可读流、可写流、双工流
     - 管道（`pipe`）

# 7. 安全
   - **常见漏洞**
     - XSS（跨站脚本攻击）
     - CSRF（跨站请求伪造）
     - SQL 注入
   - **安全实践**
     - 使用 HTTPS
     - 输入验证与清理
     - 使用 Helmet 中间件
   - **认证与授权**
     - JWT（JSON Web Token）
     - OAuth 2.0

# 8. 测试
   - **单元测试**
     - Mocha
     - Jest
   - **集成测试**
     - Supertest
   - **测试覆盖率**
     - Istanbul（`nyc`）
   - **Mocking**
     - Sinon.js

# 9. 部署与 DevOps
   - **部署**
     - Docker 容器化
     - Kubernetes
   - **CI/CD**
     - Jenkins
     - GitHub Actions
   - **日志管理**
     - Winston
     - Morgan
   - **监控**
     - Prometheus
     - Grafana

# 10. 高级主题
   - **微服务架构**
     - 服务发现
     - 消息队列（RabbitMQ、Kafka）
   - **GraphQL**
     - Apollo Server
     - 类型定义与解析器
   - **Serverless**
     - AWS Lambda
     - Google Cloud Functions
   - **TypeScript**
     - 类型系统
     - 与 Node.js 集成

# 11. 面试常见问题
   - **Node.js 事件循环机制**
   - **如何避免回调地狱？**
   - **Express 中间件的工作原理**
   - **如何优化 Node.js 应用的性能？**
   - **如何处理未捕获的异常？**
   - **如何实现 JWT 认证？**
   - **如何设计一个高并发的 Node.js 应用？**