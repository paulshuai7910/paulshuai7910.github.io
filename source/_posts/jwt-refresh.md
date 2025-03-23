---
title: JWT-token 无感刷新
date: 2025-03-18 14:15:58
tags:
---

### **1. 整体架构**

1. **核心流程**：
   • **无感刷新**：通过 Refresh Token 自动获取新的 Access Token。
   • **黑名单机制**：将需要失效的 Token（如用户主动登出后的旧 Token）存入 Redis，并在验证时检查黑名单。

2. **技术栈**：
   • JWT（Access Token + Refresh Token）
   • Redis（存储黑名单和 Refresh Token）
   • 后端框架（如 Node.js/Express、Spring Boot 等）

---

### **2. 实现步骤**

#### **2.1 服务器端设计**

##### **(1) 登录接口**

用户登录成功后，生成并返回 Access Token 和 Refresh Token，同时将 Refresh Token 存入 Redis。

```javascript
// Node.js + Express 示例
app.post("/login", async (req, res) => {
  const { username, password } = req.body
  // 1. 验证用户身份
  const user = await validateUser(username, password)
  if (!user) return res.status(401).json({ error: "Invalid credentials" })

  // 2. 生成 Access Token（有效期短）
  const accessToken = jwt.sign({ userId: user.id }, "ACCESS_TOKEN_SECRET", {
    expiresIn: "15m",
  })

  // 3. 生成 Refresh Token（有效期长）
  const refreshToken = jwt.sign({ userId: user.id }, "REFRESH_TOKEN_SECRET", {
    expiresIn: "7d",
  })

  // 4. 存储 Refresh Token 到 Redis（关联用户ID）
  await redisClient.set(
    `refresh_token:${user.id}`,
    refreshToken,
    "EX",
    7 * 24 * 60 * 60
  )

  res.json({ accessToken, refreshToken })
})
```

##### **(2) 刷新接口**

通过 Refresh Token 获取新的 Access Token，并验证 Redis 中的有效性。

```javascript
app.post("/refresh-token", async (req, res) => {
  const { refreshToken } = req.body
  try {
    // 1. 验证 Refresh Token 是否有效
    const decoded = jwt.verify(refreshToken, "REFRESH_TOKEN_SECRET")
    const userId = decoded.userId

    // 2. 检查 Redis 中是否存在该用户的 Refresh Token
    const storedRefreshToken = await redisClient.get(`refresh_token:${userId}`)
    if (refreshToken !== storedRefreshToken) {
      return res.status(401).json({ error: "Invalid refresh token" })
    }

    // 3. 生成新的 Access Token
    const newAccessToken = jwt.sign({ userId }, "ACCESS_TOKEN_SECRET", {
      expiresIn: "15m",
    })

    res.json({ accessToken: newAccessToken })
  } catch (error) {
    res.status(401).json({ error: "Refresh token expired or invalid" })
  }
})
```

##### **(3) 黑名单机制**

用户登出或需要撤销 Token 时，将旧 Access Token 加入 Redis 黑名单。

```javascript
app.post("/logout", async (req, res) => {
  const { accessToken } = req.body
  // 1. 解码 Access Token 获取过期时间
  const decoded = jwt.decode(accessToken)
  const exp = decoded.exp

  // 2. 计算剩余有效期（秒）
  const ttl = exp - Math.floor(Date.now() / 1000)

  // 3. 将 Token 加入黑名单（过期后自动删除）
  if (ttl > 0) {
    await redisClient.set(`blacklist:${accessToken}`, "revoked", "EX", ttl)
  }

  // 4. 删除该用户的 Refresh Token（可选）
  const userId = decoded.userId
  await redisClient.del(`refresh_token:${userId}`)

  res.json({ message: "Logged out successfully" })
})
```

##### **(4) 中间件：验证 Access Token**

在每次请求中验证 Access Token 是否在黑名单中。

```javascript
const authMiddleware = async (req, res, next) => {
  const authHeader = req.headers.authorization
  const accessToken = authHeader?.split(" ")[1]

  if (!accessToken) {
    return res.status(401).json({ error: "No token provided" })
  }

  try {
    // 1. 检查 Token 是否在黑名单
    const isBlacklisted = await redisClient.exists(`blacklist:${accessToken}`)
    if (isBlacklisted) {
      return res.status(401).json({ error: "Token revoked" })
    }

    // 2. 验证 Token 有效性
    const decoded = jwt.verify(accessToken, "ACCESS_TOKEN_SECRET")
    req.userId = decoded.userId
    next()
  } catch (error) {
    res.status(401).json({ error: "Invalid or expired token" })
  }
}
```

---

#### **2.2 客户端设计**

客户端需在 Access Token 过期前自动刷新，并在登出时触发黑名单机制。

```javascript
// 示例：Axios 请求拦截器（React/Vue）
axios.interceptors.request.use(async (config) => {
  const accessToken = localStorage.getItem("accessToken")
  const refreshToken = localStorage.getItem("refreshToken")

  // 1. 检查 Access Token 是否即将过期
  const decoded = jwt.decode(accessToken)
  const isExpired = decoded.exp * 1000 < Date.now() + 5 * 60 * 1000 // 过期前5分钟

  if (isExpired) {
    try {
      // 2. 自动刷新 Token
      const response = await axios.post("/refresh-token", { refreshToken })
      const newAccessToken = response.data.accessToken
      localStorage.setItem("accessToken", newAccessToken)
      config.headers.Authorization = `Bearer ${newAccessToken}`
    } catch (error) {
      // 刷新失败，跳转登录页
      window.location.href = "/login"
    }
  }

  return config
})
```

---

### **3. Redis 黑名单优化**

1. **自动清理**：通过设置 `EX` 参数，让 Redis 自动删除过期的黑名单 Token。
2. **内存管理**：定期清理无效数据，避免内存占用过多。
3. **分片存储**：使用 `blacklist:${token}` 的键名格式，避免单个 Key 过大。

---

### **4. 安全性增强**

1. **Refresh Token 存储**：
   • 使用 `httpOnly` Cookie 存储 Refresh Token，防止 XSS 攻击。
   • 每次刷新后生成新的 Refresh Token，旧 Token 立即失效（需更新 Redis）。
2. **Token 轮换**：
   • 每次刷新 Access Token 时，生成新的 Refresh Token 并替换 Redis 中的旧值。
3. **密钥管理**：
   • 使用强随机密钥（如 `ACCESS_TOKEN_SECRET` 和 `REFRESH_TOKEN_SECRET`）。
   • 定期轮换密钥，降低泄露风险。

---

### **5. 流程图**

```plaintext
用户登录
  │
  ├─ 生成 Access Token（15分钟）和 Refresh Token（7天）
  │
  ├─ 存储 Refresh Token 到 Redis（关联用户ID）
  │
  └─ 返回 Token 给客户端

用户请求受保护接口
  │
  ├─ 验证 Access Token 是否在黑名单
  │
  ├─ 若在黑名单 → 拒绝请求
  │
  └─ 若有效 → 允许访问

Access Token 过期前
  │
  ├─ 客户端自动调用刷新接口
  │
  ├─ 验证 Refresh Token（检查 Redis）
  │
  ├─ 生成新 Access Token
  │
  └─ 更新客户端存储

用户登出
  │
  ├─ 将当前 Access Token 加入 Redis 黑名单
  │
  └─ 删除 Redis 中的 Refresh Token
```

---
