---
title: AI 工作流内存泄漏问题
date: 2025-07-28 18:15:57
tags: memory-leak
---

**问题描述**
fastgpt V4.9.3 运行过程中发现每隔一段时间容器发生重启
进一步查看返现内容持续上升，达到设置阀值，触发重启

## 排查方案：

服务端增加监控，获取 profile 文件，导入插件，分析内存泄漏原因。

## 配置 docker 开启 debug 模式

```yml
获取dump文件
#1.复制工程代码
cd ~/snapshots/output
docker cp node_modules fastgpt-2:/tmp/
docker cp take-memory-snapshot.js fastgpt-2:/tmp/
#2.进入容器，开启DEBUG
docker exec -it fastgpt-2 sh
kill -USR1 1
#输出显示：
Debugger listening on ws://127.0.0.1:9229/a1111a16-b8fd-4348-9e7d-01aae1a70fd5
For help, see: https://nodejs.org/en/docs/inspector
#3.进入容器，执行dump脚本
cd /tmp
node take-memory-snapshot.js > heap-$(date +%s).heapsnapshot
#注意：dump脚本执行完后，容器会自动重启，请选择合适时间进行dump！
#4.下载dump文件，通过chrome://inspect 插件分析dump文件
Go to chrome://inspect
Click on “Open dedicated DevTools for Node”
Go to Memory tab
Press load profile button
Load all the heap snapshots you want to compare
工程代码路径：20上/home/albert/snapshots/output
```

## 分析内存泄漏原因

下载后会得到 profile-xxxxxxxxxxxxx.heapsnapshot 文件
导入到 chrome://inspect 插件中进行分析
找到内存泄漏的对象，分析泄漏路径，发现是因为工作流中的节点缓存没有清理，导致内存泄漏。

## 解决办法

对 settimeout 做了清理，继续做测试，内存稳定在 1 个多 G
