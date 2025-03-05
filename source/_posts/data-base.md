---
title: database 数据库
date: 2024-09-03 11:42:13
tags:
---

# 实现分页

## 确定分页参数

页面大小（pageSize）：表示每页显示的数据条数。
当前页码（pageNumber）：表示当前要显示的页面编号。

## 使用 skip 和 limit 方法（MongoDB）

skip 方法用于跳过指定数量的文档，limit 方法用于限制返回的文档数量。
例如，要获取第 2 页，每页显示 10 条数据：
不同的 NoSQL 数据库可能有不同的方法来实现分页，但基本思路是类似的，即通过跳过一定数量的文档并限制返回的文档数量来实现分页效果。

# edge 表查询

边（edge）表用于表示两个节点（vertex）之间的关系。通过查询边表，可以获取节点之间的关系信息。这种查询的核心原理是根据指定的条件，从边表中筛选出符合要求的边，并进一步关联节点来获取结果。
边表通常包含以下关键字段：

- Edge ID：边的唯一标识符。
- From Vertex：边的起始节点 ID。
- To Vertex：边的目标节点 ID。
- Edge Type：边的类型，表示关系的种类（例如“朋友”、“喜欢”、“工作于”等）。
- Attributes：边的属性，可以包含时间戳、权重、描述等。

# 事务

数据库的事务（Transaction）是指在数据库管理系统中，一组作为单个工作单元执行的操作。事务具有四个重要的性质，简称为 ACID 属性（Atomicity、Consistency、Isolation、Durability）。通过事务，数据库能够确保数据的一致性、完整性，并且能够正确处理并发操作和系统故障。

## 事务的四个 ACID 性质

### 原子性（Atomicity）

- 原子性意味着事务中所有的操作要么全部执行成功，要么全部不执行，如果事务中某一步失败了，系统会回滚到事务开始之前的状态，保证数据的一致性。
- 例子：银行转账事务，如果 A 向 B 转账 $100，那么必须保证从 A 的账户扣除 $100 并且 B 的账户增加 $100 这两个操作都成功执行。如果扣除成功但增加失败，事务会回滚，A 的账户不会扣款。

### 一致性（Consistency）

- 一致性确保事务在执行前后，数据库都保持一致的状态，事务执行完后，数据库的状态必须满足所有定义的规则（如主键、外键、约束等）。
- 例子： 在数据库中，转账前后的账户余额总和应该保持不变，不能因为某个事务导致总金额的改变。

### 隔离性（Isolation）

- 隔离性保证多个并发事务的执行互不干扰，事务之间是相互隔离的。即使有多个事务同时运行，每个事务看到的数据都是一致的，不会被其他事务未提交的中间状态影响。每个事务都有自己的隔离级别，事务之间的隔离级别可以是串行或并行。
- 例子： 当一个用户在更新账户余额时，另一个用户进行查询操作时，应该看不到未提交的部分修改，查询结果要么是事务开始前的值，要么是事务提交后的新值。

### 持久性（Durability）

- 持久性确保事务一旦提交，修改的数据就会永久保存到数据库中，哪怕系统出现故障或崩溃，数据也不会丢失。

例子：如果一个事务成功提交了转账操作，哪怕数据库服务器之后出现故障，已完成的转账记录依然会被保存下来。

## 事务的生命周期

- 开始事务：事务从用户发出的一组 SQL 语句开始。
- 执行操作： 事务内部的 SQL 操作按照顺序执行。
- 提交事务： 事务成功完成所有操作后，将数据写入数据库。
- 回滚事务： 如果事务中出现错误，或者用户主动回滚事务，事务将被撤销，数据恢复到事务开始之前的状态。

## 事务的隔离级别：

不同的隔离级别影响事务之间的并发控制和数据可见性。常见的隔离级别包括：

- 读未提交（Read Uncommitted）： 事务可以读取其他未提交事务的数据，这可能会导致脏读问题。
- 读已提交 (Read Committed)：事务只能读取其他已提交事务的数据，避免脏读问题，但可能出现不可重复读，即同一个事务在不同的时间点读取到的值不同。
- 可重复读（Repeatable Read）：事务在读取数据时，会锁定数据，避免了不可重复读问题，但可能出现幻读，即同一个事务在不同的时间点读取到的数据数量不同。
- 可串行化（Serializable）：事务在读取数据时，会锁定数据，避免了所有并发问题，但可能导致性能下降。

## 事务的用途

事务广泛用于需要确保数据一致性和可靠性的操作场景，特别是在涉及多个数据修改的情况下，如银行转账、订单处理、库存管理等。

# ArangoDB

在 ArangoDB 中，事务提供了一种安全操作多个集合的方式，以确保数据的一致性。使用事务可以确保一组操作要么全部执行成功，要么在出错时回滚。ArangoDB 支持 JavaScript API 和 AQL（Arango Query Language）来执行事务操作。

## 使用事务的基本步骤

在 ArangoDB 中，事务主要涉及以下步骤：

指定读写集合：定义需要参与事务的集合。
编写事务函数：定义具体的操作逻辑。
提交或回滚事务：根据操作结果，决定是否提交事务。

## ArangoDB 中的事务语法

JavaScript 事务：通过 ArangoDB 的 JavaScript API，可以在事务中执行多步操作。
AQL 事务：通过 AQL 查询，可以执行一些简单的事务操作。

## Code

```js
const db = require("@arangodb").db

function transferFunds() {
  const fromUserId = "A"
  const toUserId = "B"
  const amount = 100

  // 开始事务，指定要写入的集合
  const trx = db._executeTransaction({
    collections: {
      write: ["accounts"], // 要写入的集合
    },
    action: function () {
      const accounts = db._collection("accounts")

      // 获取用户 A 的账户
      const fromUser = accounts.document(fromUserId)
      if (fromUser.balance < amount) {
        throw new Error("Insufficient balance")
      }

      // 获取用户 B 的账户
      const toUser = accounts.document(toUserId)

      // 更新用户 A 的余额
      accounts.update(fromUserId, { balance: fromUser.balance - amount })

      // 更新用户 B 的余额
      accounts.update(toUserId, { balance: toUser.balance + amount })
    },
  })

  return "Transfer completed"
}

// 执行转账事务
transferFunds()
```

- db.\_executeTransaction: 启动事务，定义所需的读写集合。在这个示例中，accounts 集合是写集合，因为我们会更新账户余额。
- 事务函数 (action)：在这个匿名函数中定义了事务的逻辑：
  - 检查用户 A 的余额是否足够，如果不足，会抛出错误并回滚事务。
  - 更新用户 A 的余额，将转出金额从账户中扣除。
  - 更新用户 B 的余额，将转入金额添加到账户中。
- 回滚与提交：
  - 如果发生任何错误（例如余额不足），事务将自动回滚。
  - 如果操作顺利执行，事务会自动提交。
# 数据库ORM
ORM（Object-Relational Mapping，对象关系映射）​​ 是一种编程技术，用于在 ​面向对象编程语言​（如 Python、Java、C#、JavaScript）和 ​关系型数据库​（如 MySQL、PostgreSQL、Oracle）之间建立桥梁。它通过将数据库中的表（Table）映射为程序中的类（Class）、表的行（Row）映射为对象（Object）、表的列（Column）映射为对象的属性（Property），使开发者能够用面向对象的方式操作数据库，而无需直接编写 SQL 语句。
## ORM 的核心优点
### 提高开发效率
​无需手写 SQL：ORM 自动生成 SQL 语句，开发者只需操作对象，减少重复的数据库操作代码。
​代码简洁：用面向对象的方式（如 user.save()）替代冗长的 SQL 语句（如 INSERT INTO users (name, email) VALUES (...)）。
​快速迭代：在业务逻辑复杂或需求频繁变更时，ORM 能更快适应变化。
### 代码可维护性
​抽象数据库细节：业务逻辑与数据库操作解耦，代码更清晰。
​统一代码风格：所有数据库操作通过 ORM 接口完成，避免 SQL 语句分散在代码各处。
###  跨数据库兼容
​无缝切换数据库：ORM 封装了不同数据库的 SQL 方言，例如从 MySQL 切换到 PostgreSQL 时，只需修改配置，无需重写 SQL 代码。
​支持多种数据库：主流 ORM 框架（如 SQLAlchemy、TypeORM）支持 MySQL、PostgreSQL、SQLite 等。
### 防止 SQL 注入
​参数化查询：ORM 自动对用户输入进行转义，避免拼接 SQL 字符串导致的安全漏洞（如 user.query.filter_by(name=user_input) 是安全的）。
### 高级功能支持
​事务管理：通过简单的 API（如 session.commit()）处理复杂的事务。
​关联关系处理：轻松定义一对多、多对多关系（如 User 和 Order 的关联）。
​数据迁移工具：部分 ORM（如 Django ORM）提供数据库迁移工具，自动生成表结构变更脚本。
### 更好的抽象与复用
​模型复用：通过继承或组合复用模型逻辑（如基类 BaseModel 定义公共字段）。
​查询链式调用：支持链式方法调用构建复杂查询（如 User.query.filter(...).order_by(...).limit(10)）。

### **主流 ORM 框架示例**
| 语言       | ORM 框架               | 特点                              |
|------------|------------------------|-----------------------------------|
| Python     | Django ORM、SQLAlchemy | Django ORM 高度集成，SQLAlchemy 灵活 |
| Java       | Hibernate              | 企业级应用，支持 JPA 标准           |
| JavaScript | TypeORM、Sequelize     | TypeORM 支持 TypeScript，语法优雅   |
| C#         | Entity Framework       | .NET 官方推荐，功能强大             |

---
