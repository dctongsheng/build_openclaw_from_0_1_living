# 多 Agent 配置

> 学习目标：配置专门的 Agent

## 目录

- [多 Agent 架构](#多-agent-架构)
- [Agent 类型](#agent-类型)
- [Agent 路由](#agent-路由)
- [Agent 协作](#agent-协作)
- [配置示例](#配置示例)
- [Agent 管理](#agent-管理)
- [最佳实践](#最佳实践)
- [小结](#小结)

---

## 多 Agent 架构

### 为什么需要多 Agent？

```
单一 Agent 的局限：
├── 上下文冲突
├── 工具权限过大
├── 职责不明确
└── 难以优化

多 Agent 的优势：
├── 职责分离
├── 权限最小化
├── 性能优化
└── 易于维护
```

### 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                    Request Router                        │
│  ┌─────────────────────────────────────────────────────┐│
│  │  分析请求                                           ││
│  │  - 识别意图                                         ││
│  │  - 确定上下文                                       ││
│  │  - 选择合适的 Agent                                 ││
│  └─────────────────────────────────────────────────────┘│
│                         ↓                                │
│  ┌───────────┬───────────┬───────────┬─────────────┐  │
│  │  Coding   │ Writing   │ Support   │  Research   │  │
│  │  Agent    │  Agent    │  Agent    │   Agent     │  │
│  └───────────┴───────────┴───────────┴─────────────┘  │
│                         ↓                                │
│  ┌─────────────────────────────────────────────────────┐│
│  │  Response Aggregator                                ││
│  │  - 汇总结果                                         ││
│  │  - 格式化输出                                       ││
│  │  - 发送响应                                         ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
```

---

## Agent 类型

### 1. 按功能分类

#### Coding Agent

专注于编程任务：

```json
{
  "agents": {
    "list": [
      {
        "id": "coding",
        "name": "编程助手",
        "model": "anthropic/claude-sonnet-4-20250514",
        "description": "专注于代码编写、审查和调试",
        "workspace": "~/.openclaw/workspace-coding",
        "tools": {
          "profile": "coding",
          "allow": ["read", "write", "edit", "apply_patch", "exec", "web_search"],
          "deny": ["message", "cron", "gateway"]
        },
        "skills": {
          "enabled": ["code-reviewer", "test-generator", "debug-helper"]
        },
        "routing": {
          "channels": ["whatsapp", "discord"],
          "keywords": ["code", "bug", "debug", "function", "编程", "调试"],
          "patterns": [
            "(?i)\\b(write|create|generate)\\s+(code|function|class)\\b",
            "(?i)\\b(debug|fix)\\s+(the\\s+)?(code|bug|error)\\b"
          ]
        },
        "limits": {
          "maxFileSize": 1048576,
          "maxExecTime": 30000,
          "allowedDirectories": ["/workspace", "/tmp"]
        }
      }
    ]
  }
}
```

#### Writing Agent

专注于写作任务：

```json
{
  "agents": {
    "list": [
      {
        "id": "writing",
        "name": "写作助手",
        "model": "openai/gpt-4o",
        "description": "专注于内容创作、编辑和翻译",
        "workspace": "~/.openclaw/workspace-writing",
        "tools": {
          "profile": "messaging",
          "allow": ["read", "write", "web_search", "web_fetch"],
          "deny": ["exec", "process", "cron"]
        },
        "skills": {
          "enabled": ["content-writer", "translator", "editor"]
        },
        "routing": {
          "channels": ["slack", "telegram"],
          "keywords": ["write", "article", "blog", "content", "写作", "文章"],
          "patterns": [
            "(?i)\\b(write|create|draft)\\s+(article|blog|content|post)\\b",
            "(?i)\\b(translate|翻译)\\b"
          ]
        }
      }
    ]
  }
}
```

#### Support Agent

专注于客户支持：

```json
{
  "agents": {
    "list": [
      {
        "id": "support",
        "name": "客服助手",
        "model": "openai/gpt-4o",
        "description": "专注于客户服务和支持",
        "workspace": "~/.openclaw/workspace-support",
        "tools": {
          "profile": "messaging",
          "allow": ["read", "write", "message", "web_search"],
          "deny": ["exec", "process"]
        },
        "skills": {
          "enabled": ["faq-responder", "ticket-creator", "escalation-handler"]
        },
        "routing": {
          "channels": ["slack"],
          "keywords": ["help", "support", "issue", "problem", "帮助", "问题"],
          "channels": {
            "support": "support"
          }
        }
      }
    ]
  }
}
```

#### Research Agent

专注于信息收集和分析：

```json
{
  "agents": {
    "list": [
      {
        "id": "research",
        "name": "研究助手",
        "model": "anthropic/claude-sonnet-4-20250514",
        "description": "专注于信息收集、分析和报告",
        "workspace": "~/.openclaw/workspace-research",
        "tools": {
          "profile": "full",
          "allow": ["web_search", "web_fetch", "browser", "read", "write", "pdf"],
          "deny": ["exec", "process", "message"]
        },
        "skills": {
          "enabled": ["web-researcher", "data-analyst", "report-generator"]
        },
        "routing": {
          "keywords": ["research", "find", "search", "analyze", "研究", "搜索"],
          "patterns": [
            "(?i)\\b(research|investigate|look\\s+up)\\b",
            "(?i)\\b(find|search)\\s+(for|information\\s+about)\\b"
          ]
        }
      }
    ]
  }
}
```

### 2. 按性能分类

#### Fast Agent

快速响应简单任务：

```json
{
  "agents": {
    "list": [
      {
        "id": "fast",
        "name": "快速响应",
        "model": "openai/gpt-4o-mini",
        "description": "快速处理简单任务",
        "tools": {
          "profile": "minimal"
        },
        "loop": {
          "timeout": {
            "total": 30000
          }
        },
        "routing": {
          "priority": 1,
          "maxTokens": 512
        }
      }
    ]
  }
}
```

#### Thorough Agent

深度分析复杂任务：

```json
{
  "agents": {
    "list": [
      {
        "id": "thorough",
        "name": "深度分析",
        "model": "anthropic/claude-opus-4-20250514",
        "description": "深入分析复杂任务",
        "tools": {
          "profile": "full"
        },
        "loop": {
          "timeout": {
            "total": 300000
          }
        },
        "routing": {
          "priority": 3,
          "maxTokens": 8192
        }
      }
    ]
  }
}
```

---

## Agent 路由

### 基于关键词路由

```json
{
  "agents": {
    "routing": {
      "strategy": "keyword",
      "rules": [
        {
          "keywords": ["code", "bug", "debug", "编程"],
          "agent": "coding",
          "confidence": 0.8
        },
        {
          "keywords": ["write", "article", "写作"],
          "agent": "writing",
          "confidence": 0.8
        },
        {
          "keywords": ["help", "support", "帮助"],
          "agent": "support",
          "confidence": 0.9
        }
      ]
    }
  }
}
```

### 基于模式匹配路由

```json
{
  "agents": {
    "routing": {
      "strategy": "pattern",
      "rules": [
        {
          "pattern": "(?i)\\b(write|create)\\s+(code|function|class)\\b",
          "agent": "coding"
        },
        {
          "pattern": "(?i)\\b(write|draft)\\s+(article|blog|content)\\b",
          "agent": "writing"
        }
      ]
    }
  }
}
```

### 基于通道路由

```json
{
  "agents": {
    "routing": {
      "strategy": "channel",
      "rules": [
        {
          "channel": "slack",
          "channelFilter": {
            "types": ["support"]
          },
          "agent": "support"
        },
        {
          "channel": "discord",
          "channelFilter": {
            "categories": ["dev"]
          },
          "agent": "coding"
        }
      ]
    }
  }
}
```

### 智能路由

```json
{
  "agents": {
    "list": [
      {
        "id": "router",
        "name": "智能路由器",
        "model": "anthropic/claude-haiku-4-20250514",
        "description": "分析请求并路由到合适的 Agent",
        "routing": {
          "analyze": true,
          "consider": [
            "intent",
            "complexity",
            "context",
            "urgency"
          ],
          "agents": ["coding", "writing", "support", "research"],
          "fallback": "general"
        }
      }
    ]
  }
}
```

---

## Agent 协作

### 顺序协作

Agent 按顺序处理任务：

```json
{
  "agents": {
    "workflows": {
      "code-review": {
        "agents": ["coding", "research"],
        "sequence": [
          {
            "agent": "coding",
            "task": "初步代码审查"
          },
          {
            "agent": "research",
            "task": "查找最佳实践"
          },
          {
            "agent": "coding",
            "task": "生成最终报告"
          }
        ]
      }
    }
  }
}
```

### 并行协作

多个 Agent 同时工作：

```json
{
  "agents": {
    "workflows": {
      "comprehensive-analysis": {
        "agents": ["coding", "research", "writing"],
        "parallel": true,
        "merge": "summary"
      }
    }
  }
}
```

### 委托模式

主 Agent 委托任务给子 Agent：

```json
{
  "agents": {
    "list": [
      {
        "id": "manager",
        "name": "项目协调员",
        "model": "anthropic/claude-sonnet-4-20250514",
        "delegation": {
          "enabled": true,
          "agents": ["coding", "research", "writing"],
          "strategy": "task-based"
        }
      }
    ]
  }
}
```

---

## 配置示例

### 完整多 Agent 配置

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "workspace": "~/.openclaw/workspace",
      "tools": {
        "profile": "full"
      }
    },
    "list": [
      {
        "id": "coding",
        "name": "编程助手",
        "model": "anthropic/claude-sonnet-4-20250514",
        "workspace": "~/.openclaw/workspace-coding",
        "tools": {
          "profile": "coding"
        },
        "routing": {
          "keywords": ["code", "bug", "debug"],
          "channels": ["whatsapp", "discord"]
        }
      },
      {
        "id": "writing",
        "name": "写作助手",
        "model": "openai/gpt-4o",
        "workspace": "~/.openclaw/workspace-writing",
        "tools": {
          "profile": "messaging"
        },
        "routing": {
          "keywords": ["write", "article", "content"],
          "channels": ["slack", "telegram"]
        }
      },
      {
        "id": "support",
        "name": "客服助手",
        "model": "openai/gpt-4o",
        "workspace": "~/.openclaw/workspace-support",
        "tools": {
          "profile": "messaging",
          "allow": ["slack"]
        },
        "routing": {
          "channels": ["slack"]
        }
      },
      {
        "id": "research",
        "name": "研究助手",
        "model": "anthropic/claude-sonnet-4-20250514",
        "workspace": "~/.openclaw/workspace-research",
        "tools": {
          "allow": ["web_search", "web_fetch", "browser", "read", "write", "pdf"]
        },
        "routing": {
          "keywords": ["research", "find", "analyze"]
        }
      }
    ],
    "routing": {
      "strategy": "intelligent",
      "fallback": "coding"
    }
  }
}
```

---

## Agent 管理

### 查看 Agent 状态

```bash
# 列出所有 Agent
openclaw agents list

# 查看 Agent 详情
openclaw agents info coding

# 查看 Agent 路由
openclaw agents routing

# 测试 Agent 路由
openclaw agents route test "帮我写一个 Python 函数"
```

### 动态 Agent 管理

```bash
# 添加 Agent
openclaw agents add --id new-agent --model anthropic/claude-sonnet-4-20250514

# 更新 Agent
openclaw agents update coding --model anthropic/claude-opus-4-20250514

# 删除 Agent
openclaw agents remove old-agent

# 启用/禁用 Agent
openclaw agents enable coding
openclaw agents disable support
```

### Agent 性能监控

```bash
# 查看 Agent 统计
openclaw agents stats

# 查看特定 Agent 性能
openclaw agents stats coding

# 设置性能告警
openclaw agents alert set --agent coding --metric responseTime --threshold 5000
```

---

## 最佳实践

### 1. 职责分离

每个 Agent 应该有明确的职责：

```
✅ 好的设计:
- Coding Agent → 只处理代码任务
- Writing Agent → 只处理写作任务
- Support Agent → 只处理支持任务

❌ 不好的设计:
- General Agent → 处理所有任务
```

### 2. 权限最小化

每个 Agent 只有必要的权限：

```json
{
  "agents": {
    "list": [
      {
        "id": "writer",
        "tools": {
          "allow": ["read", "write", "web_search"],
          "deny": ["exec", "process", "message"]
        }
      }
    ]
  }
}
```

### 3. 明确的路由规则

确保请求路由到正确的 Agent：

```json
{
  "agents": {
    "routing": {
      "rules": [
        {
          "keywords": ["code", "programming"],
          "agent": "coding",
          "confidence": 0.9
        }
      ],
      "fallback": "general",
      "confirmation": {
        "lowConfidence": true,
        "threshold": 0.7
      }
    }
  }
}
```

### 4. 共享资源管理

避免 Agent 之间的资源冲突：

```json
{
  "agents": {
    "sharedResources": {
      "sessions": {
        "strategy": "separate"  // separate, shared, hybrid
      },
      "workspaces": {
        "strategy": "separate"
      },
      "tools": {
        "strategy": "isolated"
      }
    }
  }
}
```

### 5. 监控和日志

监控 Agent 性能和行为：

```json
{
  "agents": {
    "monitoring": {
      "enabled": true,
      "metrics": [
        "responseTime",
        "tokenUsage",
        "toolCalls",
        "errorRate"
      ],
      "logging": {
        "level": "info",
        "includeContext": true,
        "includeReasoning": false
      }
    }
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ 多 Agent 架构 - 设计和优势
2. ✅ Agent 类型 - Coding, Writing, Support, Research, Fast, Thorough
3. ✅ Agent 路由 - 关键词、模式匹配、通道、智能路由
4. ✅ Agent 协作 - 顺序、并行、委托模式
5. ✅ 配置示例 - 完整的多 Agent 配置
6. ✅ Agent 管理 - 查看、添加、更新、删除、监控
7. ✅ 最佳实践 - 职责分离、权限最小化、明确路由、资源共享、监控日志

## 第四阶段完成！

恭喜！你已经完成了 Agent 定制学习：

- ✅ Agent Loop 深入理解
- ✅ 工具系统
- ✅ Skills 系统
- ✅ 多 Agent 配置

## 下一步

[第五阶段：高级功能](../stage5-advanced/16-security.md) - 沙箱与安全
