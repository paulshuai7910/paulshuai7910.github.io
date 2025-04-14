---
title: Mongoose Schema 字段类型
date: 2025-04-09 14:40:27
tags:
---

在 **Mongoose** 中，`Schema` 用于定义 MongoDB 文档的结构，支持多种数据类型。

---

### **1. 基本数据类型**

| 类型       | 说明                   | 示例                                               |
| ---------- | ---------------------- | -------------------------------------------------- |
| `String`   | 字符串                 | `name: { type: String }`                           |
| `Number`   | 数字（整数或浮点数）   | `age: { type: Number }`                            |
| `Boolean`  | 布尔值                 | `isActive: { type: Boolean }`                      |
| `Date`     | 日期                   | `createdAt: { type: Date }`                        |
| `Buffer`   | 二进制数据（如文件）   | `file: { type: Buffer }`                           |
| `ObjectId` | MongoDB 的 `_id` 字段  | `userId: { type: mongoose.Schema.Types.ObjectId }` |
| `Array`    | 数组                   | `tags: [String]`                                   |
| `Mixed`    | 任意类型（无类型检查） | `metadata: { type: mongoose.Schema.Types.Mixed }`  |

---

### **2. 特殊类型**

| 类型                    | 说明                | 示例                                                   |
| ----------------------- | ------------------- | ------------------------------------------------------ |
| `Decimal128`            | 高精度浮点数        | `price: { type: mongoose.Schema.Types.Decimal128 }`    |
| `Map`                   | 键值对（ES6 `Map`） | `attributes: { type: Map, of: String }`                |
| `Schema.Types.ObjectId` | 引用其他文档        | `author: { type: Schema.Types.ObjectId, ref: 'User' }` |

---

### **3. 嵌套 Schema**

可以定义嵌套的子文档结构：

```javascript
const addressSchema = new mongoose.Schema({
  street: String,
  city: String,
  country: String,
})

const userSchema = new mongoose.Schema({
  name: String,
  address: addressSchema, // 嵌套 Schema
})
```

---

### **4. 数组类型**

可以定义数组，并指定数组元素的类型：

```javascript
const postSchema = new mongoose.Schema({
  title: String,
  tags: [String], // 字符串数组
  comments: [{ body: String, date: Date }], // 对象数组
})
```

---

### **5. 自定义验证**

可以添加验证规则：

```javascript
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    match: /^\S+@\S+\.\S+$/, // 正则验证
  },
  age: {
    type: Number,
    min: 18,
    max: 100,
  },
})
```

---

### **6. 默认值和自动生成**

可以设置默认值或自动生成字段：

```javascript
const postSchema = new mongoose.Schema({
  title: String,
  views: { type: Number, default: 0 },
  createdAt: { type: Date, default: Date.now },
})
```

---

### **总结**

Mongoose 提供了丰富的字段类型，包括：
• **基本类型**：`String`, `Number`, `Boolean`, `Date`, `Buffer`, `Array`, `ObjectId`, `Mixed`
• **特殊类型**：`Decimal128`, `Map`
• **嵌套 Schema** 和 **数组类型**
• **自定义验证**（`required`, `min`, `max`, `match`）
• **默认值** 和 **自动生成字段**
