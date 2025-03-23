---
title: dev Scenario
date: 2024-05-04 13:48:19
tags:
---

# Graphql n+1 问题

data-loader

# JWT 会话超时机制

### 一、前端实现（以 React 为例）

#### 1. 监听用户活动并重置计时器

```javascript
const SessionTimeout = () => {
  const [logoutTimer, setLogoutTimer] = useState(null)
  const [warningTimer, setWarningTimer] = useState(null)
  const [showWarning, setShowWarning] = useState(false)

  // 用户操作事件监听（点击、键盘、滚动）
  const resetTimers = () => {
    // 清除之前的定时器
    cleatTimeout(logoutTimer)
    cleatTimeout(warningTimer)
    setShowWarning(false)

    // 重新启动15分钟无操作检查
    const newWarningTimer = setTimeout(() => {
      setShowWarning(true)
      // 5分钟后自动登出
      const newLogoutTimer = setTimeout(() => {
        // 退出登录
        logout()
      }, 5 * 60 * 1000)
      setLogoutTimer(newLogoutTimer)
    }, 15 * 60 * 1000)
    setWarningTimer(newWarningTimer)
  }

  // 初始化事件监听
  useEffect(() => {
    const events = ["mousemove", "keydown", "click"]
    events.forEach((event) => {
      window.addEventListener(event, resetTimers)
    })
    resetTimers()

    return () => {
      events.forEach((event) => {
        window.removeEventListener(event, resetTimers)
      })
      clearTimeout(logoutTimer)
      clearTimeout(warningTimer)
    }
  }, [])

  // Token刷新逻辑（用户操作时调用）
  const refreshToken = async () => {
    try {
      const response = await fetch("/api/auth/refresh-token", {
        method: "POST",
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      })
      const newToken = await response.json()
      localStorage.setItem("token", newToken)
    } catch (error) {
      handleLogout()
    }
  }
  // 退出登录
  const handleLogout = () => {
    localStorage.removeItem("token")
    window.location.href = "/login"
  }
  // 弹窗组件
  return showWarning ? (
    <div className="timeout-warning">
      <p>长时间未操作，5分钟后将自动退出登录！</p>
      <button onClick={refreshToken}>保持登录</button>
    </div>
  ) : null
}
```

## 后端实现（nodejs）

```js
const jwt = require("jsonwebtoken")
const express = require("express")
const router = express.Router()

router.post("/api/auth/refresh-token", (req, res) => {
  const oldToken = req.headers.authorization.split(" ")[1]
  try {
    // 验证旧Token是否有效
    const decoded = jwt.verify(oldToken, process.env.JWT_SECRET)

    // 生成新Token（延长有效期）
    const newToken = jwt.sign(
      { userId: decoded.userId },
      process.env.JWT_SECRET,
      { expiresIn: "20m" } // 新Token有效期20分钟
    )

    res.json(newToken)
  } catch (error) {
    res.status(401).json({ message: "Token无效或已过期" })
  }
})

module.exports = router
```

## 关键逻辑

1. 用户无操作检测
   - 前端监听 mousemove、keydown 等事件，每次操作重置 15 分钟计时器。
   - 15 分钟无操作后弹出警告，并启动 5 分钟最终退出计时。
2. Token 刷新机制 ​
   - 用户点击弹窗中的“保持登录”按钮时，调用/api/auth/refresh-token 接口。
   - 后端验证旧 Token 有效性，生成新 Token 并返回，前端更新本地存储。
3. ​ 安全性注意事项 ​
   - JWT Secret 需存储在环境变量中，避免硬编码。
   - 使用 HTTPS 传输 Token，防止中间人攻击。
   - 前端 Token 建议存储在 HttpOnly Cookie 中

## 优化方案

1. token 自动刷新
   - 在用户操作时（如路由跳转、API 请求）自动刷新 Token，无需用户手动点击。
2. ​ 心跳检测 ​
   - 每隔固定时间（如 10 分钟）向后端发送心跳请求，主动维持会话。
3. 多标签页同步 ​
   - 使用 BroadcastChannel 或 localStorage 事件同步多个标签页的超时状态。
   ```js
   // 多标签页同步示例
   const channel = new BroadcastChannel("session_timeout")
   channel.onmessage = (e) => {
     if (e.data === "logout") handleLogout()
   }
   // BroadcastChannel 是浏览器提供的一种 ​跨标签页通信​ 的 API，允许同一域名下的不同标签页、iframe 或 Worker 之间进行消息传递。
   ```

# postgreSQL 最佳实践 -- 优化器查询计划
