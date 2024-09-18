---
title: 后端数据推送方法
date: 2024-01-25 09:18:21
tags:
---

- 长轮询相对简单但可能会消耗较多的服务器资源，
- SSE 专门用于服务器向客户端推送消息，但不支持双向通信。
- WebSocket 则提供了真正的双向实时通信能力。

# websocket

```js
const WebSocket = require("ws")

const wss = new WebSocket.Server({ port: 8080 })

wss.on("connection", (ws) => {
  console.log("客户端连接成功")

  ws.on("message", (message) => {
    console.log(`接收到消息：${message}`)
    // 向客户端发送消息
    ws.send(`服务器回应：${message}`)
  })

  ws.on("close", () => {
    console.log("客户端断开连接")
  })
})
```

client:

```js
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
</head>

<body>
  <script>
    const ws = new WebSocket('ws://localhost:8080');

    ws.onopen = () => {
      console.log('连接已建立');
      ws.send('你好，服务器！');
    };

    ws.onmessage = (event) => {
      console.log(`收到服务器消息：${event.data}`);
    };

    ws.onclose = () => {
      console.log('连接已关闭');
    };
  </script>
</body>

</html>
```

# 长轮询（Long Polling）

工作原理：
客户端向服务器发起一个请求，服务器如果没有新消息，就保持这个连接打开，直到有新消息或者超时。
当有新消息时，服务器立即响应并返回消息给客户端，客户端收到消息后，再发起新的请求。

```js
const express = require("express")
const app = express()
let messages = []

app.get("/poll", (req, res) => {
  if (messages.length > 0) {
    // 有消息时立即返回
    const message = messages.shift()
    res.json(message)
  } else {
    // 没有消息时保持连接打开
    res.writeHead(200, {
      Connection: "keep-alive",
      "Content-Type": "text/plain",
    })
    res.write("")
  }
})

// 模拟有新消息时向客户端推送
setInterval(() => {
  const newMessage = { text: "New message" }
  messages.push(newMessage)
  // 通知正在等待的客户端
  app.get("/poll").response.forEach((response) => {
    if (response.writable) {
      response.json(newMessage)
      response.end()
    }
  })
}, 5000)

app.listen(3000, () => {
  console.log("Server started on port 3000")
})
```

# 服务器发送事件（Server-Sent Events，SSE）

客户端通过 HTTP 请求建立一个持久的连接到服务器，服务器可以随时向客户端发送事件流。
客户端使用 EventSource 对象来接收服务器发送的事件。
server:

```js
const express = require("express")
const app = express()

app.get("/events", (req, res) => {
  res.writeHead(200, {
    "Content-Type": "text/event-stream",
    Connection: "keep-alive",
    "Cache-Control": "no-cache",
  })

  // 模拟定时发送消息
  setInterval(() => {
    const eventData = `data: New message at ${new Date().toLocaleTimeString()}\n\n`
    res.write(eventData)
  }, 5000)
})

app.listen(3000, () => {
  console.log("Server started on port 3000")
})
```

client:

```html
<script>
  const eventSource = new EventSource("/events")

  eventSource.onmessage = (event) => {
    console.log("Received event:", event.data)
  }

  eventSource.onerror = (error) => {
    console.error("Error with SSE connection:", error)
  }
</script>
```
