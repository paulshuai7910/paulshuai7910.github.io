---
title: Docker config
date: 2024-04-03 15:45:31
tags:
---

# Dockerfile

如果是通过自定义镜像的方式安装数据库，可能会有 Dockerfile 文件。它通常包含以下内容：

- 基础镜像选择：指定一个基础操作系统镜像作为起点，例如 FROM ubuntu:latest 等。
- 安装数据库软件：通过运行一系列命令来安装数据库软件，如 RUN apt-get update && apt-get install -y mysql-server（以 MySQL 为例）。
- 配置数据库：可以设置数据库的初始配置，如设置密码、字符集等。
- 暴露端口：EXPOSE 3306（MySQL 默认端口为例），以便外部可以访问数据库。

# docker-compose.yml （如果使用 Docker Compose 管理服务）

## 服务定义：

- 定义数据库服务，如：

```yml
services:
  db:
    image: your_database_image
    environment:
      - MYSQL_ROOT_PASSWORD=your_password
    ports:
      - "3306:3306"
```

- 可以设置环境变量来配置数据库，如设置密码、数据库名称等。
- 网络配置：可以定义网络，使得不同的服务可以在同一个网络中通信。
- 卷挂载：如果需要持久化数据，可以挂载外部存储卷，例如：

````yml
   volumes:
     - db_data:/var/lib/mysql
     ```
````
