---
title: docker build demo
date: 2025-04-15 18:12:43
tags:
---

| 代码 | 解释 |
| ---- | ---- |

| ```dockerfile

# --------- install dependence -----------

FROM node:20.14.0-alpine AS maindeps
WORKDIR /app

ARG proxy

RUN [ -z "$proxy" ] || sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
RUN apk add --no-cache libc6-compat && npm install -g pnpm@9.4.0 --registry=https://registry.npmmirror.com

# copy packages and one project

COPY pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ./packages ./packages
COPY ./projects/app/package.json ./projects/app/package.json

RUN [ -f pnpm-lock.yaml ] || (echo "Lockfile not found." && exit 1)

# if proxy exists, set proxy

RUN if [ -z "$proxy" ]; then \
 pnpm i; \
 else \
 pnpm i --registry=https://registry.npmmirror.com; \
 fi
`` | **第一阶段：安装依赖 (maindeps)**<br><br>1. 基础镜像：使用 `node:20.14.0-alpine` 作为基础镜像，这是一个轻量级的 Node.js 环境<br>2. 工作目录：设置工作目录为 `/app`<br>3. 代理设置：<br>   - 如果提供了 `proxy` 参数，将 Alpine 的软件源替换为中科大镜像源<br>4. 安装工具：<br>   - 安装 `libc6-compat` 库（兼容性库）<br>   - 安装 `pnpm@9.4.0` 包管理器，使用淘宝镜像源<br>5. 复制文件：<br>   - 复制 `pnpm-lock.yaml`、`pnpm-workspace.yaml`、`.npmrc` 配置文件<br>   - 复制 `packages` 目录<br>   - 复制 `projects/app/package.json`<br>6. 依赖安装：<br>   - 检查 `pnpm-lock.yaml` 是否存在<br>   - 根据是否有代理，选择不同的方式安装依赖 |
| ``dockerfile

# --------- builder -----------

FROM node:20.14.0-alpine AS builder
WORKDIR /app

ARG proxy
ARG base_url

# copy common node_modules and one project node_modules

COPY package.json pnpm-workspace.yaml .npmrc tsconfig.json ./
COPY --from=maindeps /app/node_modules ./node_modules
COPY --from=maindeps /app/packages ./packages
COPY ./projects/app ./projects/app
COPY --from=maindeps /app/projects/app/node_modules ./projects/app/node_modules

RUN [ -z "$proxy" ] || sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

RUN apk add --no-cache libc6-compat && npm install -g pnpm@9.4.0 --registry=https://registry.npmmirror.com

ENV NODE_OPTIONS="--max-old-space-size=4096"
ENV NEXT_PUBLIC_BASE_URL=$base_url
RUN pnpm --filter=app build
`` | **第二阶段：构建应用 (builder)**<br><br>1. 基础镜像：同样使用 `node:20.14.0-alpine`<br>2. 工作目录：设置工作目录为 `/app`<br>3. 复制文件：<br>   - 从 `maindeps` 阶段复制 `node_modules` 和 `packages`<br>   - 复制项目源代码 `projects/app`<br>   - 复制项目依赖 `projects/app/node_modules`<br>4. 环境配置：<br>   - 设置 Node.js 内存限制为 4GB<br>   - 设置 `NEXT_PUBLIC_BASE_URL` 环境变量<br>5. 构建应用：<br>   - 使用 `pnpm` 构建 app 项目 |
| ``dockerfile

# --------- runner -----------

FROM node:20.14.0-alpine AS runner
WORKDIR /app

ARG proxy
ARG base_url

# create user and use it

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

RUN [ -z "$proxy" ] || sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
RUN apk add --no-cache curl ca-certificates \
 && update-ca-certificates

# copy running files

COPY --from=builder /app/projects/app/public /app/projects/app/public
COPY --from=builder /app/projects/app/next.config.js /app/projects/app/next.config.js
COPY --from=builder --chown=nextjs:nodejs /app/projects/app/.next/standalone /app/
COPY --from=builder --chown=nextjs:nodejs /app/projects/app/.next/static /app/projects/app/.next/static

# copy server chunks

COPY --from=builder --chown=nextjs:nodejs /app/projects/app/.next/server/chunks /app/projects/app/.next/server/chunks

# copy worker

COPY --from=builder --chown=nextjs:nodejs /app/projects/app/.next/server/worker /app/projects/app/.next/server/worker

# copy standload packages

COPY --from=maindeps /app/node_modules/tiktoken ./node_modules/tiktoken
RUN rm -rf ./node_modules/tiktoken/encoders
COPY --from=maindeps /app/node_modules/@zilliz/milvus2-sdk-node ./node_modules/@zilliz/milvus2-sdk-node

# copy package.json to version file

COPY --from=builder /app/projects/app/package.json ./package.json

# copy config

COPY ./projects/app/data /app/data
RUN chown -R nextjs:nodejs /app/data

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=3000
ENV NEXT_PUBLIC_BASE_URL=$base_url

EXPOSE 3000

USER nextjs

ENV serverPath=./projects/app/server.js

ENTRYPOINT ["sh","-c","node --max-old-space-size=4096 ${serverPath}"]
```| **第三阶段：运行环境 (runner)**<br><br>1. 基础镜像：继续使用`node:20.14.0-alpine`<br>2. 用户设置：<br>   - 创建 `nodejs`用户组 (GID: 1001)<br>   - 创建`nextjs`用户 (UID: 1001)<br>3. 系统配置：<br>   - 配置软件源（如果需要）<br>   - 安装`curl`和`ca-certificates`工具<br>4. 复制构建产物：<br>   - 从`builder`阶段复制：<br>     -`public`静态文件目录<br>     -`next.config.js`配置文件<br>     -`.next/standalone`独立运行文件<br>     -`.next/static`静态资源<br>     -`.next/server/chunks`服务器代码块<br>     -`.next/server/worker`工作进程文件<br>5. 复制特定依赖：<br>   - 从`maindeps`阶段复制：<br>     -`tiktoken`模块（删除 encoders 目录）<br>     -`@zilliz/milvus2-sdk-node`模块<br>6. 复制配置文件：<br>   - 复制`package.json`作为版本文件<br>   - 复制`data`目录并设置权限<br>7. 环境变量设置：<br>   -`NODE_ENV=production`<br>   - 禁用 Next.js 遥测<br>   - 设置端口为 3000<br>   - 设置基础 URL<br>8. 容器配置：<br>   - 暴露 3000 端口<br>   - 切换到 `nextjs`用户运行<br>   - 设置入口点为`server.js`，使用 4GB 内存限制 |

这个 Dockerfile 采用了多阶段构建的方式，可以显著减小最终镜像的大小，同时保证了构建过程的清晰和可维护性。每个阶段都有其特定的职责，从依赖安装到应用构建，最后到运行环境的准备。
