# 工具系统

> 学习目标：掌握核心工具的使用

## 目录

- [工具系统概述](#工具系统概述)
- [文件系统工具](#文件系统工具)
- [执行工具](#执行工具)
- [Web 工具](#web-工具)
- [会话工具](#会话工具)
- [其他工具](#其他工具)
- [工具配置](#工具配置)
- [创建自定义工具](#创建自定义工具)
- [最佳实践](#最佳实践)
- [小结](#小结)

---

## 工具系统概述

### 什么是工具？

工具是 Agent 与外部世界交互的接口，允许 Agent 执行各种操作。

### 工具分类

```
工具系统
├── 文件系统
│   ├── read    - 读取文件
│   ├── write   - 写入文件
│   ├── edit    - 编辑文件
│   └── apply_patch - 应用补丁
├── 执行
│   ├── exec    - 执行命令
│   └── process - 进程管理
├── Web
│   ├── web_search - 网络搜索
│   ├── web_fetch  - 获取网页
│   └── browser    - 浏览器控制
├── 会话
│   ├── sessions_list    - 列出会话
│   ├── sessions_history - 查看历史
│   ├── sessions_send    - 发送消息
│   └── sessions_spawn   - 创建会话
├── AI
│   ├── image  - 图像生成
│   └── pdf    - PDF 处理
├── 通信
│   └── message - 发送消息
├── 调度
│   └── cron    - 定时任务
└── 系统
    ├── gateway - Gateway 控制
    └── nodes   - 节点管理
```

---

## 文件系统工具

### read - 读取文件

读取文件内容。

**参数**：
```typescript
{
  path: string;        // 文件路径
  startLine?: number;  // 起始行号（可选）
  endLine?: number;    // 结束行号（可选）
}
```

**示例**：
```markdown
# Agent 使用 read 工具

用户: "查看 config.json 文件"

Agent:
<tool_call>
{
  "name": "read",
  "parameters": {
    "path": "/path/to/config.json"
  }
}
</tool_call>

<tool_result>
{
  "apiVersion": "v1",
  "kind": "Config",
  "data": {
    "key": "value"
  }
}
</tool_result>
```

**最佳实践**：
- 先检查文件是否存在
- 对于大文件，使用行号范围
- 处理可能的读取错误

### write - 写入文件

创建新文件或覆盖现有文件。

**参数**：
```typescript
{
  path: string;    // 文件路径
  content: string; // 文件内容
}
```

**示例**：
```markdown
用户: "创建一个新的 Python 脚本"

Agent:
<tool_call>
{
  "name": "write",
  "parameters": {
    "path": "/path/to/script.py",
    "content": "#!/usr/bin/env python3\n\ndef hello():\n    print('Hello, World!')\n\nif __name__ == '__main__':\n    hello()\n"
  }
}
</tool_call>
```

**最佳实践**：
- 警告用户会覆盖现有文件
- 确认目录存在
- 使用适当的文件编码

### edit - 编辑文件

修改现有文件的特定部分。

**参数**：
```typescript
{
  path: string;        // 文件路径
  oldText: string;     // 要替换的文本
  newText: string;     // 新文本
}
```

**示例**：
```markdown
用户: "把 print 改成 logging"

Agent:
<tool_call>
{
  "name": "edit",
  "parameters": {
    "path": "/path/to/script.py",
    "oldText": "print('Hello, World!')",
    "newText": "logging.info('Hello, World!')"
  }
}
</tool_call>
```

**最佳实践**：
- 使用足够的上下文使 oldText 唯一
- 验证编辑是否成功
- 保留文件格式和缩进

### apply_patch - 应用补丁

应用 Git 格式的补丁文件。

**参数**：
```typescript
{
  path: string;    // 目标文件路径
  patch: string;   // 补丁内容
}
```

**示例**：
```markdown
用户: "应用这个补丁"

Agent:
<tool_call>
{
  "name": "apply_patch",
  "parameters": {
    "path": "/path/to/file.py",
    "patch": "--- a/file.py\n+++ b/file.py\n@@ -1,3 +1,3 @@\n-def old_function():\n+def new_function():\n     pass"
  }
}
</tool_call>
```

---

## 执行工具

### exec - 执行命令

执行 shell 命令并获取输出。

**参数**：
```typescript
{
  command: string;   // 命令
  args?: string[];   // 参数（可选）
  cwd?: string;      // 工作目录（可选）
  env?: Record<string, string>; // 环境变量（可选）
}
```

**示例**：
```markdown
用户: "列出当前目录的文件"

Agent:
<tool_call>
{
  "name": "exec",
  "parameters": {
    "command": "ls",
    "args": ["-la"]
  }
}
</tool_call>

<tool_result>
total 24
drwxr-xr-x  5 user  staff   160 Jan 15 10:30 .
drwxr-xr-x  3 user  staff    96 Jan 15 10:00 ..
-rw-r--r--  1 user  staff   256 Jan 15 10:30 config.json
-rw-r--r--  1 user  staff  1024 Jan 15 10:25 script.py
</tool_result>
```

**安全考虑**：
- ⚠️ 谨慎使用 exec 工具
- 验证命令安全性
- 避免破坏性操作
- 考虑使用沙箱

**最佳实践**：
```markdown
# AGENTS.md - exec 工具使用指南

## 安全规则

1. **禁止的命令** - 绝对不要执行：
   - rm -rf /
   - dd if=/dev/zero of=/dev/sda
   - :(){ :|:& };:  (fork bomb)
   - 任何格式化磁盘的命令

2. **需要确认的命令**：
   - 删除文件/目录
   - 修改系统配置
   - 安装软件包
   - 重启服务

3. **使用前检查**：
   - 验证命令语法
   - 确认工作目录
   - 检查环境变量
```

### process - 进程管理

管理长时间运行的进程。

**参数**：
```typescript
{
  action: "start" | "stop" | "status" | "logs";
  name?: string;    // 进程名称
  pid?: number;     // 进程 ID
}
```

**示例**：
```markdown
用户: "启动 nginx 服务"

Agent:
<tool_call>
{
  "name": "process",
  "parameters": {
    "action": "start",
    "name": "nginx"
  }
}
</tool_call>

<tool_result>
Process started: nginx (PID: 12345)
</tool_result>
```

---

## Web 工具

### web_search - 网络搜索

搜索网络信息。

**参数**：
```typescript
{
  query: string;         // 搜索查询
  numResults?: number;   // 结果数量（默认 10）
  language?: string;     // 语言（默认 en）
}
```

**示例**：
```markdown
用户: "搜索最新的 Python 版本"

Agent:
<tool_call>
{
  "name": "web_search",
  "parameters": {
    "query": "Python latest version 2024",
    "numResults": 5
  }
}
</tool_call>

<tool_result>
1. Python 3.12.0 released
   https://www.python.org/downloads/release/python-3120/

2. What's new in Python 3.12
   https://docs.python.org/3.12/whatsnew/3.12.html

...
</tool_result>
```

**最佳实践**：
- 使用精确的搜索关键词
- 利用搜索操作符
- 评估结果可信度
- 引用信息来源

### web_fetch - 获取网页

获取并解析网页内容。

**参数**：
```typescript
{
  url: string;           // URL
  format?: "html" | "markdown" | "text"; // 输出格式
}
```

**示例**：
```markdown
用户: "获取那篇文档的内容"

Agent:
<tool_call>
{
  "name": "web_fetch",
  "parameters": {
    "url": "https://example.com/docs/tutorial.html",
    "format": "markdown"
  }
}
</tool_call>

<tool_result>
# Tutorial

Welcome to our tutorial. This guide will help you...
</tool_result>
```

**最佳实践**：
- 检查 URL 有效性
- 处理大页面
- 尊重 robots.txt
- 处理编码问题

### browser - 浏览器控制

控制无头浏览器进行复杂交互。

**参数**：
```typescript
{
  action: "goto" | "click" | "fill" | "screenshot";
  url?: string;
  selector?: string;
  value?: string;
}
```

**示例**：
```markdown
用户: "帮我截图那个网站"

Agent:
<tool_call>
{
  "name": "browser",
  "parameters": {
    "action": "goto",
    "url": "https://example.com"
  }
}
</tool_call>

<tool_call>
{
  "name": "browser",
  "parameters": {
    "action": "screenshot"
  }
}
</tool_call>

<tool_result>
[Image data]
</tool_result>
```

---

## 会话工具

### sessions_list - 列出会话

列出所有或筛选的会话。

**参数**：
```typescript
{
  filter?: {
    user?: string;
    agent?: string;
    channel?: string;
  };
}
```

**示例**：
```markdown
用户: "显示所有活跃会话"

Agent:
<tool_call>
{
  "name": "sessions_list",
  "parameters": {
    "filter": {
      "agent": "coding"
    }
  }
}
</tool_call>

<tool_result>
Session ID        User         Channel   Messages
session-001       user1        whatsapp  45
session-002       user2        telegram  12
</tool_result>
```

### sessions_history - 查看历史

查看特定会话的消息历史。

**参数**：
```typescript
{
  sessionId: string;
  limit?: number;
}
```

### sessions_send - 发送消息

向特定会话发送消息。

**参数**：
```typescript
{
  sessionId: string;
  message: string;
}
```

### sessions_spawn - 创建会话

创建新会话。

**参数**：
```typescript
{
  agent: string;
  user: string;
  channel: string;
}
```

---

## 其他工具

### image - 图像生成

使用 AI 生成图像。

**参数**：
```typescript
{
  prompt: string;       // 图像描述
  size?: "256x256" | "512x512" | "1024x1024";
  n?: number;           // 生成数量
}
```

**示例**：
```markdown
用户: "生成一个未来城市的图片"

Agent:
<tool_call>
{
  "name": "image",
  "parameters": {
    "prompt": "A futuristic city with flying cars and tall buildings at sunset",
    "size": "1024x1024",
    "n": 1
  }
}
</tool_call>
```

### pdf - PDF 处理

处理 PDF 文件。

**参数**：
```typescript
{
  action: "extract" | "merge" | "split";
  path: string;
  output?: string;
}
```

### message - 发送消息

向其他通道发送消息。

**参数**：
```typescript
{
  channel: string;
  target: string;
  message: string;
}
```

**示例**：
```markdown
用户: "把这个结果发送到 Slack"

Agent:
<tool_call>
{
  "name": "message",
  "parameters": {
    "channel": "slack",
    "target": "#general",
    "message": "分析完成：发现了 3 个性能问题"
  }
}
</tool_call>
```

### cron - 定时任务

管理定时任务。

**参数**：
```typescript
{
  action: "create" | "list" | "delete";
  schedule?: string;  // cron 表达式
  task?: string;
}
```

**示例**：
```markdown
用户: "每天早上 9 点发送报告"

Agent:
<tool_call>
{
  "name": "cron",
  "parameters": {
    "action": "create",
    "schedule": "0 9 * * *",
    "task": "send_daily_report"
  }
}
</tool_call>
```

### gateway - Gateway 控制

控制 Gateway 操作。

**参数**：
```typescript
{
  action: "restart" | "reload" | "status";
}
```

### nodes - 节点管理

管理分布式节点。

**参数**：
```typescript
{
  action: "list" | "add" | "remove";
  node?: string;
}
```

---

## 工具配置

### 工具配置文件 (Profiles)

```json
{
  "tools": {
    "profiles": {
      "minimal": {
        "allow": ["read", "write"],
        "deny": ["exec", "process"]
      },
      "coding": {
        "allow": ["read", "write", "edit", "apply_patch", "web_search", "web_fetch"],
        "deny": ["message", "cron", "gateway"]
      },
      "messaging": {
        "allow": ["read", "write", "message", "sessions_list", "sessions_history"],
        "deny": ["exec", "process"]
      },
      "full": {
        "allow": ["*"]
      }
    }
  }
}
```

### Agent 工具配置

```json
{
  "agents": {
    "list": [
      {
        "id": "coder",
        "tools": {
          "profile": "coding",
          "config": {
            "exec": {
              "allowedCommands": ["python", "node", "npm", "git"],
              "workingDirectory": "/workspace"
            }
          }
        }
      }
    ]
  }
}
```

---

## 创建自定义工具

### 工具定义

在 workspace 中创建自定义工具：

```markdown
# TOOLS.md

## 自定义工具

### analyze_code - 代码分析工具

描述: 分析代码质量并提供改进建议

参数:
- code: 要分析的代码
- language: 编程语言

使用示例:
```
<tool_call>
{
  "name": "analyze_code",
  "parameters": {
    "code": "function foo() { return 1 }",
    "language": "javascript"
  }
}
</tool_call>
```

实现:
1. 使用 web_search 查找最佳实践
2. 分析代码结构
3. 提供改进建议
```

### 工具实现

```markdown
# AGENTS.md

## 工具实现

### analyze_code

当用户调用 analyze_code 工具时：

1. **验证输入**
   - 确认 code 参数存在
   - 验证 language 支持

2. **搜索最佳实践**
   - 使用 web_search 搜索 "{language} best practices"
   - 使用 web_fetch 获取官方指南

3. **分析代码**
   - 检查代码风格
   - 识别潜在问题
   - 评估可读性

4. **生成报告**
   - 总结发现
   - 提供具体建议
   - 给出示例代码
```

---

## 最佳实践

### 工具使用原则

```markdown
# AGENTS.md

## 工具使用原则

1. **必要性优先**
   - 只在需要时使用工具
   - 避免不必要的工具调用
   - 考虑成本和时间

2. **验证结果**
   - 检查工具返回结果
   - 验证数据完整性
   - 处理可能的错误

3. **并行执行**
   - 独立工具可以并行执行
   - 依赖工具需要顺序执行
   - 注意并发限制

4. **错误处理**
   - 捕获工具错误
   - 提供有用的错误信息
   - 尝试替代方案
```

### 工具组合示例

```markdown
## 工具组合模式

### 模式 1: 搜索 -> 获取 -> 分析
1. web_search - 查找相关资源
2. web_fetch - 获取详细内容
3. read/write - 保存和分析

### 模式 2: 读取 -> 处理 -> 写入
1. read - 读取输入文件
2. exec - 处理数据
3. write - 保存结果

### 模式 3: 创建 -> 测试 -> 部署
1. write - 创建代码
2. exec - 运行测试
3. message - 报告结果
```

---

## 小结

本节我们学习了：

1. ✅ 工具系统概述 - 分类和架构
2. ✅ 文件系统工具 - read, write, edit, apply_patch
3. ✅ 执行工具 - exec, process
4. ✅ Web 工具 - web_search, web_fetch, browser
5. ✅ 会话工具 - sessions_list, sessions_history, sessions_send, sessions_spawn
6. ✅ 其他工具 - image, pdf, message, cron, gateway, nodes
7. ✅ 工具配置 - Profiles 和 Agent 配置
8. ✅ 创建自定义工具 - 定义和实现
9. ✅ 最佳实践 - 使用原则和组合模式

## 下一步

[Skills 系统](./14-skills-system.md) - 扩展 Agent 能力
