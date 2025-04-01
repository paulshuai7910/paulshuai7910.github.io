---
title: 终端代理解决Docker镜像拉取失败问题：Mac环境实战指南
date: 2025-04-01 22:48:15
tags:
---

## 问题背景

在使用 Mac 电脑进行 Docker 开发时，执行`docker-compose up -d`命令时遇到了如下错误：

```
pg Error                     context canceled                                                                                                                                                                                                                           10.1s
✘ mongo Error                  context canceled                                                                                                                                                                                                                           10.1s
✘ aiproxy_pg Error             Get "https://registry-1.docker.io/v2/": net/http: TLS handshake timeout                                                                                                                                                                    10.1s
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: TLS handshake timeout
```

虽然 Mac 已经配置了科学上网工具，但 Docker 仍然无法正常拉取镜像。本文将详细介绍如何通过终端代理解决这个问题。

## 问题分析

1. **VPN 代理的局限性**：
   • 大多数 VPN（如 OpenVPN、WireGuard、Shadowsocks）默认只代理 TCP/UDP 流量
   • `ping`使用的是 ICMP 协议，不受 VPN 影响（这也是为什么`ping google.com`不可靠的原因）

2. **Docker 的特殊性**：
   • Docker 守护进程默认不会继承系统的代理设置
   • Docker 镜像拉取使用的是 HTTPS 协议（TCP），需要正确配置代理

## 解决方案

### 第一步：查找 VPN 代理端口

1. 打开 Mac 系统设置
2. 找到当前已连接的网络
3. 点击"详细信息"→"代理"
4. 记下 HTTP 代理端口（本例中为 15236）

![](issue.png)

### 第二步：配置终端代理

```bash
# 设置HTTP/HTTPS代理
export http_proxy=http://127.0.0.1:15236
export https_proxy=http://127.0.0.1:15236

# 验证代理是否生效（不要使用ping）
curl -vI https://registry-1.docker.io
```

### 第三步：配置 Docker 守护进程代理（可选）

如果仅设置终端代理无效，可能需要为 Docker 守护进程配置代理：

```bash
# 创建代理配置目录
sudo mkdir -p /etc/systemd/system/docker.service.d

# 创建代理配置文件
sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:15236"
Environment="HTTPS_PROXY=http://127.0.0.1:15236"
EOF

# 重新加载并重启Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 第四步：重新执行命令

```bash
docker-compose up -d
```

## 验证与测试

1. **检查镜像拉取状态**：

   ```bash
   docker images
   docker ps -a
   ```

2. **查看容器日志**：
   ```bash
   docker logs <container_name>
   ```

## 替代方案

如果上述方法仍然无效，可以考虑：

1. **使用国内镜像源**：

   ```bash
   # 编辑Docker配置
   sudo nano /etc/docker/daemon.json
   ```

   添加内容：

   ```json
   {
     "registry-mirrors": [
       "https://docker.mirrors.ustc.edu.cn",
       "https://hub-mirror.c.163.com"
     ]
   }
   ```

2. **手动下载镜像**：

   ```bash
   # 在可访问的机器上
   docker pull <image_name>
   docker save <image_name> > image.tar

   # 在目标机器上
   docker load < image.tar
   ```

## 总结

通过正确配置终端代理和 Docker 守护进程代理，我们成功解决了 Mac 环境下 Docker 镜像拉取失败的问题。关键点包括：

1. 理解 VPN 代理的局限性
2. 正确识别代理端口
3. 为 Docker 配置适当的代理设置
4. 使用可靠的验证方法（而非 ping）

希望本文能帮助遇到类似问题的开发者顺利解决问题。如果仍有疑问，可以参考 Docker 官方文档或相关社区讨论。
