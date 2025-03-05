---
title: 索引原理
date: 2025-03-09 22:04:08
tags:
---

索引的核心作用是通过特定的数据结构快速定位数据，从而减少查询时需要扫描的数据量。

# 创建索引能提升查询速度？

### 避免全表扫描（Full Table Scan）​：
如果没有索引，执行 SELECT * FROM table WHERE column = value 时，数据库需要逐行扫描整个表（全表扫描），时间复杂度为 ​**O(N)**​（N 为数据行数）。
如果存在索引，数据库可以直接定位到符合条件的数据行，时间复杂度降低到 ​**O(log N)**​（如 B-Tree 索引）。

### 减少磁盘 I/O：

索引通常存储在独立的磁盘结构中，且体积远小于表数据。
通过索引快速定位到目标数据页，减少需要读取的磁盘块数量。

### ​优化排序和连接操作：

索引本身是有序的（如 B-Tree 索引），可以避免 ORDER BY 或 GROUP BY 时的显式排序。
在表连接（JOIN）时，索引能加速关联条件的匹配。

# 索引的原理：以 B-Tree 为例
PostgreSQL 默认使用 **B-Tree（Balanced Tree）** 索引，它是一种高效的平衡树结构，适合范围查询和等值查询。

#### **1. B-Tree 的结构**
• **节点分层**：
  • **根节点（Root）**：树的顶层入口。
  • **内部节点（Internal）**：存储键值和指向子节点的指针。
  • **叶子节点（Leaf）**：存储键值和指向实际数据行（Heap）的指针（TID）。
  
• **平衡性**：
  • 所有叶子节点到根节点的路径长度相同，保证查询效率稳定。

#### **2. 查找过程**
假设执行 `SELECT * FROM users WHERE id = 42`：
1. **从根节点开始**，比较键值，找到下一层子节点。
2. **递归向下**，直到找到叶子节点。
3. **从叶子节点获取数据行的物理位置（TID）**，直接读取目标数据页。

#### **3. 范围查询**
对于 `WHERE id BETWEEN 10 AND 20`：
• B-Tree 可以快速定位到键值 10 的位置，然后顺序遍历叶子节点直到键值 20。

#### **4. 维护索引**
• **插入/删除数据**：调整树的结构以保持平衡。
• **更新数据**：如果索引列被修改，需要更新索引中的键值。

---

### **三、其他索引类型及其原理**
PostgreSQL 支持多种索引类型，适用于不同场景：

#### **1. 哈希索引（Hash Index）**
• **原理**：对索引列计算哈希值，直接映射到存储位置。
• **适用场景**：等值查询（`=`），但不支持范围查询。
• **示例**：
  ```sql
  CREATE INDEX idx_name ON table USING HASH (column);
  ```

#### **2. GiST（Generalized Search Tree）**
• **原理**：支持自定义数据类型的索引（如地理坐标、全文检索）。
• **适用场景**：复杂查询（如 `&&` 判断几何图形相交）。
• **示例**：
  ```sql
  CREATE INDEX idx_gist ON table USING GIST (geo_column);
  ```

#### **3. GIN（Generalized Inverted Index）**
• **原理**：存储键值到行的映射，适合多值类型（如数组、JSONB）。
• **适用场景**：包含操作（`@>`、`<@`）或全文检索。
• **示例**：
  ```sql
  CREATE INDEX idx_gin ON table USING GIN (jsonb_column);
  ```

#### **4. BRIN（Block Range Index）**
• **原理**：按数据块（Block）范围存储统计信息（如最小值、最大值）。
• **适用场景**：有序大数据表的范围查询。
• **示例**：
  ```sql
  CREATE INDEX idx_brin ON table USING BRIN (timestamp_column);
  ```

---

### **四、索引的代价**
虽然索引能加速查询，但需要付出额外成本：
1. **存储空间**：索引需要占用磁盘空间。
2. **写操作性能**：
   • 插入、更新、删除数据时，需要同步更新索引。
   • 对写频繁的表，索引过多可能导致性能下降。
3. **维护成本**：索引需要定期 `VACUUM` 或 `REINDEX` 以清理无效数据。

---

### **五、索引的最佳实践**
1. **选择性高的列**：
   • 如果某列的唯一值比例高（如用户 ID），索引效果更显著。
   • 计算选择性：
     ```sql
     SELECT COUNT(DISTINCT column) / COUNT(*) FROM table;
     -- 值越接近 1，选择性越高。
     ```

2. **复合索引（多列索引）**：
   • 对多列查询（如 `WHERE a=1 AND b=2`）有效。
   • 注意列的顺序，优先将高选择性列放在前面。
   • 示例：
     ```sql
     CREATE INDEX idx_ab ON table (a, b);
     ```

3. **避免冗余索引**：
   • 如果已有索引 `(a, b)`，单独的 `(a)` 索引是冗余的。

4. **监控索引使用情况**：
   ```sql
   -- 查看索引使用频率
   SELECT indexrelname, idx_scan FROM pg_stat_user_indexes;
   ```

5. **使用覆盖索引（Index-Only Scan）**：
   • 如果查询的列全部在索引中，可以直接从索引返回数据，无需访问表。
   • 示例：
     ```sql
     CREATE INDEX idx_covering ON table (a) INCLUDE (b, c);
     -- 查询 SELECT b, c FROM table WHERE a = 1 可以直接使用索引。
     ```

---

### **六、示例：索引如何加速查询**
#### **场景**
表 `orders` 有 1000 万行数据，执行以下查询：
```sql
SELECT * FROM orders WHERE user_id = 42 AND status = 'shipped';
```

#### **无索引**
• 需要全表扫描，检查每一行的 `user_id` 和 `status`。

#### **有复合索引**
```sql
CREATE INDEX idx_orders_user_status ON orders (user_id, status);
```
• 数据库直接通过索引定位到 `user_id=42` 且 `status='shipped'` 的数据行，无需扫描全表。

---

### **七、总结**
• **索引的本质**：通过高效的数据结构（如 B-Tree、哈希）快速定位数据。
• **核心优势**：减少磁盘 I/O 和 CPU 计算量，避免全表扫描。
• **适用场景**：高频查询的列、高选择性列、排序和连接操作。
• **注意事项**：权衡查询加速与写操作成本，合理设计索引。