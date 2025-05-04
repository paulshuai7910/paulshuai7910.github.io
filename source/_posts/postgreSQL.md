---
title: postgreSQL
date: 2025-05-03 14:23:01
tags: database postgresql
---

通过 DBeaver 连接 PostgreSQL 后，左侧导航栏的目录结构会根据 **服务端对象** 和 **系统元数据** 自动组织。你提到的 **「数据库」「管理员」「系统信息」** 是 DBeaver 默认的分组逻辑，具体含义和展开后的目录如下：

---

### **1. 数据库（Databases）**

• **含义**：所有用户创建的数据库（包含默认的 `postgres` 数据库）。
• **展开层级**：

```plaintext
→ 数据库名（如 postgres）
  → 模式（Schemas）
    → public（默认模式）
      → 表（Tables）
        → 表名（如 users）
          → 列（Columns）
          → 索引（Indexes）
          → 约束（Constraints）
          → 触发器（Triggers）
      → 视图（Views）
      → 函数（Functions）
      → 序列（Sequences）
  → 外部表（Foreign Tables）
  → 物化视图（Materialized Views）
```

---

### **2. 管理员（Administration）**

• **含义**：与数据库管理相关的系统对象（用户、角色、权限、表空间等）。
• **展开层级**：

```plaintext
→ 用户与角色（Users & Roles）
  → 角色名（如 postgres）
    → 权限（Privileges）
    → 所属成员（Member Of）
→ 表空间（Tablespaces）
  → 表空间名（如 pg_default）
→ 权限（Privileges）
  → 全局权限配置
→ 数据库连接（Database Links）
```

---

### **3. 系统信息（System Information）**

• **含义**：PostgreSQL 内部的系统元数据（系统表、扩展、配置等）。
• **展开层级**：

```plaintext
→ 系统表（System Tables）
  → pg_catalog（核心系统表）
    → pg_tables（所有表信息）
    → pg_indexes（索引信息）
    → pg_roles（角色信息）
  → information_schema（标准信息模式）
    → tables（表元数据）
    → columns（列元数据）
→ 扩展（Extensions）
  → 已安装扩展（如 pgvector、postgis）
→ 服务器配置（Server Config）
  → postgresql.conf 参数（如 max_connections）
→ 系统函数（System Functions）
  → 内置函数（如 pg_stat_activity()）
```

---

### **关键区别**

| **目录组**   | **核心内容**               | **典型操作场景**             |
| ------------ | -------------------------- | ---------------------------- |
| **数据库**   | 用户数据（表、视图、函数） | 业务数据查询、DDL 操作       |
| **管理员**   | 权限、用户、表空间         | 权限分配、存储管理           |
| **系统信息** | 元数据、系统表、配置       | 性能监控、故障排查、扩展管理 |

---

### **扩展说明**

• **模式（Schemas）**：类似命名空间，用于组织表、视图等对象。默认的 `public` 模式是用户数据的存储位置。
• **系统表（pg_catalog）**：存储数据库引擎的核心元数据，例如 `pg_class` 记录所有表和索引，`pg_attribute` 记录列信息。
• **扩展（Extensions）**：通过 `CREATE EXTENSION` 安装的功能插件（如 `pgvector` 提供向量检索）。

通过此目录结构，DBeaver 将 **用户数据**、**管理对象** 和 **系统元数据** 清晰分类，方便开发者快速定位和操作目标对象。
