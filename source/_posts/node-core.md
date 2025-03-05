---
title: node 多核
date: 2025-03-05 22:08:56
tags:
---

Node.js 是单线程的，默认情况下无法充分利用多核 CPU 的性能。为了解决这个问题，Node.js 提供了 **Cluster 模块**，它可以帮助我们创建多个子进程（Worker），每个子进程运行在独立的 CPU 核心上，从而提升应用程序的性能和吞吐量。

以下是使用 Cluster 模块提升多核 CPU 利用率的详细说明：

---

### **一、Cluster 模块的工作原理**
1. **主进程（Master）**：
   • 负责创建和管理子进程。
   • 监听端口，并将请求分发给子进程。
   • 可以监控子进程的状态（如崩溃后重启）。

2. **子进程（Worker）**：
   • 每个子进程都是一个独立的 Node.js 实例。
   • 处理主进程分发的请求。
   • 共享同一个端口。

3. **通信机制**：
   • 主进程和子进程通过 IPC（Inter-Process Communication）进行通信。

---

### **二、使用 Cluster 模块的步骤**

#### **1. 引入 Cluster 模块**
```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');
```

#### **2. 判断当前进程是主进程还是子进程**
```javascript
if (cluster.isMaster) {
  // 主进程逻辑
} else {
  // 子进程逻辑
}
```

#### **3. 主进程创建子进程**
```javascript
if (cluster.isMaster) {
  const numCPUs = os.cpus().length; // 获取 CPU 核心数
  console.log(`Master process is running. Forking for ${numCPUs} CPUs...`);

  // 根据 CPU 核心数创建子进程
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // 监听子进程退出事件
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Forking a new one...`);
    cluster.fork(); // 重启子进程
  });
}
```

#### **4. 子进程处理请求**
```javascript
else {
  // 创建一个 HTTP 服务器
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello, World!\n');
  }).listen(3000);

  console.log(`Worker ${process.pid} started`);
}
```

---

### **三、完整代码示例**
```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  console.log(`Master process is running. Forking for ${numCPUs} CPUs...`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Forking a new one...`);
    cluster.fork();
  });
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('Hello, World!\n');
  }).listen(3000);

  console.log(`Worker ${process.pid} started`);
}
```

---

### **四、Cluster 模块的优势**
1. **提升性能**：
   • 充分利用多核 CPU，提高并发处理能力。
2. **高可用性**：
   • 子进程崩溃后，主进程可以自动重启新的子进程。
3. **负载均衡**：
   • 主进程将请求均匀分发给子进程，避免单个进程过载。

---

### **五、Cluster 模块的注意事项**
1. **共享状态**：
   • 子进程之间不共享内存，需要通过 Redis 或数据库共享状态。
2. **端口占用**：
   • 所有子进程共享同一个端口，由操作系统负责负载均衡。
3. **资源消耗**：
   • 创建过多子进程可能导致内存和 CPU 资源耗尽。
4. **不适合所有场景**：
   • 如果应用是 CPU 密集型任务，Cluster 模块效果显著；如果是 I/O 密集型任务，效果可能有限。

---

### **六、优化 Cluster 模块的使用**
1. **动态调整子进程数量**：
   • 根据系统负载动态创建或销毁子进程。
   • 示例：
     ```javascript
     if (cluster.isMaster) {
       const numCPUs = os.cpus().length;
       for (let i = 0; i < numCPUs; i++) {
         cluster.fork();
       }

       // 动态调整子进程数量
       setInterval(() => {
         const load = os.loadavg()[0]; // 获取系统负载
         if (load > numCPUs && Object.keys(cluster.workers).length < numCPUs * 2) {
           cluster.fork();
         } else if (load < numCPUs / 2 && Object.keys(cluster.workers).length > numCPUs) {
           const workerId = Object.keys(cluster.workers)[0];
           cluster.workers[workerId].kill();
         }
       }, 5000);
     }
     ```

2. **使用 PM2 管理进程**：
   • PM2 是一个进程管理工具，可以自动使用 Cluster 模块，并提供监控、日志等功能。
   • 启动命令：
     ```bash
     pm2 start app.js -i max
     ```

3. **结合其他技术**：
   • 使用 Nginx 或 HAProxy 进行反向代理和负载均衡。
   • 结合 Redis 或消息队列（如 RabbitMQ）实现进程间通信。

---

### **七、适用场景**
1. **Web 服务器**：
   • 处理大量并发 HTTP 请求。
2. **实时应用**：
   • 如聊天服务器、实时数据推送。
3. **CPU 密集型任务**：
   • 如加密解密、图像处理。

---

通过 Cluster 模块，可以显著提升 Node.js 应用在多核 CPU 环境下的性能，但需要根据实际场景合理配置和优化。