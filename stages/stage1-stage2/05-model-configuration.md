# 模型配置

> 学习目标：配置和优化 AI 模型

## 目录

- [支持的 Provider](#支持的-provider)
- [Provider 配置](#provider-配置)
- [模型选择策略](#模型选择策略)
- [模型参数优化](#模型参数优化)
- [成本优化](#成本优化)
- [Fallback 机制](#fallback-机制)
- [模型性能对比](#模型性能对比)
- [小结](#小结)

---

## 支持的 Provider

OpenClaw 支持多个 LLM Provider：

| Provider | 模型 | 特点 | 成本 |
|---------|-----|------|------|
| **OpenAI** | GPT-4o, GPT-4o-mini | 强大、通用 | $ |
| **Anthropic** | Claude Sonnet/Opus/Haiku | 安全、长上下文 | $$ |
| **OpenRouter** | 多种模型 | 灵活、聚合 | $$ |
| **Google AI** | Gemini Pro | 多模态 | $ |
| **AWS Bedrock** | 多种模型 | 企业级 | $$$ |
| **本地模型** | Llama, Mistral | 隐私、免费 | 免费 |

---

## Provider 配置

### OpenAI 配置

```json
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "${OPENAI_API_KEY}",
        "baseURL": "https://api.openai.com/v1",
        "models": {
          "chat": "gpt-4o",
          "fast": "gpt-4o-mini",
          "image": "dall-e-3"
        },
        "parameters": {
          "temperature": 0.7,
          "maxTokens": 4096
        }
      }
    }
  }
}
```

### Anthropic 配置

```json
{
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}",
        "baseURL": "https://api.anthropic.com",
        "version": "2023-06-01",
        "models": {
          "chat": "claude-sonnet-4-20250514",
          "smart": "claude-opus-4-20250514",
          "fast": "claude-haiku-4-20250514"
        },
        "parameters": {
          "temperature": 0.7,
          "maxTokens": 8192
        }
      }
    }
  }
}
```

### OpenRouter 配置

```json
{
  "models": {
    "providers": {
      "openrouter": {
        "apiKey": "${OPENROUTER_API_KEY}",
        "baseURL": "https://openrouter.ai/api/v1",
        "models": {
          "chat": "anthropic/claude-sonnet-4",
          "fallback": "openai/gpt-4o"
        },
        "parameters": {
          "temperature": 0.7,
          "maxTokens": 4096,
          "retryAttempts": 3
        }
      }
    }
  }
}
```

### Google AI 配置

```json
{
  "models": {
    "providers": {
      "google": {
        "apiKey": "${GOOGLE_API_KEY}",
        "baseURL": "https://generativelanguage.googleapis.com/v1beta",
        "models": {
          "chat": "gemini-pro"
        }
      }
    }
  }
}
```

### AWS Bedrock 配置

```json
{
  "models": {
    "providers": {
      "bedrock": {
        "region": "us-east-1",
        "accessKeyId": "${AWS_ACCESS_KEY_ID}",
        "secretAccessKey": "${AWS_SECRET_ACCESS_KEY}",
        "models": {
          "chat": "anthropic.claude-3-sonnet-20240229-v1:0"
        }
      }
    }
  }
}
```

### 本地模型配置 (Ollama)

```json
{
  "models": {
    "providers": {
      "ollama": {
        "baseURL": "http://localhost:11434",
        "models": {
          "chat": "llama3:8b",
          "smart": "llama3:70b"
        }
      }
    }
  }
}
```

---

## 模型选择策略

### 按任务类型选择

```json
{
  "agents": {
    "list": [
      {
        "id": "coding",
        "model": "anthropic/claude-sonnet-4-20250514",  // 代码任务
        "description": "擅长代码理解和生成"
      },
      {
        "id": "writing",
        "model": "openai/gpt-4o",  // 写作任务
        "description": "擅长创意写作"
      },
      {
        "id": "quick",
        "model": "openai/gpt-4o-mini",  // 快速响应
        "description": "快速简单任务"
      },
      {
        "id": "analysis",
        "model": "anthropic/claude-opus-4-20250514",  // 深度分析
        "description": "复杂分析和推理"
      }
    ]
  }
}
```

### 按成本选择

```json
{
  "models": {
    "tiers": {
      "free": {
        "provider": "ollama",
        "model": "llama3:8b",
        "description": "本地免费模型"
      },
      "low": {
        "provider": "openai",
        "model": "gpt-4o-mini",
        "description": "低成本快速模型"
      },
      "medium": {
        "provider": "anthropic",
        "model": "claude-haiku-4-20250514",
        "description": "中等成本平衡模型"
      },
      "high": {
        "provider": "anthropic",
        "model": "claude-sonnet-4-20250514",
        "description": "高性能模型"
      }
    }
  }
}
```

### 动态模型选择

```json
{
  "agents": {
    "list": [
      {
        "id": "adaptive",
        "model": "auto",  // 自动选择
        "modelSelection": {
          "strategy": "cost-aware",
          "thresholds": {
            "simple": "low",
            "medium": "medium",
            "complex": "high"
          }
        }
      }
    ]
  }
}
```

---

## 模型参数优化

### Temperature 参数

控制响应的随机性：

```json
{
  "agents": {
    "list": [
      {
        "id": "creative",
        "model": "openai/gpt-4o",
        "temperature": 0.9,  // 高创造性
        "description": "创意写作、头脑风暴"
      },
      {
        "id": "balanced",
        "model": "openai/gpt-4o",
        "temperature": 0.7,  // 平衡
        "description": "一般任务"
      },
      {
        "id": "precise",
        "model": "openai/gpt-4o",
        "temperature": 0.1,  // 低随机性
        "description": "代码、精确回答"
      }
    ]
  }
}
```

### Max Tokens 参数

控制响应长度：

```json
{
  "agents": {
    "list": [
      {
        "id": "concise",
        "maxTokens": 512,  // 简短回答
        "description": "快速简短响应"
      },
      {
        "id": "detailed",
        "maxTokens": 4096,  // 详细回答
        "description": "需要详细解释"
      },
      {
        "id": "longform",
        "maxTokens": 8192,  // 长内容生成
        "description": "文章、报告生成"
      }
    ]
  }
}
```

### Top P 参数

核采样参数：

```json
{
  "agents": {
    "list": [
      {
        "id": "focused",
        "topP": 0.5,  // 更专注
        "description": "专注特定主题"
      },
      {
        "id": "diverse",
        "topP": 0.95,  // 更多样化
        "description": "探索更多可能性"
      }
    ]
  }
}
```

---

## 成本优化

### Token 使用追踪

```json
{
  "models": {
    "monitoring": {
      "enabled": true,
      "alertThreshold": 1000000,  // 1M tokens 警告
      "resetPeriod": "monthly",
      "reports": {
        "enabled": true,
        "frequency": "weekly",
        "format": "json"
      }
    }
  }
}
```

查看使用情况：

```bash
# 查看 token 使用
openclaw stats tokens

# 查看成本
openclaw stats cost

# 查看详细报告
openclaw stats report --format table
```

### 模型切换策略

```json
{
  "agents": {
    "list": [
      {
        "id": "cost-optimized",
        "model": "anthropic/claude-haiku-4-20250514",  // 默认低成本
        "upgradeTrigger": {
          "complexity": "high",  // 复杂任务升级
          "upgradeTo": "anthropic/claude-sonnet-4-20250514"
        }
      }
    ]
  }
}
```

### Session Pruning

自动修剪会话历史以减少 token 使用：

```json
{
  "sessions": {
    "pruning": {
      "enabled": true,
      "strategy": "smart",  // smart, recent, summary
      "maxTokens": 50000,
      "keepRecent": 10,
      "summarizeOld": true
    }
  }
}
```

### 缓存策略

```json
{
  "performance": {
    "cache": {
      "enabled": true,
      "strategy": "lru",
      "maxSize": 1000,
      "ttl": 3600,  // 1 小时
      "cachePrompts": true,
      "cacheResponses": true
    }
  }
}
```

---

## Fallback 机制

### 自动 Fallback

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
      "retryDelay": 1000,
      "exponentialBackoff": true
    }
  }
}
```

### 条件 Fallback

```json
{
  "agents": {
    "list": [
      {
        "id": "resilient",
        "model": "anthropic/claude-sonnet-4-20250514",
        "fallback": {
          "onError": "openai/gpt-4o",
          "onTimeout": "openai/gpt-4o-mini",
          "onRateLimit": "openrouter/anthropic/claude-sonnet-4"
        }
      }
    ]
  }
}
```

### 手动 Fallback

```bash
# 设置备用模型
openclaw config set models.fallback.providers[0] anthropic
openclaw config set models.fallback.providers[1] openai
openclaw config set models.fallback.maxRetries 5

# 测试 fallback
openclaw test fallback
```

---

## 模型性能对比

### 速度对比

| 模型 | 响应时间 | 适用场景 |
|-----|---------|---------|
| GPT-4o-mini | 快 | 简单任务、快速响应 |
| Claude Haiku | 快 | 轻量级任务 |
| GPT-4o | 中等 | 通用任务 |
| Claude Sonnet | 中等 | 复杂任务 |
| Claude Opus | 慢 | 最复杂任务 |
| Llama 3 8B (本地) | 中等 | 隐私敏感任务 |

### 质量对比

| 模型 | 代码 | 写作 | 推理 | 数学 |
|-----|-----|------|------|------|
| Claude Sonnet | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| GPT-4o | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Claude Opus | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| GPT-4o-mini | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Claude Haiku | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

### 成本对比 (每 1M tokens)

| 模型 | 输入 | 输出 | 性价比 |
|-----|------|------|-------|
| GPT-4o-mini | $0.15 | $0.60 | ⭐⭐⭐⭐⭐ |
| Claude Haiku | $0.25 | $1.25 | ⭐⭐⭐⭐ |
| GPT-4o | $2.50 | $10.00 | ⭐⭐⭐ |
| Claude Sonnet | $3.00 | $15.00 | ⭐⭐⭐ |
| Claude Opus | $15.00 | $75.00 | ⭐⭐ |
| Llama 3 8B | $0.00 | $0.00 | ⭐⭐⭐⭐⭐ |

---

## 最佳实践

### 选择建议

1. **日常使用** - Claude Sonnet 4 或 GPT-4o
2. **代码任务** - Claude Sonnet 4（最佳代码理解）
3. **快速响应** - GPT-4o-mini 或 Claude Haiku
4. **复杂任务** - Claude Opus 4
5. **隐私敏感** - 本地模型（Llama 3）
6. **成本敏感** - GPT-4o-mini 或本地模型

### 配置建议

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.7,
      "maxTokens": 4096
    },
    "list": [
      {
        "id": "default",
        "model": "anthropic/claude-sonnet-4-20250514"
      },
      {
        "id": "fast",
        "model": "openai/gpt-4o-mini",
        "maxTokens": 2048
      },
      {
        "id": "coding",
        "model": "anthropic/claude-sonnet-4-20250514",
        "temperature": 0.2
      }
    ]
  },
  "models": {
    "fallback": {
      "enabled": true,
      "providers": ["anthropic", "openai"]
    },
    "monitoring": {
      "enabled": true,
      "alertThreshold": 500000
    }
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ 支持的 Provider - OpenAI、Anthropic、OpenRouter 等
2. ✅ Provider 配置 - 各 Provider 的详细配置
3. ✅ 模型选择策略 - 按任务、成本、动态选择
4. ✅ 模型参数优化 - Temperature、Max Tokens、Top P
5. ✅ 成本优化 - Token 追踪、模型切换、Session Pruning
6. ✅ Fallback 机制 - 自动和手动 Fallback
7. ✅ 模型性能对比 - 速度、质量、成本

## 下一步

[Agent Workspace 配置](./06-agent-workspace.md) - 定制 Agent 行为
