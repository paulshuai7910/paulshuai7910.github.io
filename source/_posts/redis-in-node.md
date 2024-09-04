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
