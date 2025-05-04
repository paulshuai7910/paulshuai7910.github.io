---
title: prisma 使用
date: 2025-04-25 10:52:20
tags: database prisma
---

# prisma 使用

## 安装

```bash
npm install prisma @prisma/client
```

## 初始化

```bash
npx prisma init
```

这个命令创建了一个名为 prisma 的新目录，其中包含一个名为 schema.prisma 的文件和一个位于项目根目录中的.env 文件。
schema.prisma 包含 prisma 模式以及数据库连接和 prisma 客户端生成器。

## 连接你的数据库

为了连接你的数据库，你需要在你的 Prisma 模式中设置 datasource 块的 url 字段到你的数据库连接 URL:

```bash
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

url 是通过环境变量设置，它在.env 中定义:

```bash
DATABASE_URL="postgresql://username:password@host:port/database"
```

## 安装 Prisma Client

```bash
npm install @prisma/client
```

注意，安装命令会自动调用 prisma generate，它会读取 prisma 模式并生成一个适合您的模型的 prisma Client 版本

## 创建模型

```bash
npx prisma generate
```

## 运行

```bash
npx prisma studio
```
