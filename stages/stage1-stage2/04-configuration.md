# 配置文件详解

> 学习目标：掌握 openclaw.json 的核心配置

## 目录

- [配置文件位置](#配置文件位置)
- [配置文件结构](#配置文件结构)
- [Agents 配置](#agents-配置)
- [Channels 配置](#channels-配置)
- [Models 配置](#models-配置)
- [Tools 配置](#tools-配置)
- [安全配置](#安全配置)
- [高级配置](#高级配置)
- [配置验证](#配置验证)
- [小结](#小结)

---

## 配置文件位置

### 默认位置

OpenClaw 配置文件位于：

```
~/.openclaw/openclaw.json
```

### 自定义位置

可以通过环境变量指定自定义位置：

```bash
export OPENCLAW_CONFIG_PATH="/path/to/custom/openclaw.json"
```

### 配置文件目录结构

```
~/.openclaw/
├── openclaw.json          # 主配置文件
├── workspace/             # Agent workspace
│   ├── AGENTS.md
│   ├── SOUL.md
│   ├── TOOLS.md
│   ├── IDENTITY.md
│   └── USER.md
├── sessions/              # 会话存储
│   ├── session-1.jsonl
│   └── session-2.jsonl
└── skills/                # Local skills
    └── my-skill/
```

---

## 配置文件结构

### 最小配置

最基本的配置只需要指定通道：

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+8613800138000"]
    }
  }
}
```

### 完整配置示例

```json
{
  "$schema": "https://openclaw.ai/schema/config.json",
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
        "model": "anthropic/claude-sonnet-4-20250514",
        "tools": {
          "profile": "coding"
        }
      }
    ]
  },
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "baseURL": "https://api.openai.com/v1"
      },
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      }
    }
  },
  "channels": {
    "whatsapp": {
      "allowFrom": ["+8613800138000"],
      "groupMentionTrigger": "@ai"
    },
    "telegram": {
      "allowFrom": ["@my_username"],
      "webhook": {
        "url": "https://my-server.com/webhook"
      }
    }
  },
  "tools": {
    "profiles": {
      "coding": {
        "allow": ["read", "write", "edit", "exec", "web_search"],
        "deny": ["message", "cron"]
      }
    }
  },
  "sessions": {
    "directory": "~/.openclaw/sessions",
    "pruning": {
      "enabled": true,
      "maxTokens": 100000
    }
  },
  "logging": {
    "level": "info",
    "file": "~/.openclaw/gateway.log"
  }
}
```

---

## Agents 配置

### defaults 配置

默认 Agent 的配置：

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "workspace": "~/.openclaw/workspace",
      "tools": {
        "profile": "full"
      },
      "systemPrompt": "You are a helpful AI assistant.",
      "temperature": 0.7,
      "maxTokens": 4096
    }
  }
}
```

### 多 Agent 配置

配置多个专门的 Agent：

```json
{
  "agents": {
    "list": [
      {
        "id": "coding",
        "name": "Coding Assistant",
        "model": "anthropic/claude-sonnet-4-20250514",
        "tools": {
          "profile": "coding"
        },
        "workspace": "~/.openclaw/workspace-coding",
        "routing": {
          "channels": ["whatsapp", "discord"],
          "keywords": ["code", "bug", "function", "debug"]
        }
      },
      {
        "id": "support",
        "name": "Customer Support",
        "model": "openai/gpt-4o",
        "tools": {
          "profile": "messaging",
          "allow": ["slack"]
        },
        "workspace": "~/.openclaw/workspace-support",
        "routing": {
          "channels": ["slack"]
        }
      },
      {
        "id": "research",
        "name": "Research Assistant",
        "model": "anthropic/claude-sonnet-4-20250514",
        "tools": {
          "profile": "full",
          "allow": ["web_search", "web_fetch", "pdf"]
        },
        "routing": {
          "keywords": ["research", "find", "search", "analyze"]
        }
      }
    ]
  }
}
```

### Agent 属性说明

| 属性 | 类型 | 说明 | 默认值 |
|-----|------|-----|-------|
| `id` | string | Agent 唯一标识符 | 必填 |
| `name` | string | Agent 显示名称 | id 值 |
| `model` | string | 使用的模型 | defaults.model |
| `workspace` | string | Agent workspace 路径 | defaults.workspace |
| `tools.profile` | string | 工具配置文件 | full |
| `tools.allow` | array | 允许的工具 | - |
| `tools.deny` | array | 禁止的工具 | - |
| `systemPrompt` | string | 系统提示词 | - |
| `temperature` | number | 温度参数 (0-1) | 0.7 |
| `maxTokens` | number | 最大 token 数 | 4096 |
| `routing` | object | 路由规则 | - |

---

## Channels 配置

### WhatsApp 配置

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+8613800138000", "+8613800138001"],
      "groupMentionTrigger": "@ai",
      "businessProfile": {
        "name": "My AI Assistant",
        "photo": "/path/to/photo.jpg"
      }
    }
  }
}
```

### Telegram 配置

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "allowFrom": ["@my_username", 123456789],
      "webhook": {
        "url": "https://my-server.com/webhook",
        "secret": "my-webhook-secret"
      }
    }
  }
}
```

### Discord 配置

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "guilds": {
        "allow": ["123456789012345678"],
        "deny": ["987654321098765432"]
      },
      "commands": {
        "prefix": "!",
        "commands": {
          "ask": {
            "description": "Ask the AI a question",
            "usage": "!ask <your question>"
          }
        }
      }
    }
  }
}
```

### Slack 配置

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-...",
      "botToken": "xoxb-...",
      "workspaces": {
        "allow": ["T12345678"]
      }
    }
  }
}
```

### 通道通用属性

| 属性 | 类型 | 说明 | 默认值 |
|-----|------|-----|-------|
| `enabled` | boolean | 是否启用通道 | true |
| `allowFrom` | array | 允许的用户/群组列表 | [] |
| `denyFrom` | array | 禁止的用户/群组列表 | [] |
| `webhook.url` | string | Webhook URL | - |
| `webhook.secret` | string | Webhook 密钥 | - |

---

## Models 配置

### Provider 配置

```json
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "baseURL": "https://api.openai.com/v1",
        "models": {
          "chat": "gpt-4o",
          "image": "dall-e-3"
        }
      },
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}",
        "baseURL": "https://api.anthropic.com",
        "models": {
          "chat": "claude-sonnet-4-20250514"
        }
      },
      "openrouter": {
        "apiKey": "${OPENROUTER_API_KEY}",
        "baseURL": "https://openrouter.ai/api/v1",
        "models": {
          "chat": "anthropic/claude-sonnet-4"
        }
      }
    }
  }
}
```

### 模型别名配置

```json
{
  "models": {
    "aliases": {
      "fast": "openai/gpt-3.5-turbo",
      "smart": "anthropic/claude-sonnet-4-20250514",
      "creative": "openai/gpt-4o"
    }
  }
}
```

使用别名：

```json
{
  "agents": {
    "defaults": {
      "model": "smart"  // 使用 anthropic/claude-sonnet-4-20250514
    }
  }
}
```

### Fallback 配置

```json
{
  "models": {
    "fallback": {
      "enabled": true,
      "providers": [
        "anthropic",
        "openai",
        "openrouter"
      ],
      "maxRetries": 3,
      "retryDelay": 1000
    }
  }
}
```

---

## Tools 配置

> ⚠️ **重要提示**：工具配置是 OpenClaw 中最容易出问题的地方之一。如果发现 Agent 无法使用工具（如文件操作、命令执行等），通常是 `tools.profile` 配置不正确。
>
> **快速诊断**：
> ```bash
> # 检查当前工具配置
> openclaw config get tools.profile
>
> # 如果显示 "messaging"，说明工具权限受限
> # 解决方法：设置为 full 模式
> openclaw config set tools.profile full
> openclaw gateway restart
> ```
>
> 详细问题排查请参考：[故障排查指南](../stage8-topics/31-troubleshooting.md#工具权限问题)

### 工具配置文件 (Profiles)

OpenClaw 提供了预定义的工具配置文件：

```json
{
  "tools": {
    "profiles": {
      "minimal": {
        "description": "仅基本工具",
        "allow": ["read", "write"]
      },
      "coding": {
        "description": "编程相关工具",
        "allow": [
          "read",
          "write",
          "edit",
          "exec",
          "apply_patch",
          "web_search",
          "web_fetch"
        ],
        "deny": ["message", "cron", "gateway"]
      },
      "messaging": {
        "description": "消息相关工具",
        "allow": [
          "read",
          "write",
          "message",
          "sessions_list",
          "sessions_history"
        ],
        "deny": ["exec", "process", "cron"]
      },
      "full": {
        "description": "所有工具",
        "allow": ["*"]
      }
    }
  }
}
```

### 自定义工具配置

```json
{
  "agents": {
    "list": [
      {
        "id": "custom",
        "tools": {
          "allow": ["read", "write", "web_search"],
          "deny": ["exec", "message"],
          "config": {
            "web_search": {
              "maxResults": 5,
              "timeout": 10000
            }
          }
        }
      }
    ]
  }
}
```

### 可用工具列表

| 工具名称 | 说明 | 类别 |
|---------|-----|------|
| `read` | 读取文件 | 文件系统 |
| `write` | 写入文件 | 文件系统 |
| `edit` | 编辑文件 | 文件系统 |
| `exec` | 执行命令 | 执行 |
| `process` | 进程管理 | 执行 |
| `web_search` | 网络搜索 | Web |
| `web_fetch` | 获取网页 | Web |
| `browser` | 浏览器 | Web |
| `message` | 发送消息 | 通信 |
| `sessions_*` | 会话管理 | 会话 |
| `cron` | 定时任务 | 调度 |
| `image` | 图像生成 | AI |
| `pdf` | PDF 处理 | 文档 |

---

## 安全配置

### allowFrom 白名单

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": [
        "+8613800138000",  // 个人号码
        "+8613800138001"   // 另一个号码
      ]
    }
  }
}
```

### 群组配置

```json
{
  "channels": {
    "whatsapp": {
      "groupMentionTrigger": "@ai",  // 需要 @ai 才会响应
      "groups": {
        "allow": ["Team Group", "Family Group"]
      }
    }
  }
}
```

### 配对机制

某些通道支持配对机制：

```json
{
  "channels": {
    "whatsapp": {
      "pairing": {
        "enabled": true,
        "code": "PAIR-1234",  // 配对码
        "timeout": 300        // 5 分钟超时
      }
    }
  }
}
```

---

## 高级配置

### Sessions 配置

```json
{
  "sessions": {
    "directory": "~/.openclaw/sessions",
    "format": "jsonl",
    "pruning": {
      "enabled": true,
      "maxTokens": 100000,
      "maxMessages": 100,
      "strategy": "recent"
    },
    "retention": {
      "days": 30
    }
  }
}
```

### Logging 配置

```json
{
  "logging": {
    "level": "info",  // debug, info, warn, error
    "file": "~/.openclaw/gateway.log",
    "console": true,
    "format": "json",
    "maxSize": "100M",
    "maxFiles": 10
  }
}
```

### Gateway 配置

```json
{
  "gateway": {
    "port": 18789,
    "host": "127.0.0.1",
    "controlUI": {
      "enabled": true,
      "auth": {
        "enabled": true,
        "username": "admin",
        "password": "password123"
      }
    }
  }
}
```

### Performance 配置

```json
{
  "performance": {
    "concurrency": {
      "maxSessions": 10,
      "maxMessagesPerSession": 5
    },
    "queue": {
      "enabled": true,
      "maxSize": 100
    },
    "cache": {
      "enabled": true,
      "ttl": 3600
    }
  }
}
```

---

## 配置验证

### 验证命令

```bash
# 验证配置文件
openclaw config validate

# 查看当前配置
openclaw config get

# 查看特定配置
openclaw config get agents
openclaw config get channels.whatsapp

# 测试配置
openclaw doctor --config
```

### 配置测试

```bash
# 测试通道配置
openclaw test channel whatsapp

# 测试模型配置
openclaw test model anthropic/claude-sonnet-4-20250514

# 测试 Agent 配置
openclaw test agent coding
```

---

## 小结

本节我们学习了：

1. ✅ 配置文件位置和结构
2. ✅ Agents 配置 - 默认 Agent 和多 Agent
3. ✅ Channels 配置 - 各通道的详细配置
4. ✅ Models 配置 - Provider 和模型别名
5. ✅ Tools 配置 - 工具配置文件和自定义
6. ✅ 安全配置 - 白名单和配对机制
7. ✅ 高级配置 - Sessions、Logging、Performance
8. ✅ 配置验证 - 验证和测试配置

## 下一步

[模型配置](./05-model-configuration.md) - 配置和优化 AI 模型
