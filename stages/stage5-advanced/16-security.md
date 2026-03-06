# 沙箱与安全

> 学习目标：理解并配置安全沙箱

## 目录

- [安全概述](#安全概述)
- [Sandbox 模式](#sandbox-模式)
- [工具策略](#工具策略)
- [Elevated 模式](#elevated-模式)
- [安全最佳实践](#安全最佳实践)
- [安全审计](#安全审计)
- [合规性](#合规性)
- [小结](#小结)

---

## 安全概述

### 安全威胁模型

```
威胁来源:
├── 外部威胁
│   ├── 未授权访问
│   ├── 恶意输入
│   └── 攻击尝试
├── 内部威胁
│   ├── 意外操作
│   ├── 权限滥用
│   └── 数据泄露
└── 系统威胁
    ├── 资源耗尽
    ├── 并发问题
    └── 配置错误
```

### 安全层级

```
防御层级:
1. 网络层
   ├── TLS/SSL 加密
   ├── API 认证
   └── 速率限制

2. 应用层
   ├── 输入验证
   ├── 输出过滤
   └── 会话管理

3. Agent 层
   ├── 沙箱隔离
   ├── 工具限制
   └── 权限控制

4. 数据层
   ├── 加密存储
   ├── 访问控制
   └── 审计日志
```

---

## Sandbox 模式

### 什么是 Sandbox？

Sandbox 是一个受限的执行环境，限制 Agent 的操作权限。

### Sandbox 级别

#### 级别 1：完全开放

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "enabled": false
      }
    }
  }
}
```

**特点**：
- 无限制
- 适用于可信环境
- 不推荐用于生产环境

#### 级别 2：基本限制

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "enabled": true,
        "level": "basic",
        "restrictions": {
          "fileSystem": {
            "allowedPaths": ["/workspace", "/tmp"],
            "deniedPaths": ["/etc", "/var", "/sys"],
            "maxFileSize": 10485760
          },
          "network": {
            "allowedDomains": ["api.example.com"],
            "deniedDomains": ["localhost", "internal.com"]
          },
          "commands": {
            "allowed": ["python", "node", "git"],
            "denied": ["rm", "dd", "mkfs"]
          }
        }
      }
    }
  }
}
```

**特点**：
- 路径限制
- 网络限制
- 命令白名单

#### 级别 3：严格限制

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "enabled": true,
        "level": "strict",
        "filesystem": {
          "readOnly": false,
          "allowedPaths": ["~/.openclaw/workspace"],
          "maxFileSize": 1048576,
          "maxTotalSize": 10485760
        },
        "network": {
          "enabled": false
        },
        "execution": {
          "maxProcesses": 1,
          "maxMemory": "512M",
          "maxCpuTime": 30
        }
      }
    }
  }
}
```

**特点**：
- 严格的文件系统访问
- 禁用网络
- 资源限制

### 沙箱配置选项

```json
{
  "sandbox": {
    "filesystem": {
      "enabled": true,
      "allowedPaths": ["path1", "path2"],
      "deniedPaths": ["path3", "path4"],
      "maxFileSize": 1048576,
      "maxTotalSize": 10485760,
      "readOnly": false,
      "quota": {
        "enabled": true,
        "soft": 1048576,
        "hard": 10485760
      }
    },
    "network": {
      "enabled": true,
      "allowedDomains": ["*.example.com"],
      "deniedDomains": ["malicious.com"],
      "allowedPorts": [443, 80],
      "maxConnections": 10,
      "maxBandwidth": 1048576
    },
    "execution": {
      "enabled": true,
      "allowedCommands": ["python", "node"],
      "deniedCommands": ["rm", "dd"],
      "maxProcesses": 3,
      "maxMemory": "1G",
      "maxCpuTime": 60,
      "maxWallTime": 120
    },
    "environment": {
      "allowedVars": ["PATH", "HOME"],
      "deniedVars": ["API_KEY", "SECRET"],
      "maxVars": 100
    }
  }
}
```

---

## 工具策略

### 工具白名单/黑名单

```json
{
  "agents": {
    "defaults": {
      "tools": {
        "strategy": "allowlist",
        "allow": ["read", "write", "edit", "web_search"],
        "deny": ["exec", "process", "message"]
      }
    }
  }
}
```

### 工具配置文件

```json
{
  "tools": {
    "profiles": {
      "safe": {
        "description": "只读工具",
        "allow": ["read", "web_search", "web_fetch"],
        "deny": ["*"]
      },
      "cautious": {
        "description": "需要确认的写入操作",
        "allow": ["read", "write", "edit"],
        "deny": ["exec", "process"],
        "confirmation": {
          "write": true,
          "edit": true
        }
      },
      "privileged": {
        "description": "完全访问",
        "allow": ["*"],
        "confirmation": {
          "exec": true,
          "process": true,
          "message": true
        }
      }
    }
  }
}
```

### 工具权限矩阵

| 工具 | Safe | Cautious | Privileged |
|-----|------|----------|------------|
| read | ✅ | ✅ | ✅ |
| write | ❌ | ✅* | ✅ |
| edit | ❌ | ✅* | ✅ |
| exec | ❌ | ❌ | ✅* |
| process | ❌ | ❌ | ✅* |
| web_search | ✅ | ✅ | ✅ |
| web_fetch | ✅ | ✅ | ✅ |
| message | ❌ | ❌ | ✅* |

*需要确认

### 工具使用确认

```json
{
  "tools": {
    "confirmation": {
      "enabled": true,
      "tools": {
        "write": {
          "enabled": true,
          "message": "即将写入文件 {path}，是否继续？"
        },
        "exec": {
          "enabled": true,
          "message": "即将执行命令：{command}，是否继续？"
        },
        "message": {
          "enabled": true,
          "message": "即将发送消息到 {channel}，是否继续？"
        }
      },
      "timeout": 300,
      "autoReject": true
    }
  }
}
```

---

## Elevated 模式

### 什么是 Elevated 模式？

Elevated 模式允许 Agent 执行需要特殊权限的操作，需要显式授权。

### 配置 Elevated 模式

```json
{
  "agents": {
    "defaults": {
      "elevated": {
        "enabled": true,
        "tools": ["exec", "process", "message", "cron", "gateway"],
        "authorization": {
          "required": true,
          "method": "prompt",
          "timeout": 300
        }
      }
    }
  }
}
```

### 授权流程

```
1. Agent 请求 elevated 操作
       ↓
2. 检查是否需要授权
       ↓
3. 发送授权请求给用户
       ↓
4. 用户批准或拒绝
       ↓
5a. 批准：执行操作
5b. 拒绝：返回错误
```

### 授权方法

#### 方法 1：交互式提示

```json
{
  "authorization": {
    "method": "prompt",
    "message": "Agent 请求执行：{operation}\n详情：{details}\n是否批准？",
    "options": ["批准", "拒绝", "永久批准此工具"]
  }
}
```

#### 方法 2：API 端点

```json
{
  "authorization": {
    "method": "api",
    "endpoint": "/api/authorize",
    "token": "${AUTH_TOKEN}"
  }
}
```

#### 方法 3：预授权令牌

```json
{
  "authorization": {
    "method": "token",
    "tokens": {
      "exec": "${ELEVATED_EXEC_TOKEN}",
      "process": "${ELEVATED_PROCESS_TOKEN}"
    }
  }
}
```

### 会话级别授权

```json
{
  "authorization": {
    "session": {
      "enabled": true,
      "duration": 3600,
      "maxOperations": 10,
      "revokeable": true
    }
  }
}
```

---

## 安全最佳实践

### 1. 最小权限原则

```json
{
  "agents": {
    "list": [
      {
        "id": "reader",
        "tools": {
          "allow": ["read", "web_search"]
        }
      },
      {
        "id": "writer",
        "tools": {
          "allow": ["read", "write", "edit"],
          "confirmation": {
            "write": true,
            "edit": true
          }
        }
      }
    ]
  }
}
```

### 2. 防御深度

```json
{
  "security": {
    "layers": [
      {
        "name": "network",
        "enabled": true,
        "config": {
          "tls": true,
          "rateLimit": true
        }
      },
      {
        "name": "input",
        "enabled": true,
        "config": {
          "validation": true,
          "sanitization": true
        }
      },
      {
        "name": "sandbox",
        "enabled": true,
        "config": {
          "level": "strict"
        }
      },
      {
        "name": "audit",
        "enabled": true,
        "config": {
          "log": true,
          "alert": true
        }
      }
    ]
  }
}
```

### 3. 输入验证

```markdown
# AGENTS.md

## 输入验证规则

### 文件路径
- 必须在允许的路径内
- 不能包含 .. (父目录引用)
- 长度限制：256 字符

### 命令
- 必须在白名单中
- 参数必须验证
- 不能包含管道操作符

### URL
- 必须使用 HTTPS
- 必须在允许的域名内
- 不能包含私有 IP 地址

### 用户输入
- 限制长度
- 过滤危险字符
- 验证格式
```

### 4. 输出过滤

```markdown
## 输出过滤规则

### 敏感信息
- 不输出 API 密钥
- 不输出密码
- 不输出私人信息

### 代码执行
- 不输出可执行代码（除非请求）
- 警告用户运行代码的风险
- 提供安全使用指南

### 文件操作
- 警告覆盖现有文件
- 确认删除操作
- 显示将要修改的内容
```

### 5. 审计日志

```json
{
  "audit": {
    "enabled": true,
    "log": {
      "file": "~/.openclaw/audit.log",
      "format": "json",
      "rotation": {
        "enabled": true,
        "maxSize": "100M",
        "maxFiles": 10
      }
    },
    "events": [
      "agent.start",
      "agent.stop",
      "tool.call",
      "elevated.request",
      "authorization.grant",
      "authorization.deny",
      "security.violation"
    ],
    "alert": {
      "enabled": true,
      "events": ["security.violation", "authorization.deny"],
      "notification": ["email", "webhook"]
    }
  }
}
```

---

## 安全审计

### 审计检查清单

```bash
# 运行安全审计
openclaw audit security

# 输出:
# ✓ 输入验证: 启用
# ✓ 输出过滤: 启用
# ✓ 沙箱: 启用 (级别: strict)
# ⚠ 工具确认: 部分启用
# ✓ 审计日志: 启用
# ⚠ API 密钥: 未加密
```

### 审计命令

```bash
# 完整安全审计
openclaw audit security --full

# 审计特定 Agent
openclaw audit agent coding

# 审计工具权限
openclaw audit tools

# 生成审计报告
openclaw audit report --output security-report.pdf
```

### 安全评分

```bash
# 查看安全评分
openclaw audit score

# 输出:
# 安全评分: 85/100
#
# 分数详情:
# 网络安全: 90/100
# 应用安全: 85/100
# Agent 安全: 80/100
# 数据安全: 90/100
```

### 改进建议

```bash
# 获取安全改进建议
openclaw audit suggest

# 输出:
# 建议改进:
# 1. 启用工具确认
# 2. 加密 API 密钥存储
# 3. 添加速率限制
# 4. 定期轮换密钥
```

---

## 合规性

### GDPR 合规

```json
{
  "compliance": {
    "gdpr": {
      "enabled": true,
      "dataRetention": {
        "days": 30,
        "autoDelete": true
      },
      "rightToErasure": {
        "enabled": true,
        "endpoint": "/api/gdpr/erase"
      },
      "consent": {
        "required": true,
        "logging": true
      }
    }
  }
}
```

### SOC 2 合规

```json
{
  "compliance": {
    "soc2": {
      "enabled": true,
      "controls": {
        "access": {
          "mfa": true,
          "sessionTimeout": 3600
        },
        "encryption": {
          "atRest": true,
          "inTransit": true
        },
        "monitoring": {
          "enabled": true,
          "alerting": true
        }
      }
    }
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ 安全概述 - 威胁模型和防御层级
2. ✅ Sandbox 模式 - 三种安全级别和配置
3. ✅ 工具策略 - 白名单/黑名单和权限矩阵
4. ✅ Elevated 模式 - 特殊权限和授权流程
5. ✅ 安全最佳实践 - 最小权限、防御深度、输入验证、输出过滤、审计日志
6. ✅ 安全审计 - 检查清单、命令、评分、改进建议
7. ✅ 合规性 - GDPR 和 SOC 2

## 安全检查清单

- [ ] 启用 Sandbox
- [ ] 配置工具白名单
- [ ] 启用工具确认
- [ ] 配置 Elevated 模式
- [ ] 启用审计日志
- [ ] 设置安全告警
- [ ] 定期安全审计
- [ ] 加密敏感数据

## 下一步

[插件系统](./17-plugins.md) - 通过插件扩展功能
