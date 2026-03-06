# 多通道管理

> 学习目标：统一管理多个通道

## 目录

- [多通道架构](#多通道架构)
- [通道优先级](#通道优先级)
- [消息路由](#消息路由)
- [统一响应格式](#统一响应格式)
- [通道特定配置](#通道特定配置)
- [跨通道会话](#跨通道会话)
- [通道监控](#通道监控)
- [最佳实践](#最佳实践)
- [小结](#小结)

---

## 多通道架构

### 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                    OpenClaw Gateway                      │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Message Router                        │  │
│  │  - 接收所有通道消息                               │  │
│  │  - 识别来源通道                                   │  │
│  │  - 应用通道特定规则                               │  │
│  │  - 路由到合适的 Agent                             │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Channel Layer                        │  │
│  │  ┌─────┐ ┌──────┐ ┌────────┐ ┌──────┐           │  │
│  │  │ WA  │ │ TG   │ │Discord │ │Slack │ ...       │  │
│  │  └─────┘ └──────┘ └────────┘ └──────┘           │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Agent Layer                          │  │
│  │  - 统一的 Agent 逻辑                              │  │
│  │  - 通道无关的响应生成                             │  │
│  │  - 格式化适配                                     │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 配置示例

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "priority": 1,
      "allowFrom": ["+8613800138000"]
    },
    "telegram": {
      "enabled": true,
      "priority": 2,
      "allowFrom": ["@username"]
    },
    "discord": {
      "enabled": true,
      "priority": 3,
      "guilds": {
        "allow": ["123456789012345678"]
      }
    },
    "slack": {
      "enabled": true,
      "priority": 4,
      "workspaces": {
        "allow": ["T12345678"]
      }
    }
  }
}
```

---

## 通道优先级

### 优先级概念

当用户在多个通道中活跃时，优先级决定：

1. **主要通道** - 首选通信方式
2. **备用通道** - 主要通道不可用时使用
3. **紧急通道** - 紧急通知使用

### 配置优先级

```json
{
  "channels": {
    "whatsapp": {
      "priority": 1,
      "role": "primary",
      "description": "主要通信通道"
    },
    "telegram": {
      "priority": 2,
      "role": "secondary",
      "description": "备用通道"
    },
    "discord": {
      "priority": 3,
      "role": "community",
      "description": "社区讨论"
    }
  }
}
```

### 优先级规则

```json
{
  "routing": {
    "priorityRules": {
      "workHours": {
        "primary": "slack",
        "fallback": "teams"
      },
      "personalHours": {
        "primary": "whatsapp",
        "fallback": "telegram"
      },
      "emergency": {
        "channels": ["whatsapp", "telegram"],
        "ignoreOthers": true
      }
    }
  }
}
```

---

## 消息路由

### 路由策略

#### 基于内容路由

```json
{
  "routing": {
    "contentBased": {
      "enabled": true,
      "rules": [
        {
          "pattern": "(?i)\\b(bug|error|fix)\\b",
          "agent": "coding",
          "priority": "high"
        },
        {
          "pattern": "(?i)\\b(design|ui|ux)\\b",
          "agent": "design",
          "priority": "medium"
        },
        {
          "pattern": "(?i)\\b(help|support|question)\\b",
          "agent": "support",
          "priority": "low"
        }
      ]
    }
  }
}
```

#### 基于通道路由

```json
{
  "routing": {
    "channelBased": {
      "whatsapp": {
        "defaultAgent": "personal",
        "rules": {
          "groups": "community"
        }
      },
      "slack": {
        "defaultAgent": "work",
        "rules": {
          "#general": "general",
          "#dev": "coding"
        }
      },
      "discord": {
        "defaultAgent": "community",
        "rules": {}
      }
    }
  }
}
```

#### 基于时间路由

```json
{
  "routing": {
    "timeBased": {
      "enabled": true,
      "schedule": [
        {
          "name": "workHours",
          "hours": "9-18",
          "days": ["Mon", "Tue", "Wed", "Thu", "Fri"],
          "agent": "work"
        },
        {
          "name": "personalHours",
          "hours": "18-9",
          "agent": "personal"
        },
        {
          "name": "weekend",
          "days": ["Sat", "Sun"],
          "agent": "casual"
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
        "routing": {
          "analyzeContext": true,
          "considerUrgency": true,
          "checkAvailability": true,
          "learnFromHistory": true
        }
      }
    ]
  }
}
```

---

## 统一响应格式

### 格式化模板

```json
{
  "channels": {
    "defaults": {
      "responseFormat": {
        "includeSource": false,
        "includeTimestamp": true,
        "includeAgent": false
      }
    },
    "whatsapp": {
      "responseFormat": {
        "style": "concise",
        "maxLength": 1000,
        "markdown": false,
        "emoji": true
      }
    },
    "telegram": {
      "responseFormat": {
        "style": "standard",
        "markdown": true,
        "html": false,
        "emoji": true
      }
    },
    "discord": {
      "responseFormat": {
        "style": "rich",
        "embeds": true,
        "markdown": true,
        "emoji": true
      }
    },
    "slack": {
      "responseFormat": {
        "style": "professional",
        "markdown": true,
        "blocks": true,
        "emoji": true
      }
    }
  }
}
```

### 响应适配器

```markdown
# TOOLS.md

## 响应格式化

### 通用规则
1. 根据通道自动调整格式
2. 保留核心内容
3. 适配通道特性

### 通道特定适配

**WhatsApp**
- 限制消息长度（1000 字符）
- 使用简单文本（无 Markdown）
- 适当使用表情

**Telegram/Discord**
- 支持 Markdown
- 可以使用代码块
- 支持链接和格式

**Slack**
- 使用 Slack Blocks
- 专业语调
- 结构化输出
```

---

## 通道特定配置

### WhatsApp 特定

```json
{
  "channels": {
    "whatsapp": {
      "businessProfile": {
        "name": "AI Assistant",
        "description": "Your Personal AI",
        "photo": "/path/to/photo.jpg"
      },
      "templateMessages": {
        "enabled": true,
        "templates": {
          "greeting": "Hello! How can I help you today?",
          "farewell": "Have a great day!",
          "error": "Sorry, something went wrong. Please try again."
        }
      }
    }
  }
}
```

### Telegram 特定

```json
{
  "channels": {
    "telegram": {
      "customKeyboard": {
        "enabled": true,
        "keyboard": [
          ["Help", "Status"],
          ["Ask", "Clear"]
        ]
      },
      "inlineMode": {
        "enabled": true,
        "placeholder": "Ask AI..."
      }
    }
  }
}
```

### Discord 特定

```json
{
  "channels": {
    "discord": {
      "slashCommands": true,
      "embeds": {
        "enabled": true,
        "color": 5814783,
        "thumbnail": true
      },
      "components": {
        "buttons": true,
        "selectMenus": true,
        "modals": true
      }
    }
  }
}
```

### Slack 特定

```json
{
  "channels": {
    "slack": {
      "appHome": {
        "enabled": true,
        "tabs": ["home", "messages"]
      },
      "shortcuts": {
        "global": [
          {
            "name": "Ask AI",
            "callbackId": "ask_ai",
            "description": "Ask the AI a question"
          }
        ]
      },
      "workflowSteps": {
        "enabled": true
      }
    }
  }
}
```

---

## 跨通道会话

### 统一会话 ID

```json
{
  "sessions": {
    "unified": {
      "enabled": true,
      "strategy": "userBased",
      "userIdFormat": "user:{platform}:{userId}",
      "crossChannel": {
        "enabled": true,
        "sync": true,
        "context": true
      }
    }
  }
}
```

### 会话同步

```json
{
  "sessions": {
    "sync": {
      "enabled": true,
      "channels": ["whatsapp", "telegram", "slack"],
      "syncInterval": 60,
      "syncContent": true,
      "syncContext": true
    }
  }
}
```

### 使用场景

```
用户在 WhatsApp 开始对话：
User: "帮我写一个 Python 函数"

切换到 Slack 继续对话：
User: "继续上面的任务"
AI: "好的，这是你之前请求的 Python 函数..."

上下文保持同步！
```

---

## 通道监控

### 健康检查

```json
{
  "monitoring": {
    "channels": {
      "enabled": true,
      "checkInterval": 30,
      "alerts": {
        "down": true,
        "slow": true,
        "error": true
      },
      "thresholds": {
        "responseTime": 5000,
        "errorRate": 0.05
      }
    }
  }
}
```

### 监控命令

```bash
# 检查所有通道状态
openclaw channels status

# 检查特定通道
openclaw channel whatsapp health

# 查看通道统计
openclaw channels stats

# 查看通道使用情况
openclaw channels usage

# 设置通道告警
openclaw channels alert set --channel whatsapp --type down --notify email:user@example.com
```

### 仪表板

```bash
# 打开监控仪表板
openclaw monitor

# 查看实时指标
openclaw monitor --realtime

# 查看历史数据
openclaw monitor --history 7d
```

---

## 最佳实践

### 1. 渐进式部署

```markdown
## 部署顺序

1. **测试阶段**
   - 只启用 Telegram（最简单）
   - 测试基本功能
   - 收集反馈

2. **扩展阶段**
   - 添加 WhatsApp
   - 测试群组功能
   - 验证媒体处理

3. **企业阶段**
   - 集成 Slack
   - 配置工作流
   - 设置监控

4. **社区阶段**
   - 添加 Discord
   - 配置社区规则
   - 设置权限
```

### 2. 一致的用户体验

```markdown
## 保持一致性

### 命名规范
- 使用一致的 Bot 名称
- 统一的命令前缀
- 相同的帮助格式

### 响应风格
- 统一的语调
- 相似的结构
- 一致的表情使用

### 功能对等
- 核心功能在所有通道可用
- 通道特定功能明确标识
- 能力限制清晰说明
```

### 3. 通道特定优化

```markdown
## 优化建议

### WhatsApp
- 保持消息简洁
- 使用模板消息
- 优化图片大小

### Telegram
- 利用内联模式
- 使用自定义键盘
- 启用深链接

### Discord
- 使用 Slash 命令
- 创建丰富的嵌入
- 添加交互组件

### Slack
- 使用 App Home
- 创建快捷方式
- 利用 Blocks
```

### 4. 安全考虑

```json
{
  "channels": {
    "defaults": {
      "security": {
        "rateLimit": {
          "enabled": true,
          "maxMessages": 10,
          "period": 60
        },
        "spamFilter": {
          "enabled": true,
          "threshold": 5
        },
        "suspiciousActivity": {
          "block": true,
          "notify": true
        }
      }
    }
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ 多通道架构 - 统一的消息路由和 Agent 层
2. ✅ 通道优先级 - 主要、备用、紧急通道
3. ✅ 消息路由 - 基于内容、通道、时间的路由
4. ✅ 统一响应格式 - 通道特定的格式化
5. ✅ 通道特定配置 - 各通道的专门设置
6. ✅ 跨通道会话 - 统一会话和同步
7. ✅ 通道监控 - 健康检查和告警
8. ✅ 最佳实践 - 渐进部署、一致性、优化、安全

## 第三阶段完成！

恭喜！你已经完成了通道集成学习：

- ✅ WhatsApp 集成
- ✅ Telegram 集成
- ✅ Discord 集成
- ✅ 其他通道（iMessage, Slack, Signal 等）
- ✅ 多通道管理

## 下一步

[第四阶段：Agent 定制](../stage4-agent/12-agent-loop.md) - 深入理解 Agent Loop 工作原理
