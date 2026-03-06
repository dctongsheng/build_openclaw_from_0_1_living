# 插件系统

> 学习目标：通过插件扩展功能

## 目录

- [插件概述](#插件概述)
- [核心插件](#核心插件)
- [插件安装](#插件安装)
- [插件配置](#插件配置)
- [创建自定义插件](#创建自定义插件)
- [插件开发](#插件开发)
- [最佳实践](#最佳实践)
- [小结](#小结)

---

## 插件概述

### 什么是插件？

插件是 OpenClaw 的扩展模块，可以添加新功能、集成外部服务或修改现有行为。

### 插件架构

```
OpenClaw Core
├── Built-in Features
├── Plugin System
│   ├── Plugin Loader
│   ├── Plugin Manager
│   └── Plugin API
└── Plugins
    ├── Lobster (Workflow)
    ├── LLM Task (Structured Output)
    ├── Voice Call
    ├── Channel Plugins
    └── Custom Plugins
```

### 插件类型

| 类型 | 说明 | 示例 |
|-----|------|------|
| **通道插件** | 添加新的消息通道 | Matrix, IRC |
| **工具插件** | 添加新工具 | OCR, Spreadsheet |
| **工作流插件** | 自定义工作流 | Lobster |
| **集成插件** | 集成外部服务 | GitHub, Jira |
| **UI 插件** | 扩展 Web UI | Dashboard Widgets |

---

## 核心插件

### 1. Lobster - 工作流运行时

Lobster 是一个强大的工作流引擎，允许定义复杂的多步骤工作流。

#### 安装

```bash
# 通过 npm 安装
npm install -g @openclaw/plugin-lobster

# 或通过 OpenClaw CLI
openclaw plugins install lobster
```

#### 配置

```json
{
  "plugins": {
    "lobster": {
      "enabled": true,
      "workflows": {
        "directory": "~/.openclaw/workflows"
      },
      "execution": {
        "timeout": 300000,
        "maxConcurrent": 5
      }
    }
  }
}
```

#### 工作流定义示例

```yaml
# code-review-workflow.yaml
name: Code Review
description: 自动化代码审查流程

steps:
  - name: checkout
    action: git.checkout
    params:
      repository: "{{.repo}}"
      branch: "{{.branch}}"

  - name: analyze
    action: code.analyze
    params:
      path: "./src"

  - name: test
    action: test.run
    params:
      framework: "pytest"

  - name: report
    action: report.generate
    params:
      format: "markdown"
      output: "review-report.md"

on:
  success:
    - notify:
        channel: "#dev"
        message: "代码审查完成"
  failure:
    - notify:
        channel: "#alerts"
        message: "代码审查失败"
```

#### 使用工作流

```markdown
# 用户请求
用户: "运行代码审查流程"

Agent:
检测到 code-review 工作流，执行步骤：
1. ✅ 检出代码
2. ✅ 分析代码
3. ✅ 运行测试
4. ✅ 生成报告

代码审查完成！查看报告：review-report.md
```

### 2. LLM Task - 结构化输出

提供结构化输出和验证功能。

#### 安装

```bash
openclaw plugins install llm-task
```

#### 配置

```json
{
  "plugins": {
    "llm-task": {
      "enabled": true,
      "schemas": {
        "directory": "~/.openclaw/schemas"
      },
      "validation": {
        "strict": true
      }
    }
  }
}
```

#### Schema 定义

```json
{
  "name": "user-profile",
  "description": "用户信息提取",
  "schema": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string",
        "description": "用户姓名"
      },
      "email": {
        "type": "string",
        "format": "email",
        "description": "用户邮箱"
      },
      "age": {
        "type": "integer",
        "minimum": 0,
        "maximum": 150,
        "description": "用户年龄"
      }
    },
    "required": ["name", "email"]
  }
}
```

#### 使用

```markdown
用户: "从这段文本提取用户信息：张三，zhangsan@example.com，25岁"

Agent:
<tool_result>
{
  "name": "张三",
  "email": "zhangsan@example.com",
  "age": 25
}
</tool_result>
```

### 3. Voice Call - 语音通话

支持语音通话功能。

#### 安装

```bash
openclaw plugins install voice-call
```

#### 配置

```json
{
  "plugins": {
    "voice-call": {
      "enabled": true,
      "provider": "twilio",
      "credentials": {
        "accountSid": "${TWILIO_ACCOUNT_SID}",
        "authToken": "${TWILIO_AUTH_TOKEN}"
      },
      "settings": {
        "voice": "alice",
        "language": "zh-CN"
      }
    }
  }
}
```

#### 使用

```markdown
用户: "给我打电话"

Agent: [发起语音通话]
```

---

## 插件安装

### 从 npm 安装

```bash
# 全局安装
npm install -g @openclaw/plugin-<name>

# 列出已安装的插件
openclaw plugins list

# 查看插件详情
openclaw plugins info <name>
```

### 从 GitHub 安装

```bash
# 克隆仓库
git clone https://github.com/user/openclaw-plugin.git
cd openclaw-plugin

# 安装依赖
npm install

# 链接到 OpenClaw
openclaw plugins link .
```

### 从本地文件安装

```bash
# 安装本地插件包
openclaw plugins install ./path/to/plugin.tgz

# 或开发模式链接
openclaw plugins link /path/to/plugin
```

---

## 插件配置

### 全局配置

```json
{
  "plugins": {
    "enabled": true,
    "directory": "~/.openclaw/plugins",
    "autoLoad": true,
    "autoUpdate": false
  }
}
```

### 插件特定配置

```json
{
  "plugins": {
    "my-plugin": {
      "enabled": true,
      "config": {
        "apiKey": "${PLUGIN_API_KEY}",
        "endpoint": "https://api.example.com",
        "options": {
          "timeout": 30000,
          "retries": 3
        }
      }
    }
  }
}
```

### 插件权限

```json
{
  "plugins": {
    "my-plugin": {
      "permissions": {
        "tools": ["read", "write"],
        "network": ["api.example.com"],
        "filesystem": ["~/.openclaw/data"]
      }
    }
  }
}
```

---

## 创建自定义插件

### 插件结构

```
my-plugin/
├── package.json       # 包配置
├── plugin.json        # 插件清单
├── index.js           # 主入口
├── lib/               # 库代码
│   ├── tool.js
│   └── handler.js
├── schemas/           # Schema 定义
└── README.md          # 文档
```

### package.json

```json
{
  "name": "@openclaw/plugin-my-plugin",
  "version": "1.0.0",
  "description": "My custom OpenClaw plugin",
  "main": "index.js",
  "keywords": ["openclaw", "plugin"],
  "openclaw": {
    "type": "tool",
    "version": ">=1.0.0"
  },
  "dependencies": {
    "@openclaw/plugin-sdk": "^1.0.0"
  }
}
```

### plugin.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "type": "tool",
  "description": "My custom plugin",
  "author": "Your Name",
  "homepage": "https://github.com/user/my-plugin",
  "tools": [
    {
      "name": "my_tool",
      "description": "Does something useful",
      "parameters": {
        "input": {
          "type": "string",
          "required": true
        }
      }
    }
  ],
  "hooks": [
    {
      "event": "before_tool_call",
      "handler": "beforeToolCall"
    }
  ]
}
```

### index.js

```javascript
const { Plugin } = require('@openclaw/plugin-sdk');

class MyPlugin extends Plugin {
  constructor(config) {
    super(config);
    this.name = 'my-plugin';
  }

  // 注册工具
  getTools() {
    return [
      {
        name: 'my_tool',
        description: 'Does something useful',
        parameters: {
          input: { type: 'string', required: true }
        },
        handler: this.myToolHandler.bind(this)
      }
    ];
  }

  // 工具处理器
  async myToolHandler(params, context) {
    const { input } = params;

    // 执行逻辑
    const result = await this.processInput(input);

    return {
      success: true,
      data: result
    };
  }

  // Hook 处理器
  async beforeToolCall(toolCall, context) {
    this.log(`Tool ${toolCall.name} called`);
    return toolCall;
  }

  // 辅助方法
  async processInput(input) {
    // 处理输入
    return `Processed: ${input}`;
  }
}

module.exports = MyPlugin;
```

---

## 插件开发

### 开发环境设置

```bash
# 创建插件目录
mkdir my-plugin && cd my-plugin

# 初始化项目
npm init -y

# 安装 SDK
npm install @openclaw/plugin-sdk

# 创建开发链接
openclaw plugins link .
```

### 测试插件

```bash
# 测试插件
openclaw plugins test my-plugin

# 运行特定测试
openclaw plugins test my-plugin --tool my_tool

# 调试模式
openclaw plugins test my-plugin --debug
```

### 发布插件

```bash
# 构建
npm run build

# 发布到 npm
npm publish

# 或发布到 GitHub Releases
npm pack
```

---

## 最佳实践

### 1. 错误处理

```javascript
async myToolHandler(params, context) {
  try {
    const result = await this.doSomething(params);
    return { success: true, data: result };
  } catch (error) {
    this.error('Operation failed', error);
    return {
      success: false,
      error: error.message
    };
  }
}
```

### 2. 日志记录

```javascript
class MyPlugin extends Plugin {
  constructor(config) {
    super(config);
    this.logger = this.getLogger('my-plugin');
  }

  async myToolHandler(params, context) {
    this.logger.info('Processing request', { params });
    // ...
    this.logger.debug('Processing complete');
  }
}
```

### 3. 配置验证

```javascript
class MyPlugin extends Plugin {
  validateConfig(config) {
    const schema = {
      apiKey: { type: 'string', required: true },
      endpoint: { type: 'string', required: true }
    };

    return this.validate(schema, config);
  }
}
```

### 4. 版本兼容

```json
{
  "openclaw": {
    "version": ">=1.0.0 <2.0.0"
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ 插件概述 - 架构和类型
2. ✅ 核心插件 - Lobster, LLM Task, Voice Call
3. ✅ 插件安装 - npm, GitHub, 本地
4. ✅ 插件配置 - 全局和特定配置
5. ✅ 创建自定义插件 - 结构和示例代码
6. ✅ 插件开发 - 环境设置、测试、发布
7. ✅ 最佳实践 - 错误处理、日志、配置验证、版本兼容

## 实践练习

1. 创建一个简单的工具插件
2. 添加一个自定义 Hook
3. 编写插件测试
4. 发布你的插件

## 下一步

[Hooks 系统](./18-hooks.md) - 使用 Hooks 实现自定义逻辑
