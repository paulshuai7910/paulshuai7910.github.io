---
title: FastGPT
date: 2025-04-02 12:53:20
tags: eventloop nodejs
---

## 底层核心

用户提出问题，在向量数据库中做向量检索，检索：Retrieve
检索匹配到的知识内容发送给 AI，作为背景知识，叫做增强：Augment
AI 回答之前并不知道的问题，叫做生成：Generation
RAG：检索（Retrieve） 增强（Augment） 生成（Generation）

## 本地运行

### 模式一：docker 运行 fastgpt

notes: 如果 docker 有相关 imanges、volumes、contains，需要彻底删除。

    - mkdir <myFolder>
    - cd <myFolder>

```bash
   curl -O https://raw.githubusercontent.com/labring/FastGPT/main/projects/app/data/config.json

# pgvector 版本(测试推荐，简单快捷)

curl -o docker-compose.yml https://raw.githubusercontent.com/labring/FastGPT/main/deploy/docker/docker-compose-pgvector.yml

```

    - run: docker-compose up -d
    - open: http://localhost:300

### 模式二：本地运行 fastGPT，docker 运行数据库以及代理镜像

notes: 如果 docker 有相关 imanges、volumes、contains，需要彻底删除。

- pull docker-compose file
- 注释 fastGPT、sandbox
- 解注释 mongo、pg 端口号内容
- run: docker-compose up -d
