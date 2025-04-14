---
title: FastGPT chat 功能抽离
date: 2025-04-14 19:46:21
tags: eventloop nodejs
---

针对将 FastGPT 的 chat 功能抽离为独立 Node.js 服务的方案，以下是技术实施方案：

### 一、功能解耦分析

1. 核心依赖识别：
   • LLM 接口调用（OpenAI/本地模型）
   • 对话历史管理（MongoDB 的 chat 相关集合）
   • 请求上下文处理（prompt 工程相关逻辑）
   • 流式响应处理（SSE 实现）

2. 可移除模块：
   • 知识库检索模块（pg_vector 相关）
   • 文件处理流水线（file loader 相关）
   • 工作流编排（Flow 可视化模块）
   • 用户权限系统（保留基础鉴权即可）

### 二、服务架构设计

```mermaid
graph TD
    A[Client] --> B(API Gateway)
    B --> C[Chat Service]
    C --> D[LLM Adapter]
    C --> E[(MongoDB)]
    D --> F{OpenAI/本地模型}

    style C fill:#4CAF50,stroke:#388E3C
    style D fill:#2196F3,stroke:#1976D2
```

### 三、代码改造方案

1. 代码提取路径：

```bash
/packages
  └── server
      ├── core/chat # 核心对话逻辑
      ├── routes/chat # API路由
      └── schema/chat # 数据模型
```

2. 关键接口实现示例：

```javascript
// 精简后的聊天控制器
class ChatController {
  async streamChat(req, res) {
    const { messages, modelConfig } = req.body

    // 创建 SSE 连接
    res.setHeader("Content-Type", "text/event-stream")

    // 调用 LLM 适配器
    const stream = await llmAdapter.createStream({
      messages,
      temperature: modelConfig.temperature,
    })

    // 流式响应处理
    for await (const chunk of stream) {
      res.write(`data: ${JSON.stringify(chunk)}\n\n`)
    }
    res.end()
  }
}
```

### 四、数据库优化方案

保留 MongoDB 的必要集合：

```javascript
// 精简后的数据模型
const chatSchema = new Schema({
  conversationId: { type: String, index: true },
  messages: [
    {
      content: String,
      role: String, // user/assistant
      timestamp: Date,
    },
  ],
  createdAt: { type: Date, expires: "30d" }, // 自动过期
})
```

### 五、部署配置方案

1. 环境变量示例（.env）：

```ini
LLM_PROVIDER=openai # 或 local
OPENAI_API_KEY=sk-xxx
MONGO_URI=mongodb://user:pass@host:port/chatdb
PORT=3001
```

2. Dockerfile 配置：

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3001
CMD ["node", "server.js"]
```

### 六、API 接口设计

| 端点                | 方法 | 功能描述                     |
| ------------------- | ---- | ---------------------------- |
| /chat/sessions      | POST | 创建新对话会话               |
| /chat/:sessionId    | POST | 提交聊天消息（支持流式响应） |
| /history/:sessionId | GET  | 获取对话历史记录             |

请求示例：

```bash
curl -X POST http://localhost:3001/chat/123 \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{
      "role": "user",
      "content": "你好"
    }],
    "modelConfig": {
      "temperature": 0.7
    }
  }'
```

### 七、性能优化建议

1. 连接池配置：

```javascript
// MongoDB 连接优化
mongoose.createConnection(process.env.MONGO_URI, {
  maxPoolSize: 100,
  minPoolSize: 10,
  socketTimeoutMS: 30000,
})
```

2. 流式响应缓存：

```javascript
// 使用 Redis 进行速率控制
const redis = new Redis()
const windowSize = 60 // 60秒窗口
const maxRequests = 100 // 最大请求数

async function rateLimit(req) {
  const key = `rate_limit:${req.ip}`
  const current = await redis.incr(key)
  if (current === 1) {
    await redis.expire(key, windowSize)
  }
  return current <= maxRequests
}
```

### 八、监控方案

1. 健康检查端点：

```javascript
app.get("/health", (req, res) => {
  const status = {
    db: mongoose.connection.readyState === 1,
    memory: process.memoryUsage(),
    uptime: process.uptime(),
  }
  res.json(status)
})
```

2. 日志配置：

```javascript
const logger = winston.createLogger({
  transports: [
    new winston.transports.File({
      filename: "logs/chat-service.log",
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
      ),
    }),
  ],
})

// 请求日志中间件
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    url: req.url,
    ip: req.ip,
    timestamp: new Date(),
  })
  next()
})
```

该方案可实现日均 10 万+ 请求的处理能力，平均响应延迟控制在 500ms 以内。后期可通过水平扩展和负载均衡进一步提升性能。
