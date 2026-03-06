# 内存与上下文管理

> 学习目标：优化 Agent 内存和上下文

## 目录

- [内存系统概述](#内存系统概述)
- [Session 管理](#session-管理)
- [Context 管理](#context-管理)
- [Compaction 策略](#compaction-策略)
- [Token 使用优化](#token-使用优化)
- [性能监控](#性能监控)
- [最佳实践](#最佳实践)
- [小结](#小结)

---

## 内存系统概述

### 为什么需要内存管理？

```
问题:
├── 上下文窗口有限
├── Token 成本高
├── 响应时间慢
└── 内存占用大

解决方案:
├── Session 管理
├── Context 优化
├── Compaction 策略
└── 缓存机制
```

### 内存层级

```
内存层级:
├── 短期内存 (工作内存)
│   └── 当前会话上下文
├── 中期内存 (会话记忆)
│   └── 会话历史和摘要
└── 长期记忆 (知识库)
    └── 持久化信息和用户画像
```

---

## Session 管理

### Session 生命周期

```
创建 → 激活 → 使用 → 压缩 → 归档 → 删除
  ↓      ↓      ↓      ↓      ↓      ↓
init  active  prune compact archive delete
```

### Session 配置

```json
{
  "sessions": {
    "directory": "~/.openclaw/sessions",
    "format": "jsonl",
    "compression": {
      "enabled": true,
      "algorithm": "gzip"
    },
    "retention": {
      "active": 7,          // 天
      "archived": 30,       // 天
      "maxSessions": 1000
    },
    "cleanup": {
      "enabled": true,
      "schedule": "0 2 * * *"  // 每天凌晨 2 点
    }
  }
}
```

### Session 存储格式

```json
{
  "id": "session-001",
  "agent": "coding",
  "user": "user123",
  "channel": "whatsapp",
  "created": "2024-01-15T10:00:00Z",
  "updated": "2024-01-15T11:30:00Z",
  "messages": [
    {
      "role": "user",
      "content": "帮我写一个 Python 函数",
      "timestamp": "2024-01-15T10:00:00Z"
    },
    {
      "role": "assistant",
      "content": "好的，这是...",
      "timestamp": "2024-01-15T10:00:05Z"
    }
  ],
  "summary": "用户请求编写 Python 函数，已完成",
  "metadata": {
    "messageCount": 45,
    "tokensUsed": 15234,
    "tags": ["python", "coding"]
  }
}
```

### Session 操作

```bash
# 列出会话
openclaw sessions list

# 查看会话详情
openclaw sessions info session-001

# 删除会话
openclaw sessions delete session-001

# 归档会话
openclaw sessions archive session-001

# 清理旧会话
openclaw sessions cleanup --older-than 30d
```

---

## Context 管理

### Context 结构

```typescript
interface AgentContext {
  // 系统信息
  system: SystemInfo;

  // 用户信息
  user: UserInfo;

  // 当前消息
  current: CurrentMessage;

  // 对话历史
  history: MessageHistory;

  // 工具信息
  tools: ToolsInfo;

  // 外部数据
  external?: ExternalData;
}
```

### Context 优化策略

#### 1. 分层加载

```javascript
async function loadContext(session, options) {
  const context = {};

  // 始终加载：基本信息
  context.basic = {
    system: loadSystemInfo(),
    user: loadUserInfo(session.userId)
  };

  // 按需加载：详细信息
  if (options.includeHistory) {
    context.history = loadHistory(session.id, options.historyLimit);
  }

  if (options.includeTools) {
    context.tools = loadToolsInfo(session.agentId);
  }

  // 可选加载：外部数据
  if (options.includeExternal) {
    context.external = await loadExternalData(session);
  }

  return context;
}
```

#### 2. 增量更新

```javascript
// 不每次都重建整个 context
function updateContext(context, updates) {
  return {
    ...context,
    ...updates,
    updated: new Date()
  };
}
```

#### 3. 智能缓存

```json
{
  "context": {
    "cache": {
      "enabled": true,
      "strategy": "lru",
      "maxSize": 100,
      "ttl": 3600,
      "items": {
        "userInfo": 7200,
        "systemInfo": 86400,
        "toolsInfo": 3600
      }
    }
  }
}
```

---

## Compaction 策略

### 什么是 Compaction？

Compaction 是压缩和优化会话历史的过程，减少 token 使用。

### Compaction 策略

#### 1. 消息窗口

只保留最近的 N 条消息：

```json
{
  "sessions": {
    "compaction": {
      "strategy": "window",
      "windowSize": 50,
      "keepRecent": 10,
      "summarizeOld": true
    }
  }
}
```

#### 2. Token 限制

根据 token 数量限制：

```json
{
  "sessions": {
    "compaction": {
      "strategy": "tokens",
      "maxTokens": 10000,
      "reserveTokens": 2000,
      "summarizeThreshold": 8000
    }
  }
}
```

#### 3. 智能摘要

保留重要信息，摘要其他内容：

```json
{
  "sessions": {
    "compaction": {
      "strategy": "smart",
      "rules": [
        {
          "condition": "has_code",
          "action": "keep_full"
        },
        {
          "condition": "is_question",
          "action": "keep_summary"
        },
        {
          "condition": "is_chitchat",
          "action": "compress"
        }
      ]
    }
  }
}
```

### 摘要生成

```markdown
# AGENTS.md

## 会话摘要

当会话需要压缩时：

1. **识别关键信息**
   - 用户目标
   - 重要决策
   - 代码/数据
   - 待办事项

2. **生成摘要**
   - 简洁描述
   - 保留关键细节
   - 时间戳标记

3. **格式化**
   ```markdown
   ## 会话摘要 (2024-01-15)

   **目标**: 创建 Python API

   **完成**:
   - ✅ 设计数据模型
   - ✅ 实现 API 端点
   - ⏳ 测试 (进行中)

   **关键代码**: 见 `/workspace/api/main.py`

   **待办**: 添加认证
   ```
```

---

## Token 使用优化

### Token 监控

```json
{
  "monitoring": {
    "tokens": {
      "enabled": true,
      "alertThreshold": 100000,
      "resetPeriod": "monthly",
      "breakdown": {
        "byAgent": true,
        "byUser": true,
        "byChannel": true
      }
    }
  }
}
```

### 优化技巧

#### 1. 使用摘要

```javascript
// 不发送完整历史，发送摘要
const context = {
  messages: recentMessages,  // 最近 10 条
  summary: sessionSummary,   // 历史摘要
  totalMessages: session.messageCount
};
```

#### 2. 去除冗余

```javascript
// 去除重复或相似的消息
function deduplicateMessages(messages) {
  return messages.filter((msg, i) => {
    if (i === 0) return true;
    const prev = messages[i - 1];
    return !isSimilar(msg, prev);
  });
}
```

#### 3. 压缩长文本

```javascript
// 对于很长的内容，使用引用或摘要
function compressLongContent(content) {
  if (content.length > 1000) {
    return {
      type: 'compressed',
      original: content,
      summary: summarize(content),
      reference: store(content)
    };
  }
  return content;
}
```

#### 4. 选择性包含

```javascript
// 只包含相关的历史消息
function selectRelevantHistory(currentMessage, allMessages) {
  const keywords = extractKeywords(currentMessage);

  return allMessages.filter(msg => {
    return hasKeywords(msg, keywords) ||
           isRecent(msg, 7);  // 或最近 7 条
  });
}
```

---

## 性能监控

### 监控指标

```bash
# 查看内存使用
openclaw stats memory

# 查看 Token 使用
openclaw stats tokens

# 查看会话统计
openclaw stats sessions

# 查看性能报告
openclaw stats performance
```

### 性能基准

```json
{
  "performance": {
    "targets": {
      "responseTime": {
        "p50": 2000,
        "p95": 5000,
        "p99": 10000
      },
      "tokenUsage": {
        "perMessage": 1000,
        "perSession": 50000
      },
      "memory": {
        "perSession": "10M",
        "total": "1G"
      }
    }
  }
}
```

### 告警配置

```json
{
  "alerts": {
    "memory": {
      "enabled": true,
      "threshold": "80%",
      "action": "compact"
    },
    "tokens": {
      "enabled": true,
      "threshold": 100000,
      "action": "notify"
    },
    "sessions": {
      "enabled": true,
      "threshold": 500,
      "action": "cleanup"
    }
  }
}
```

---

## 最佳实践

### 1. 定期清理

```bash
# 设置定期清理任务
openclaw cron create --schedule "0 2 * * *" --command "sessions cleanup --older-than 30d"
```

### 2. 监控和调整

```javascript
// 定期检查性能并调整
async function optimizeMemory() {
  const stats = await getStats();

  if (stats.memory > threshold) {
    await compactSessions();
  }

  if (stats.sessions > maxSessions) {
    await archiveOldSessions();
  }
}
```

### 3. 使用摘要

```json
{
  "sessions": {
    "summarization": {
      "enabled": true,
      "trigger": "after",
      "interval": 10,  // 每 10 条消息
      "method": "ai"
    }
  }
}
```

### 4. 分层存储

```javascript
// 热数据：内存
// 温数据：快速存储
// 冷数据：归档存储

const storage = {
  hot: new MemoryStore(),
  warm: new FileStore(),
  cold: new ArchiveStore()
};
```

### 5. 用户控制

```json
{
  "sessions": {
    "userControl": {
      "enabled": true,
      "allowDelete": true,
      "allowArchive": true,
      "allowExport": true
    }
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ 内存系统概述 - 问题和解决方案
2. ✅ Session 管理 - 生命周期、配置、操作
3. ✅ Context 管理 - 结构和优化策略
4. ✅ Compaction 策略 - 消息窗口、Token 限制、智能摘要
5. ✅ Token 使用优化 - 监控、优化技巧
6. ✅ 性能监控 - 指标、基准、告警
7. ✅ 最佳实践 - 清理、监控、摘要、分层存储、用户控制

## 性能检查清单

- [ ] 启用 Session 压缩
- [ ] 配置 Token 限制
- [ ] 设置摘要生成
- [ ] 启用智能缓存
- [ ] 配置性能监控
- [ ] 设置定期清理
- [ ] 优化上下文加载
- [ ] 测试性能基准

## 下一步

[流式传输与队列](./20-streaming.md) - 理解响应流和队列机制
