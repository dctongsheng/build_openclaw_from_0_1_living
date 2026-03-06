# 流式传输与队列

> 学习目标：理解响应流和队列机制

## 目录

- [流式传输概述](#流式传输概述)
- [Streaming 模式](#streaming-模式)
- [Block streaming](#block-streaming)
- [Queue 模式](#queue-模式)
- [并发控制](#并发控制)
- [流式配置](#流式配置)
- [最佳实践](#最佳实践)
- [小结](#小结)

---

## 流式传输概述

### 为什么需要流式传输？

```
问题:
├── 响应时间长
├── 用户体验差
├── 超时风险
└── 资源浪费

解决方案:
├── 流式传输
├── 队列管理
├── 并发控制
└── 优先级调度
```

### 流式传输架构

```
LLM Provider
    ↓ (流式响应)
Gateway
    ↓ (分块处理)
Formatter
    ↓ (通道适配)
Channel
    ↓ (实时发送)
User
```

---

## Streaming 模式

### 启用流式传输

```json
{
  "streaming": {
    "enabled": true,
    "chunkSize": 100,
    "interval": 50,
    "channels": {
      "whatsapp": false,  // WhatsApp 不支持流式
      "telegram": true,
      "discord": true,
      "slack": true
    }
  }
}
```

### 流式传输流程

```javascript
async function streamResponse(response, channel) {
  const stream = response.stream();

  for await (const chunk of stream) {
    // 处理分块
    const processed = processChunk(chunk);

    // 格式化
    const formatted = formatForChannel(processed, channel);

    // 发送
    await sendToChannel(channel, formatted);

    // 检查完成
    if (chunk.done) {
      await completeStream(channel);
      break;
    }
  }
}
```

### 分块策略

#### 固定大小分块

```json
{
  "streaming": {
    "strategy": "fixed",
    "chunkSize": 100,
    "overlap": 0
  }
}
```

#### 智能分块

```json
{
  "streaming": {
    "strategy": "smart",
    "rules": [
      {
        "condition": "sentence_end",
        "action": "split"
      },
      {
        "condition": "max_length",
        "value": 200,
        "action": "split"
      }
    ]
  }
}
```

---

## Block streaming

### 什么是 Block streaming？

Block streaming 会累积内容直到完成，然后一次性发送。

### 配置

```json
{
  "streaming": {
    "mode": "block",
    "channels": {
      "whatsapp": true,
      "email": true
    }
  }
}
```

### 使用场景

```markdown
适用场景:
✅ 不支持流式的通道
✅ 需要完整内容的场景
✅ 邮件等异步消息
✅ 文档生成

不适用场景:
❌ 实时对话
❌ 长内容生成
❌ 需要即时反馈
```

### 混合模式

```json
{
  "streaming": {
    "mode": "hybrid",
    "rules": [
      {
        "condition": "length < 500",
        "mode": "stream"
      },
      {
        "condition": "length >= 500",
        "mode": "block"
      }
    ]
  }
}
```

---

## Queue 模式

### Queue 类型

```
Queue 类型:
├── steer - 引导队列
│   └── 智能路由
├── followup - 后续队列
│   └── 后续任务
└── collect - 收集队列
    └── 批量处理
```

### Steer Queue

引导用户或 Agent 到特定流程：

```json
{
  "queues": {
    "steer": {
      "enabled": true,
      "flows": [
        {
          "name": "onboarding",
          "trigger": "new_user",
          "steps": [
            {
              "action": "welcome",
              "message": "欢迎使用 OpenClaw!"
            },
            {
              "action": "configure",
              "message": "让我们配置你的 Agent"
            },
            {
              "action": "test",
              "message": "发送测试消息"
            }
          ]
        }
      ]
    }
  }
}
```

### Followup Queue

自动生成后续任务：

```json
{
  "queues": {
    "followup": {
      "enabled": true,
      "rules": [
        {
          "condition": "code_generated",
          "actions": [
            {
              "type": "test",
              "prompt": "为这段代码编写测试"
            },
            {
              "type": "document",
              "prompt": "为这段代码添加文档"
            }
          ]
        }
      ]
    }
  }
}
```

### Collect Queue

批量收集和处理请求：

```json
{
  "queues": {
    "collect": {
      "enabled": true,
      "batchSize": 10,
      "timeout": 60,
      "processor": "batchProcessor"
    }
  }
}
```

---

## 并发控制

### 并发限制

```json
{
  "concurrency": {
    "enabled": true,
    "maxSessions": 10,
    "maxMessagesPerSession": 3,
    "maxTotalMessages": 30
  }
}
```

### 优先级队列

```json
{
  "queues": {
    "priority": {
      "enabled": true,
      "levels": [
        {
          "name": "urgent",
          "weight": 10,
          "conditions": ["@urgent", "critical"]
        },
        {
          "name": "high",
          "weight": 5,
          "conditions": ["@high", "important"]
        },
        {
          "name": "normal",
          "weight": 1,
          "conditions": ["*"]
        }
      ]
    }
  }
}
```

### 速率限制

```json
{
  "rateLimit": {
    "enabled": true,
    "perUser": {
      "messages": 10,
      "period": 60
    },
    "perChannel": {
      "messages": 100,
      "period": 60
    },
    "global": {
      "messages": 1000,
      "period": 60
    }
  }
}
```

---

## 流式配置

### 完整配置示例

```json
{
  "streaming": {
    "enabled": true,

    // 流式模式
    "mode": "stream",
    "chunkSize": 100,
    "interval": 50,

    // 通道配置
    "channels": {
      "whatsapp": {
        "enabled": false,
        "mode": "block"
      },
      "telegram": {
        "enabled": true,
        "chunkSize": 150
      },
      "discord": {
        "enabled": true,
        "chunkSize": 200
      },
      "slack": {
        "enabled": true,
        "chunkSize": 100
      }
    },

    // 格式化
    "formatting": {
      "preserveBreaks": true,
      "markdown": true,
      "codeBlocks": true
    },

    // 错误处理
    "errorHandling": {
      "retry": true,
      "maxRetries": 3,
      "retryDelay": 1000
    }
  },

  // 队列配置
  "queues": {
    "steer": {
      "enabled": true
    },
    "followup": {
      "enabled": true,
      "maxQueued": 5
    },
    "collect": {
      "enabled": false
    }
  },

  // 并发控制
  "concurrency": {
    "maxSessions": 10,
    "maxMessagesPerSession": 3,
    "maxTotalMessages": 30
  },

  // 速率限制
  "rateLimit": {
    "perUser": {
      "messages": 10,
      "period": 60
    }
  }
}
```

---

## 最佳实践

### 1. 选择合适的模式

```javascript
// 根据内容和通道选择
function getStreamingMode(content, channel) {
  // 短内容：block
  if (content.length < 500) {
    return 'block';
  }

  // 不支持流式的通道：block
  if (!supportsStreaming(channel)) {
    return 'block';
  }

  // 长内容：stream
  return 'stream';
}
```

### 2. 优化分块大小

```javascript
// 根据通道特性优化
function getOptimalChunkSize(channel) {
  const sizes = {
    telegram: 150,
    discord: 200,
    slack: 100,
    whatsapp: Infinity  // 不支持流式
  };

  return sizes[channel] || 100;
}
```

### 3. 处理错误

```javascript
async function streamWithErrorHandling(stream, channel) {
  try {
    for await (const chunk of stream) {
      await sendChunk(chunk, channel);
    }
  } catch (error) {
    // 记录错误
    logError(error);

    // 尝试恢复
    if (isRecoverable(error)) {
      await retryStream(stream, channel);
    } else {
      // 通知用户
      await notifyError(channel, error);
    }
  }
}
```

### 4. 监控性能

```bash
# 监控流式传输
openclaw monitor streaming

# 查看队列状态
openclaw queues status

# 查看并发情况
openclaw stats concurrency
```

### 5. 用户体验

```javascript
// 提供视觉反馈
async function streamWithFeedback(stream, channel) {
  // 发送"正在输入"指示
  await sendTypingIndicator(channel, true);

  // 流式传输
  for await (const chunk of stream) {
    await sendChunk(chunk, channel);
  }

  // 取消"正在输入"指示
  await sendTypingIndicator(channel, false);
}
```

---

## 小结

本节我们学习了：

1. ✅ 流式传输概述 - 必要性和架构
2. ✅ Streaming 模式 - 配置和流程
3. ✅ Block streaming - 使用场景和混合模式
4. ✅ Queue 模式 - steer, followup, collect
5. ✅ 并发控制 - 并发限制、优先级队列、速率限制
6. ✅ 流式配置 - 完整配置示例
7. ✅ 最佳实践 - 模式选择、分块优化、错误处理、性能监控、用户体验

## 性能优化检查清单

- [ ] 启用流式传输（支持流式的通道）
- [ ] 优化分块大小
- [ ] 配置并发控制
- [ ] 设置速率限制
- [ ] 配置优先级队列
- [ ] 启用错误处理
- [ ] 监控性能指标
- [ ] 测试用户体验

## 第五阶段完成！

恭喜！你已经完成了高级功能学习：

- ✅ 沙箱与安全
- ✅ 插件系统
- ✅ Hooks 系统
- ✅ 内存与上下文管理
- ✅ 流式传输与队列

## 下一步

[第六阶段：生产部署](../stage6-deployment/21-deployment.md) - 选择合适的部署方案
