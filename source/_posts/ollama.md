---
title: ollama 本地大语言模型框架
date: 2025-04-30 23:45:26
tags: ollama
---

### **Ollama 本地大语言模型框架**

一个轻量级、可扩展的开源框架，支持在本地计算机运行大型语言模型（LLM）

---

### **快速开始**

#### 安装方法

```bash
# macOS/Linux 一键安装
curl -fsSL https://ollama.com/install.sh | sh
```

#### 基础使用

```bash
# 运行示例模型（如llama3.2）
ollama run llama3.2
```

---

### **模型库**

| 模型名称                 | 参数量 | 所需内存 | 下载命令                         |
| ------------------------ | ------ | -------- | -------------------------------- |
| Llama3.2                 | 3B     | 2.0GB    | `ollama run llama3.2`            |
| Gemma3                   | 27B    | 17GB     | `ollama run gemma3:27b`          |
| DeepSeek-R1              | 671B   | 404GB    | `ollama run deepseek-r1:671b`    |
| Mistral                  | 7B     | 4.1GB    | `ollama run mistral`             |
| 视觉模型 Llama3.2-Vision | 90B    | 55GB     | `ollama run llama3.2-vision:90b` |

> **硬件要求建议**：
>
> - 7B 模型：至少 8GB 内存
> - 13B 模型：至少 16GB 内存
> - 33B+模型：32GB+内存

---

### **自定义模型**

#### 1. 导入 GGUF 格式模型

```bash
# 创建Modelfile
echo "FROM ./my_model.gguf" > Modelfile

# 创建自定义模型
ollama create mymodel -f Modelfile

# 运行模型
ollama run mymodel
```

#### 2. 角色定制（示例：马里奥角色）

```modelfile
FROM llama3.2
PARAMETER temperature 1  # 创造性参数（1为较高）
SYSTEM "你是超级马里奥兄弟中的马里奥，请始终以马里奥的身份回答"
```

创建命令：

```bash
ollama create mario -f Modelfile
ollama run mario
```

---

### **核心 CLI 命令**

| 命令                          | 功能描述           |
| ----------------------------- | ------------------ |
| `ollama list`                 | 列出本地所有模型   |
| `ollama ps`                   | 查看正在运行的模型 |
| `ollama stop <模型名>`        | 停止运行中的模型   |
| `ollama show <模型名>`        | 显示模型详细信息   |
| `ollama cp <源模型> <新名称>` | 复制模型副本       |

---

### **REST API 接口**

#### 基础请求示例

```bash
# 生成文本
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "为什么天空是蓝色的？"
}'

# 多轮对话
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {"role": "user", "content": "如何煮意大利面？"}
  ]
}'
```

---

### **生态集成**

#### 客户端工具

• **桌面端**：Ellama（多平台客户端）、SwiftChat（苹果全系支持）
• **移动端**：Ollama Android Chat（一键启动）、Reins（参数调优）
• **编辑器插件**：VSCode 扩展、Qt Creator 插件
• **浏览器扩展**：网页摘要工具、写作助手

#### 数据库支持

• PostgreSQL 向量库：`pgai`扩展
• MindsDB：连接 200+数据源

#### 监控调试

• Opik：LLM 应用调试平台
• MLflow Tracing：可视化追踪模型表现

---

### **高级功能**

#### 多模态处理

```bash
# 图像分析（需llava模型）
ollama run llava "图片内容是什么？/path/to/image.png"
```

#### 文档处理

```bash
# 文件内容摘要
ollama run llama3.2 "总结此文件：$(cat report.txt)"
```

---

### **开发者构建**

```bash
# 本地编译运行
./ollama serve          # 启动服务
./ollama run llama3.2   # 新终端运行模型
```

完整开发文档参考：[GitHub 仓库](https://github.com/ollama/ollama)

---

此版本文档采用模块化结构，重点信息使用表格/代码块突出显示，删减了冗余的技术术语，增加中文场景示例，便于快速理解核心功能和使用方法。
