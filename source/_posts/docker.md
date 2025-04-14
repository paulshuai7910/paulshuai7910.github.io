---
title: Docker config
date: 2024-04-03 15:45:31
tags: docker
---

# Dockerfile

Dockerfile 是用于定义一个 Docker 镜像（Image）的文本文件。它包含了从基础镜像开始，逐步构建和配置该镜像所需要的所有步骤和指令。它描述了如何构建一个容器镜像，并为该镜像提供了具体的环境配置。

作用：
定义基础镜像：指定镜像的起始点，例如使用官方的 Node.js、Python、Java 等镜像作为基础。
安装依赖：在镜像中安装需要的软件包或依赖项。
复制项目文件：将本地的项目文件复制到容器中。
暴露端口：指定容器将要开放的端口，使其可以与外部通信。
指定启动命令：定义容器启动时的命令或进程。

如果是通过自定义镜像的方式安装数据库，可能会有 Dockerfile 文件。它通常包含以下内容：

- 基础镜像选择：指定一个基础操作系统镜像作为起点，例如 FROM ubuntu:latest 等。
- 安装数据库软件：通过运行一系列命令来安装数据库软件，如 RUN apt-get update && apt-get install -y mysql-server（以 MySQL 为例）。
- 配置数据库：可以设置数据库的初始配置，如设置密码、字符集等。
- 暴露端口：EXPOSE 3306（MySQL 默认端口为例），以便外部可以访问数据库。

# docker-compose.yml （如果使用 Docker Compose 管理服务）

docker-compose.yml 是一个 YAML 文件，用于定义和运行多个 Docker 容器应用。它通过声明不同的服务、网络、卷等来帮助我们方便地配置和启动一个由多个容器组成的应用环境。

作用：
管理多个容器：当项目包含多个服务（如前端、后端、数据库等）时，docker-compose.yml 可以将它们统一配置和管理。
简化部署和启动：通过一个命令 docker-compose up 就能启动整个应用的所有服务，无需手动管理每个容器。
环境配置：通过 docker-compose.yml，可以轻松配置服务的端口、环境变量、依赖关系、网络等。

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

# 关系

Dockerfile 主要用于 构建容器镜像，它定义了单个容器的配置。
docker-compose.yml 主要用于 管理多个容器服务，它定义了如何运行多个容器，并且可以让你通过一个命令启动或停止所有相关的容器。

# 协作

在项目中，通常是这样的：

使用 Dockerfile 来定义如何构建每个服务的 Docker 镜像。
使用 docker-compose.yml 来定义和管理多个服务，甚至定义它们之间的依赖关系。

# 常用命令

### 1. 容器生命周期管理

#### 启动/停止容器

```bash
docker start <容器名/ID>          # 启动已停止的容器
docker stop <容器名/ID>           # 停止运行中的容器（优雅停止）
docker restart <容器名/ID>        # 重启容器
docker kill <容器名/ID>           # 强制停止容器
docker pause <容器名/ID>         # 暂停容器
docker unpause <容器名/ID>       # 恢复暂停的容器
```

#### 创建/删除容器

```bash
docker create <镜像名>            # 创建容器但不启动
docker run <镜像名>               # 创建并启动容器
docker rm <容器名/ID>             # 删除已停止的容器
docker rm -f <容器名/ID>          # 强制删除运行中的容器
docker container prune           # 删除所有停止的容器
```

### 2. 容器操作

#### 查看容器

```bash
docker ps                       # 查看运行中的容器
docker ps -a                    # 查看所有容器（包括停止的）
docker ps -q                    # 只显示容器ID
docker ps --filter "status=exited"  # 过滤特定状态的容器
```

#### 进入容器

```bash
docker exec -it <容器名/ID> /bin/bash   # 进入运行中的容器
docker attach <容器名/ID>               # 附加到运行中的容器
```

#### 容器日志

```bash
docker logs <容器名/ID>          # 查看容器日志
docker logs -f <容器名/ID>       # 实时查看日志（类似tail -f）
docker logs --tail 100 <容器名/ID> # 查看最后100行日志
```

### 3. 镜像管理

#### 查看镜像

```bash
docker images                   # 列出本地镜像
docker images -a                # 列出所有镜像（包括中间层）
docker image ls                 # 同上（新语法）
```

#### 镜像操作

```bash
docker pull <镜像名:标签>        # 拉取镜像
docker push <镜像名:标签>        # 推送镜像到仓库
docker rmi <镜像名/ID>          # 删除镜像
docker image prune              # 删除未被使用的镜像
docker build -t <标签> .        # 构建镜像（当前目录Dockerfile）
```

### 4. 网络管理

```bash
docker network ls               # 列出所有网络
docker network inspect <网络名>  # 查看网络详情
docker network create <网络名>   # 创建自定义网络
docker network connect <网络名> <容器名> # 连接容器到网络
docker network disconnect <网络名> <容器名> # 从网络断开容器
```

### 5. 数据卷管理

```bash
docker volume ls                # 列出数据卷
docker volume create <卷名>      # 创建数据卷
docker volume inspect <卷名>     # 查看数据卷详情
docker volume rm <卷名>          # 删除数据卷
docker volume prune             # 删除未使用的数据卷
```

### 6. 系统信息与维护

```bash
docker info                     # 显示Docker系统信息
docker version                  # 显示Docker版本信息
docker stats                    # 显示容器资源使用统计
docker top <容器名>              # 查看容器内运行的进程
docker system df                # 查看磁盘使用情况
docker system prune             # 清理所有未使用的对象
```

### 7. Docker Compose 常用命令

```bash
docker-compose up               # 启动所有服务
docker-compose up -d            # 后台启动服务
docker-compose down             # 停止并移除所有容器
docker-compose ps               # 列出所有服务容器
docker-compose logs             # 查看服务日志
docker-compose build            # 构建或重新构建服务
docker-compose restart          # 重启所有服务
```

### 8. 实用组合命令

```bash
# 删除所有停止的容器和未使用的镜像
docker system prune -a

# 批量停止所有运行中的容器
docker stop $(docker ps -q)

# 批量删除所有容器（包括运行中的）
docker rm -f $(docker ps -aq)

# 查看容器资源使用情况（动态更新）
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# 导出容器为镜像
docker export <容器ID> > container.tar

# 导入镜像
docker import container.tar new-image:tag
```

### 9. 容器与宿主机文件操作

```bash
docker cp <容器名>:<容器路径> <宿主机路径>  # 从容器复制文件到宿主机
docker cp <宿主机路径> <容器名>:<容器路径>  # 从宿主机复制文件到容器
```

这些命令覆盖了 Docker 日常使用的大部分场景，建议收藏备用！根据实际需求选择合适的命令组合使用。
