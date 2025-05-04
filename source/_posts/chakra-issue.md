---
title: chakra 安装问题
date: 2025-05-04 19:32:09
tags:
---

## install successful

-> % HTTPS_PROXY=http://127.0.0.1:15236 npx @chakra-ui/cli snippet add color-mode
┌ Chakra CLI ⚡️
│
● Adding 1 snippet(s)...
│
◇ Installing required dependencies
│
◇ Writing selected snippets
│
▲ Skipping 1 file(s) that already exist. Use the --force flag to overwrite.
│
└ 🎉 Done!

## instal fail

npx @chakra-ui/cli snippet add color-mode
┌ Chakra CLI ⚡️
file:///Users/paul.shuai/.npm/\_npx/de2e382d70c335ff/node_modules/node-fetch/src/index.js:108
reject(new FetchError(`request to ${request.url} failed, reason: ${error.message}`, 'system', error));
^

FetchError: request to https://chakra-v3-docs.vercel.app/compositions/index.json failed, reason:
at ClientRequest.<anonymous> (file:///Users/paul.shuai/.npm/\_npx/de2e382d70c335ff/node_modules/node-fetch/src/index.js:108:11)
at ClientRequest.emit (node:events:520:28)
at emitErrorEvent (node:\_http_client:103:11)
at TLSSocket.socketErrorListener (node:\_http_client:506:5)
at TLSSocket.emit (node:events:520:28)
at emitErrorNT (node:internal/streams/destroy:170:8)
at emitErrorCloseNT (node:internal/streams/destroy:129:3)
at process.processTicksAndRejections (node:internal/process/task_queues:90:21) {
type: 'system',
errno: 'ETIMEDOUT',
code: 'ETIMEDOUT',
erroredSysCall: undefined
}

Node.js v22.8.0

## 操作方法

对命令行工具进行代理
HTTPS_PROXY=http://127.0.0.1:15236 npx @chakra-ui/cli snippet add color-mode

https 必须为大写字母，小写字母不可以

<!-- add image -->

![](issue.jpg)
