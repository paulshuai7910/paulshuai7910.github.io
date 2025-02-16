---
title: redis in node
date: 2024-05-04 09:55:23
tags:
---

# 安装 Redis 客户端库

```bash
npm install ioredis
```

# 配置连接

在你的 Node 服务器代码中，进行 Redis 的连接配置。如下：

```js
const Redis = require("ioredis")

const redisClient = new Redis({
  host: "redis_host", // 如果在同一网络中，可以使用 Redis 容器的服务名称，比如 'redis'
  port: 6379, // Redis 默认端口
  password: "your_redis_password", // 如果设置了密码
})

// 示例：设置一个键值对
redisClient.set("key", "value", (err, reply) => {
  if (err) {
    console.error("设置键值对失败：", err)
  } else {
    console.log("设置键值对成功，回复：", reply)
  }
})

// 示例：获取一个键的值
redisClient.get("key", (err, value) => {
  if (err) {
    console.error("获取键值失败：", err)
  } else {
    console.log("获取键值成功，值为：", value)
  }
})
```

# 使用位置

## 缓存数据

- 频繁查询的数据：对于一些经常被查询但不经常变动的数据，可以将其存储在 Redis 中。比如用户的基本信息、商品的详情等。当需要这些数据时，先从 Redis 中获取，如果 Redis 中不存在，再从数据库中查询，并将查询结果存入 Redis 以便下次快速获取。

````js
     async function getUserDetails(userId) {
       const cachedData = await redisClient.get(`user:${userId}`);
       if (cachedData) {
         return JSON.parse(cachedData);
       }
       const userFromDatabase = await databaseQueryToFetchUser(userId);
       await redisClient.set(`user:${userId}`,JSON.stringify(userFromDatabase));
       return userFromDatabase;
     }
     ```

````

- 计算结果缓存
  对于一些复杂的计算结果，如报表生成、数据分析等，如果计算过程耗时较长，可以将计算结果存入 Redis，下次需要时直接从 Redis 中获取，避免重复计算。

## 会话存储

可以将用户的会话信息存储在 Redis 中。这样可以实现分布式的会话管理，尤其在多个服务器节点的情况下，确保用户在不同请求之间的会话状态保持一致。

```js
// 存储会话
async function storeSession(sessionId, sessionData) {
  await redisClient.set(`session:${sessionId}`, JSON.stringify(sessionData))
}

// 获取会话
async function getSession(sessionId) {
  const sessionData = await redisClient.get(`session:${sessionId}`)
  if (sessionData) {
    return JSON.parse(sessionData)
  }
  return null
}
```

## 消息队列

Redis 的列表（list）数据结构可以用作简单的消息队列。可以将任务、事件等放入队列中，然后由后台的工作进程从队列中取出并处理。

```js
// 将任务放入队列
async function enqueueTask(task) {
  await redisClient.rpush("taskQueue", JSON.stringify(task))
}

// 从队列中取出任务
async function dequeueTask() {
  const taskData = await redisClient.lpop("taskQueue")
  if (taskData) {
    return JSON.parse(taskData)
  }
  return null
}
```

## 分布式锁

在分布式环境下，当多个进程需要对共享资源进行互斥访问时，可以使用 Redis 实现分布式锁。确保在同一时间只有一个进程能够访问特定的资源。

```js
// 获取分布式锁
async function acquireLock(key, timeout) {
  const result = await redisClient.set(key, "locked", "PX", timeout, "NX")
  return result === "OK"
}

async function releaseLock(key) {
  await redisClient.del(key)
}
```

# 存储限制

Redis 是一个高性能的键值存储数据库，它支持多种数据类型，包括字符串、哈希、列表、集合、有序集合等。但是，Redis 有一些限制，比如键的长度、值的大小、键的数量等。如果存储的数据超过了可用内存，Redis 会根据配置的内存淘汰策略来处理。常见的内存淘汰策略有

- Volatile LRU：根据数据的访问时间来淘汰数据，适用于缓存数据。使用 LRU（Least Recently Used，最近最少使用）算法进行淘汰。

- allkeys-lru:从所有键中，使用 LRU 算法进行淘汰。
- volatile-random: 从设置了过期时间的键中，随机淘汰。

- allkeys-random：从所有键中，随机淘汰。
- 可以通过修改 Redis 的配置文件或者在启动时使用命令行参数来调整内存限制。例如，可以设置 maxmemory 参数来指定 Redis 最大可用内存大小。

# 过期时间设置

Redis 可以为每个键设置过期时间，当键过期后，Redis 会自动删除该键。设置过期时间的方法有以下几种：
EXPIRE 命令：
可以在存储键值对后，使用 EXPIRE 命令为键设置过期时间，单位为秒。例如：EXPIRE key 60 表示设置键 key 的过期时间为 60 秒。
PEXPIRE 命令：
与 EXPIRE 类似，但单位为毫秒。例如：PEXPIRE key 60000 表示设置键 key 的过期时间为 60000 毫秒。
在 SET 命令中设置过期时间：
可以在使用 SET 命令存储键值对时，同时设置过期时间。例如：SET key value EX 60 表示设置键 key 的值为 value，并设置过期时间为 60 秒。
设置过期时间可以有效地管理 Redis 的内存使用，避免存储不必要的数据。同时，也可以用于实现一些特定的业务逻辑，如缓存过期、会话超时等。

# 配置 redis 内存淘汰策略

## Redis 容器的配置方式

如果是使用 Docker Compose 启动 Redis 容器，可以在 docker-compose.yml 文件中为 Redis 服务添加配置环境变量来设置内存淘汰策略。例如：

```yml
services:
  redis:
    image: redis:latest
    environment:
      - REDIS_ARGS=--maxmemory-policy volatile-lru
```

如果是通过 docker run 命令启动 Redis 容器，可以在命令中添加参数来设置内存淘汰策略。例如：

```yml
docker run --name some-redis -d redis redis-server --maxmemory-policy allkeys-lru
```
在使用 Node.js 作为后端开发时，设计合理的 Redis 缓存策略可以显著提升系统性能，减少数据库负载。以下是设计 Redis 缓存策略的关键步骤和注意事项，包括 **决定缓存哪些数据** 和 **设置缓存过期时间**。

# Redis 设计

## 1. **决定缓存哪些数据**

### **1.1 缓存适合的数据类型**
以下类型的数据适合缓存：
- **高频读取的数据**：如热门商品信息、用户基本信息、配置数据等。
- **计算成本高的数据**：如复杂的查询结果、聚合数据、统计结果等。
- **变化频率低的数据**：如静态配置、历史数据等。

### **1.2 不适合缓存的数据**
- **频繁更新的数据**：如实时库存、秒杀活动的剩余库存等。
- **敏感数据**：如密码、支付信息等（除非加密存储）。
- **数据一致性要求高的数据**：如金融交易数据。

---

## 2. **缓存策略设计**

### **2.1 缓存粒度**
- **细粒度缓存**：缓存单个对象（如用户信息、商品详情）。
  - 优点：灵活性高，可以单独更新或失效。
  - 缺点：缓存键数量多，管理复杂。
- **粗粒度缓存**：缓存整个页面或复杂查询结果。
  - 优点：减少缓存键数量，管理简单。
  - 缺点：更新或失效时影响范围大。

### **2.2 缓存更新策略**
- **写时更新（Write-Through）**：
  - 在写入数据库的同时更新缓存。
  - 优点：缓存与数据库保持一致。
  - 缺点：写操作性能较低。
- **写后更新（Write-Back）**：
  - 先更新缓存，异步更新数据库。
  - 优点：写操作性能高。
  - 缺点：存在数据不一致的风险。
- **缓存失效（Cache Invalidation）**：
  - 当数据更新时，使缓存失效，下次读取时重新加载。
  - 优点：简单易实现。
  - 缺点：可能导致缓存击穿（大量请求同时访问数据库）。

---

## 3. **设置缓存过期时间**

### **3.1 过期时间的作用**
- 避免缓存数据长期不更新，导致数据不一致。
- 自动清理不再使用的缓存，节省内存空间。

### **3.2 过期时间的设置原则**
- **高频访问的数据**：设置较短的过期时间（如 5 分钟），确保数据及时更新。
- **低频访问的数据**：设置较长的过期时间（如 1 小时），减少数据库访问。
- **静态数据**：设置非常长的过期时间（如 24 小时），甚至可以不设置过期时间。
- **动态数据**：根据业务需求设置合理的过期时间（如 10 分钟）。

### **3.3 过期时间的实现**
在 Redis 中，可以通过 `EXPIRE` 命令设置键的过期时间：
```javascript
// 设置缓存并指定过期时间（单位：秒）
await redisClient.set('user:123', JSON.stringify(userData), 'EX', 600); // 10 分钟
```

---

## 4. **缓存穿透、缓存击穿和缓存雪崩**

### **4.1 缓存穿透**
- **问题**：查询不存在的数据，导致请求直接访问数据库。
- **解决方案**：
  - 使用布隆过滤器（Bloom Filter）过滤无效请求。
  - 缓存空值（如 `null`），并设置较短的过期时间。

### **4.2 缓存击穿**
- **问题**：热点数据过期后，大量请求同时访问数据库。
- **解决方案**：
  - 使用互斥锁（Mutex Lock），只允许一个请求加载数据，其他请求等待。
  - 设置热点数据永不过期，通过后台任务定期更新。

### **4.3 缓存雪崩**
- **问题**：大量缓存同时过期，导致数据库负载骤增。
- **解决方案**：
  - 设置随机的过期时间，避免缓存集中失效。
  - 使用多级缓存（如本地缓存 + Redis 缓存）。

---

## 5. **Node.js 中的 Redis 缓存实现示例**

### **5.1 安装 Redis 客户端**
```bash
npm install redis
```

### **5.2 初始化 Redis 客户端**
```javascript
const redis = require('redis');
const redisClient = redis.createClient();

redisClient.on('error', (err) => {
  console.error('Redis error:', err);
});
```

### **5.3 缓存数据**
```javascript
async function getUserById(userId) {
  const cacheKey = `user:${userId}`;
  const cachedData = await redisClient.get(cacheKey);

  if (cachedData) {
    return JSON.parse(cachedData); // 返回缓存数据
  }

  // 从数据库加载数据
  const userData = await db.query('SELECT * FROM users WHERE id = ?', [userId]);

  if (userData) {
    // 将数据存入缓存，设置过期时间
    await redisClient.set(cacheKey, JSON.stringify(userData), 'EX', 600); // 10 分钟
  }

  return userData;
}
```

### **5.4 更新缓存**
```javascript
async function updateUser(userId, newData) {
  // 更新数据库
  await db.query('UPDATE users SET ? WHERE id = ?', [newData, userId]);

  // 使缓存失效
  await redisClient.del(`user:${userId}`);
}
```

---

## 6. **总结**

### **缓存策略设计要点**
1. **缓存适合的数据**：高频读取、计算成本高、变化频率低的数据。
2. **设置合理的过期时间**：根据数据访问频率和业务需求动态调整。
3. **处理缓存异常**：防止缓存穿透、击穿和雪崩。

### **Node.js 实现 Redis 缓存的步骤**
1. 安装 Redis 客户端。
2. 初始化 Redis 连接。
3. 在读取数据时优先检查缓存。
4. 在更新数据时使缓存失效。

通过合理的缓存策略，可以显著提升系统性能，减少数据库负载，同时保证数据的一致性和实时性。