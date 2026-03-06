# 其他通道

> 学习要点：iMessage, Slack, Signal, Google Chat, 企业级通道

## 目录

- [通道对比](#通道对比)
- [iMessage (BlueBubbles)](#imessage-bluebubbles)
- [Slack](#slack)
- [Signal](#signal)
- [Google Chat](#google-chat)
- [Microsoft Teams](#microsoft-teams)
- [Mattermost](#mattermost)
- [IRC](#irc)
- [LINE](#line)
- [Matrix](#matrix)
- [小结](#小结)

---

## 通道对比

### 快速参考表

| 通道 | 配置难度 | 功能丰富度 | 成本 | 推荐度 | 适用场景 |
|-----|---------|-----------|------|-------|---------|
| **iMessage** | ⭐⭐⭐ 复杂 | ⭐⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ | Apple 生态 |
| **Slack** | ⭐⭐ 中等 | ⭐⭐⭐⭐⭐ | 付费 | ⭐⭐⭐⭐⭐ | 企业团队 |
| **Signal** | ⭐ 简单 | ⭐⭐⭐ | 免费 | ⭐⭐⭐ | 隐私优先 |
| **Google Chat** | ⭐⭐ 中等 | ⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ | Google Workspace |
| **Teams** | ⭐⭐⭐ 复杂 | ⭐⭐⭐⭐⭐ | 付费 | ⭐⭐⭐⭐ | Microsoft 365 |
| **Mattermost** | ⭐⭐ 中等 | ⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ | 自托管团队 |
| **IRC** | ⭐ 简单 | ⭐⭐ | 免费 | ⭐⭐ | 开源社区 |
| **LINE** | ⭐⭐ 中等 | ⭐⭐⭐ | 免费 | ⭐⭐⭐ | 亚洲市场 |
| **Matrix** | ⭐⭐⭐ 复杂 | ⭐⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ | 去中心化 |

### 选择建议

| 使用场景 | 推荐通道 |
|---------|---------|
| **企业协作** | Slack, Microsoft Teams |
| **Apple 用户** | iMessage |
| **隐私优先** | Signal, Matrix |
| **开源社区** | IRC, Mattermost |
| **Google 用户** | Google Chat |
| **亚洲市场** | LINE |
| **去中心化** | Matrix |

---

## iMessage (BlueBubbles)

### 概述

iMessage 需要通过 BlueBubbles Server 间接访问，因为 Apple 没有公开的 iMessage API。

### 准备条件

1. **Mac 设备** - 需要一台 Mac 运行 BlueBubbles
2. **BlueBubbles Server** - 安装并配置 BlueBubbles
3. **网络访问** - OpenClaw 能访问 BlueBubbles Server

### BlueBubbles 安装

```bash
# 1. 下载 BlueBubbles
# 访问 https://bluebubbles.app/download

# 2. 安装并启动 BlueBubbles

# 3. 配置服务器
# - 设置端口
# - 启用 API
# - 生成 API 密钥
```

### OpenClaw 配置

```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "bluebubbles": {
        "url": "http://your-mac:8080",
        "apiKey": "your-bluebubbles-api-key"
      },
      "allowFrom": ["+8613800138000"],
      "groupMentionTrigger": "@ai"
    }
  }
}
```

### 功能限制

| 功能 | 支持情况 |
|-----|---------|
| 文本消息 | ✅ |
| 图片 | ✅ |
| 视频 | ✅ |
| 反应 | ✅ |
| 群组 | ✅ |
| Tapbacks | ✅ |
| 手写 | ❌ |

---

## Slack

### 概述

Slack 是最流行的企业通信平台之一，功能丰富，API 完善。

### 准备条件

1. **Slack 工作区** - 创建或访问工作区
2. **Slack App** - 创建 Slack 应用
3. **Bot Token** - OAuth 或 Bot Token
4. **权限范围** - 必要的 scopes

### 创建 Slack App

1. 访问 https://api.slack.com/apps
2. 点击 **Create New App**
3. 选择 **From scratch**
4. 输入 App 名称和工作区
5. 配置 OAuth & Permissions
6. 安装到工作区

### OpenClaw 配置

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "${SLACK_BOT_TOKEN}",
      "signingSecret": "${SLACK_SIGNING_SECRET}",
      "appToken": "${SLACK_APP_TOKEN}",
      "workspaces": {
        "allow": ["T12345678"]
      },
      "commands": {
        "/ask": {
          "description": "Ask the AI a question",
          "usage": "/ask <your question>"
        }
      },
      "message": {
        "mentionRequired": true,
        "respondInThread": false
      }
    }
  }
}
```

### 高级功能

```json
{
  "channels": {
    "slack": {
      "features": {
        "appHome": true,        // App Home 页面
        "shortcuts": true,      // 全局快捷方式
        "modals": true,         // 模态框
        "views": true,          // 视图
        "workflowSteps": true   // Workflow Builder 步骤
      }
    }
  }
}
```

### 最佳实践

1. **使用 App Home** - 为用户提供个性化体验
2. **线程回复** - 在频道中使用线程回复
3. ** Slash 命令** - 提供易于记忆的命令
4. **快捷方式** - 创建全局快捷方式

---

## Signal

### 概述

Signal 是注重隐私的即时通讯应用，有开源的实现。

### 准备条件

1. **Signal Desktop** - 安装 Signal Desktop
2. **signal-cli** - 安装 signal-cli
3. **电话号码** - Signal 账号

### signal-cli 安装

```bash
# macOS
brew install signal-cli

# Linux (Ubuntu)
sudo apt install signal-cli

# 验证安装
signal-cli --version
```

### 注册 Signal

```bash
# 注册（验证码会发送到你的手机）
signal-cli -u +8613800138000 register

# 验证
signal-cli -u +8613800138000 verify <verification-code>

# 测试
signal-cli -u +8613800138000 send "+8613800138001" "Test message"
```

### OpenClaw 配置

```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "phoneNumber": "+8613800138000",
      "allowFrom": ["+8613800138001"],
      "groupMentionTrigger": "@ai",
      "groups": {
        "allow": ["groupId1", "groupId2"]
      }
    }
  }
}
```

### 特性

| 特性 | 支持情况 |
|-----|---------|
| 端到端加密 | ✅ |
| 文本消息 | ✅ |
| 图片 | ✅ |
| 视频 | ✅ |
| 群组 | ✅ |
| 语音消息 | ⚠️ |
| 通话 | ❌ |

---

## Google Chat

### 概述

Google Chat 是 Google Workspace 的一部分，适合企业环境。

### 准备条件

1. **Google Cloud 项目** - 创建 GCP 项目
2. **Google Chat API** - 启用 API
3. **服务账号** - 创建并配置
4. **Google Workspace** - 订阅（可选）

### 创建 Google Chat Bot

1. 访问 Google Cloud Console
2. 创建新项目或选择现有项目
3. 启用 Google Chat API
4. 配置 Chat 应用
5. 部署到 Google Cloud Run

### OpenClaw 配置

```json
{
  "channels": {
    "googleChat": {
      "enabled": true,
      "credentials": "/path/to/service-account.json",
      "project": "your-gcp-project-id",
      "spaces": {
        "allow": ["SPACE_ID"]
      },
      "commands": {
        "/ask": {
          "description": "Ask the AI a question",
          "params": {
            "question": {
              "type": "string",
              "required": true
            }
          }
        }
      }
    }
  }
}
```

### 功能

| 功能 | 支持情况 |
|-----|---------|
| 文本消息 | ✅ |
| 卡片 | ✅ |
| 对话框 | ✅ |
| 附件 | ✅ |
| 群组 | ✅ |
| 线程 | ✅ |

---

## Microsoft Teams

### 概述

Teams 是 Microsoft 365 的核心通信平台。

### 准备条件

1. **Microsoft 365 订阅** - 企业或教育版
2. **Azure AD** - 应用注册
3. **Bot Framework** - Bot Framework 资源
4. **App Studio** - Teams 应用创建

### 创建 Teams Bot

1. 在 Azure Portal 创建 Bot Channels Registration
2. 在 App Studio 创建 Teams 应用
3. 配置清单和功能
4. 发布到组织

### OpenClaw 配置

```json
{
  "channels": {
    "teams": {
      "enabled": true,
      "appId": "${TEAMS_APP_ID}",
      "appPassword": "${TEAMS_APP_PASSWORD}",
      "tenantId": "${TEAMS_TENANT_ID}",
      "teams": {
        "allow": ["*"]
      },
      "commands": {
        "ask": {
          "title": "Ask AI",
          "description": "Ask the AI a question",
          "initialRun": true
        }
      }
    }
  }
}
```

### 功能

| 功能 | 支持情况 |
|-----|---------|
| 个人聊天 | ✅ |
| 频道消息 | ✅ |
| 自适应卡片 | ✅ |
| 任务模块 | ✅ |
| 文件上传 | ✅ |

---

## Mattermost

### 概述

Mattermost 是开源的 Slack 替代品。

### 准备条件

1. **Mattermost 服务器** - 自托管或 Mattermost Cloud
2. **Bot Token** - 创建 Bot 账号
3. **WebSocket** - 启用 WebSocket

### 创建 Bot

```bash
# 使用 mmctl 创建 Bot
mmctl --url https://your-mattermost.com \
      --access-token YOUR_TOKEN \
      bot create \
      --name openclaw \
      --description "OpenClaw AI Bot"
```

### OpenClaw 配置

```json
{
  "channels": {
    "mattermost": {
      "enabled": true,
      "url": "https://your-mattermost.com",
      "botToken": "${MATTERMOST_BOT_TOKEN}",
      "teams": {
        "allow": ["*"]
      },
      "commands": {
        "/ask": {
          "description": "Ask the AI a question"
        }
      }
    }
  }
}
```

---

## IRC

### 概述

IRC（Internet Relay Chat）是最古老的聊天协议。

### 准备条件

1. **IRC 服务器** - Libera.Chat, OFTC 等
2. **IRC 客户端** - 用于注册（可选）
3. **NickServ** - 注册昵称

### OpenClaw 配置

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "server": "irc.libera.chat",
      "port": 6697,
      "ssl": true,
      "nickname": "OpenClawBot",
      "password": "${IRC_PASSWORD}",
      "channels": ["#openclaw", "#test"],
      "nickserv": {
        "enabled": true,
        "password": "${NICKSERV_PASSWORD}"
      }
    }
  }
}
```

### 特点

- ✅ 简单轻量
- ✅ 广泛支持
- ✅ 文本为主
- ❌ 功能有限

---

## LINE

### 概述

LINE 是亚洲流行的消息应用。

### 准备条件

1. **LINE Developers 账号** - 注册开发者
2. **Messaging API** - 启用 Messaging API
3. **Channel Access Token** - API 认证
4. **Webhook** - 配置 Webhook URL

### OpenClaw 配置

```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelSecret": "${LINE_CHANNEL_SECRET}",
      "channelAccessToken": "${LINE_CHANNEL_ACCESS_TOKEN}",
      "webhook": {
        "url": "https://your-domain.com/line/webhook"
      },
      "allowFrom": ["*"],
      "features": {
        "richMenu": true,
        "quickReply": true
      }
    }
  }
}
```

### 功能

| 功能 | 支持情况 |
|-----|---------|
| 文本消息 | ✅ |
| 图片 | ✅ |
| 视频 | ✅ |
| 音频 | ✅ |
| 位置 | ✅ |
| 贴纸 | ✅ |
| Rich Menu | ✅ |

---

## Matrix

### 概述

Matrix 是去中心化的通信协议。

### 准备条件

1. **Matrix 服务器** - 自托管或使用公共服务器
2. **Homeserver** - matrix.org, 代理等
3. **Access Token** - 获取访问令牌

### OpenClaw 配置

```json
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.org",
      "accessToken": "${MATRIX_ACCESS_TOKEN}",
      "userId": "@openclawbot:matrix.org",
      "deviceId": "DEVICE_ID",
      "rooms": {
        "allow": ["!roomId:matrix.org"]
      },
      "encryption": {
        "enabled": false
      }
    }
  }
}
```

### 特点

- ✅ 去中心化
- ✅ 联邦
- ✅ 端到端加密
- ✅ 桥接到其他协议
- ⚠️ 配置复杂

---

## 小结

本节我们学习了多种额外通道：

| 通道 | 难度 | 成本 | 推荐场景 |
|-----|------|------|---------|
| iMessage | ⭐⭐⭐ | 免费 | Apple 用户 |
| Slack | ⭐⭐ | 付费 | 企业团队 |
| Signal | ⭐ | 免费 | 隐私优先 |
| Google Chat | ⭐⭐ | 免费 | Google 用户 |
| Teams | ⭐⭐⭐ | 付费 | Microsoft 365 |
| Mattermost | ⭐⭐ | 免费 | 自托管 |
| IRC | ⭐ | 免费 | 开源社区 |
| LINE | ⭐⭐ | 免费 | 亚洲市场 |
| Matrix | ⭐⭐⭐ | 免费 | 去中心化 |

### 选择建议

1. **企业环境** - Slack, Microsoft Teams
2. **隐私优先** - Signal, Matrix
3. **开源/自托管** - Mattermost, Matrix, IRC
4. **地区特定** - LINE (亚洲)
5. **Apple 用户** - iMessage

## 下一步

[多通道管理](./11-multi-channel.md) - 统一管理多个通道
