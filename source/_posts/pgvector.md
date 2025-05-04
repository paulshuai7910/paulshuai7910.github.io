---
title: pgvector
date: 2025-05-02 21:43:13
tags: ollama
---

PostgreSQL（简称 Postgre）和 pgvector 是密切相关但又有明显区别的两个技术组件，以下是它们的核心区别与联系：

### 核心区别

1. **定位与功能**  
   • **PostgreSQL**：是开源关系型数据库，专注于结构化数据的存储和管理，支持标准 SQL 和 ACID 事务。  
   • **pgvector**：是 PostgreSQL 的扩展插件，专为向量数据设计，支持高维向量的存储、相似性搜索（如余弦距离、欧氏距离）和索引优化（如 HNSW、IVFFlat）。

2. **数据处理能力**  
   • PostgreSQL 原生不支持向量运算，需通过扩展实现。  
   • pgvector 直接提供向量数据类型（如`vector(1536)`）和相似性搜索操作符（如`<=>`余弦距离）。

3. **应用场景**  
   • PostgreSQL 适用于传统业务系统（如订单管理、用户数据）。  
   • pgvector 用于 AI 相关场景（如语义搜索、推荐系统、图像检索）。

### 核心联系

1. **依赖关系**  
   pgvector 必须运行在 PostgreSQL 上，通过扩展机制（`CREATE EXTENSION vector`）集成，继承 PostgreSQL 的事务、安全性和 SQL 接口。

2. **生态互补**  
   • PostgreSQL 的扩展性允许 pgvector 增强其对非结构化数据的处理能力。  
   • pgvector 使 PostgreSQL 能胜任 AI 时代的需求（如 RAG 技术中的向量检索）。

3. **性能结合**  
   pgvector 利用 PostgreSQL 的索引（如 GIN）和并行查询能力，优化向量搜索性能。

### 对比示例

| **维度** | **PostgreSQL**                   | **pgvector**                                                     |
| -------- | -------------------------------- | ---------------------------------------------------------------- |
| 数据类型 | 结构化数据（表、JSON 等）        | 高维向量（如[0.1, 0.2, 0.3]）                                    |
| 典型查询 | `SELECT * FROM users WHERE id=1` | `SELECT * FROM docs ORDER BY embedding <=> '[0.1, 0.2]' LIMIT 5` |
| 索引类型 | B-tree, GIN, BRIN                | IVFFlat, HNSW（近似最近邻搜索）                                  |

### 总结

PostgreSQL 是通用数据库，而 pgvector 是其针对向量数据的专用扩展。两者结合后，PostgreSQL 可同时处理结构化数据和向量搜索，成为 AI 应用的理想数据平台。若需纯向量数据库的高性能，可评估专用方案（如 Milvus），但 PostgreSQL+pgvector 在兼容性和生态整合上更具优势。
