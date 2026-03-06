# Discord 集成

> 学习目标：集成到 Discord 社区

## 目录

- [Discord 通道概述](#discord-通道概述)
- [准备条件](#准备条件)
- [创建 Discord 应用](#创建-discord-应用)
- [Bot 配置](#bot-配置)
- [Gateway 连接](#gateway-连接)
- [权限配置](#权限配置)
- [服务器管理](#服务器管理)
- [高级功能](#高级功能)
- [故障排查](#故障排查)
- [小结](#小结)

---

## Discord 通道概述

### 为什么选择 Discord？

- **社区友好** - 专为社区和服务器设计
- **丰富的 API** - 强大的 Bot API
- **实时通信** - 支持实时消息和事件
- **多媒体** - 支持丰富的媒体内容
- **角色系统** - 细粒度的权限控制

### Discord 通道特性

| 特性 | 支持状态 | 说明 |
|-----|---------|------|
| **文本消息** | ✅ | 完全支持 |
| **表情符号** | ✅ | 自定义表情支持 |
| **嵌入消息** | ✅ | 富文本嵌入 |
| **附件** | ✅ | 文件上传下载 |
| **语音** | ⚠️ | 有限支持 |
| **线程** | ✅ | 支持线程消息 |
| **Slash 命令** | ✅ | 支持 Discord 命令 |
| **按钮/菜单** | ✅ | 交互组件 |
| **模态框** | ✅ | 表单输入 |

---

## 准备条件

### 1. Discord 账号

- 确保你有有效的 Discord 账号
- 建议使用个人账号（不是服务器账号）

### 2. Discord 服务器

- 创建一个测试服务器
- 或使用现有服务器

### 3. 权限要求

你需要在服务器中拥有：
- **管理员权限** - 管理 Bot
- **服务器管理** - 邀请 Bot

---

## 创建 Discord 应用

### 步骤 1：访问 Discord 开发者门户

访问 https://discord.com/developers/applications

### 步骤 2：创建应用

1. 点击 **New Application**
2. 输入应用名称（如 "OpenClaw Bot"）
3. 点击 **Create**

### 步骤 3：创建 Bot

1. 在左侧菜单点击 **Bot**
2. 点击 **Add Bot**
3. 确认创建

### 步骤 4：配置 Bot

在 Bot 设置页面：

```
Name: OpenClaw Bot
Icon: [上传图标]
Description: Your AI Assistant
```

### 步骤 5：生成 Token

1. 点击 **Reset Token**
2. 复制生成的 Token
3. **安全保存** - 这个 Token 只显示一次！

```
YOUR_DISCORD_BOT_TOKEN_HERE  # 替换为你的实际 Token
```

### 步骤 6：获取应用信息

在 **General Information** 页面：

- **Application ID** - 应用程序 ID
- **Client Secret** - 客户端密钥

```
Application ID: 123456789012345678
Client Secret: AbCdEf1234567890
```

---

## Bot 配置

### 环境变量配置（推荐）

```bash
# Discord Bot Token
export DISCORD_BOT_TOKEN="YOUR_DISCORD_BOT_TOKEN_HERE"  # 替换为实际 Token

# Discord Client ID
export DISCORD_CLIENT_ID="YOUR_CLIENT_ID_HERE"  # 替换为实际 Client ID

# 添加到 shell 配置
echo 'export DISCORD_BOT_TOKEN="your-token"' >> ~/.bashrc
echo 'export DISCORD_CLIENT_ID="your-client-id"' >> ~/.bashrc
source ~/.bashrc
```

### 配置文件配置

编辑 `~/.openclaw/openclaw.json`:

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "botToken": "${DISCORD_BOT_TOKEN}",
      "clientId": "${DISCORD_CLIENT_ID}",
      "guilds": {
        "allow": ["123456789012345678"]
      }
    }
  }
}
```

### 完整配置示例

```json
{
  "channels": {
    "discord": {
      "enabled": true,

      // Bot 认证
      "botToken": "${DISCORD_BOT_TOKEN}",
      "clientId": "${DISCORD_CLIENT_ID}",

      // 服务器配置
      "guilds": {
        "allow": ["123456789012345678"],
        "deny": ["987654321098765432"]
      },

      // 命令配置
      "commands": {
        "prefix": "!",
        "registerCommands": true,
        "commands": {
          "ask": {
            "description": "Ask the AI a question",
            "usage": "!ask <your question>",
            "examples": [
              "!ask What is the weather today?",
              "!ask Help me debug this code"
            ]
          },
          "help": {
            "description": "Show help information",
            "usage": "!help [command]"
          }
        }
      },

      // 消息配置
      "message": {
        "respondToBots": false,
        "respondToDM": true,
        "mentionRequired": true
      },

      // 嵌入消息配置
      "embeds": {
        "enabled": true,
        "color": 5814783,  // 蓝色
        "timestamp": true
      },

      // 权限配置
      "permissions": {
        "adminRole": "Admin",
        "modRole": "Moderator",
        "userRole": "@everyone"
      },

      // 限制配置
      "rateLimit": {
        "enabled": true,
        "commands": 5,
        "period": 60
      }
    }
  }
}
```

---

## Gateway 连接

### 邀请 Bot 到服务器

#### 方式 1：使用邀请链接

1. 在 Discord Developer Portal，导航到 **OAuth2** -> **URL Generator**
2. 选择以下 Scopes：
   - `bot`
   - `applications.commands`
3. 选择 Bot Permissions：
   ```
   Send Messages
   Read Message History
   Add Reactions
   Use Slash Commands
   Embed Links
   Attach Files
   Read Messages/View Channels
   ```
4. 复制生成的 URL
5. 在浏览器中打开 URL 并邀请 Bot

#### 方式 2：使用 OpenClaw 生成链接

```bash
# 生成邀请链接
openclaw channel discord invite

# 输出:
# https://discord.com/api/oauth2/authorize?client_id=123456789012345678&permissions=8&scope=bot%20applications.commands
```

### 连接 Gateway

```bash
# 启动 Gateway
openclaw start

# 查看连接状态
openclaw channel discord status

# 输出示例:
# Discord: Connected
# Guilds: 2
# Channels: 15
# Users: 150
```

---

## 权限配置

### Bot 权限

在服务器设置中配置 Bot 权限：

| 权限 | 说明 | 必需 |
|-----|------|------|
| **Send Messages** | 发送消息 | ✅ 是 |
| **Read Messages** | 读取消息 | ✅ 是 |
| **Read Message History** | 读取历史消息 | ✅ 是 |
| **Add Reactions** | 添加表情反应 | ⚠️ 推荐 |
| **Use Slash Commands** | 使用斜杠命令 | ⚠️ 推荐 |
| **Embed Links** | 嵌入链接 | ⚠️ 推荐 |
| **Attach Files** | 附加文件 | ⚠️ 推荐 |
| **Connect** | 连接语音 | ❌ 否 |
| **Speak** | 语音说话 | ❌ 否 |
| **Administrator** | 管理员 | ❌ 不推荐 |

### 角色权限

```json
{
  "channels": {
    "discord": {
      "permissions": {
        "adminRole": "Admin",
        "modRole": "Moderator",
        "trustedRole": "Trusted",
        "userRole": "@everyone",
        "permissions": {
          "admin": ["*"],  // 所有权限
          "moderator": ["ask", "help", "status"],
          "trusted": ["ask", "help"],
          "user": ["ask"]
        }
      }
    }
  }
}
```

### 频道权限

限制 Bot 只在特定频道响应：

```json
{
  "channels": {
    "discord": {
      "allowedChannels": [
        "123456789012345678",  // AI 频道
        "123456789012345679"   // 帮助频道
      ],
      "ignoredChannels": [
        "987654321098765432"   // 不响应的频道
      ]
    }
  }
}
```

---

## 服务器管理

### 管理多个服务器

```json
{
  "channels": {
    "discord": {
      "guilds": {
        "allow": [
          "123456789012345678",  // 主服务器
          "123456789012345679"   // 测试服务器
        ]
      }
    }
  }
}
```

### 服务器特定配置

```json
{
  "channels": {
    "discord": {
      "guilds": {
        "configs": {
          "123456789012345678": {
            "prefix": "!",
            "mentionRequired": true,
            "allowedChannels": ["ai-channel"]
          },
          "123456789012345679": {
            "prefix": "?",
            "mentionRequired": false,
            "allowedChannels": ["*"]
          }
        }
      }
    }
  }
}
```

### 管理命令

```bash
# 查看已连接的服务器
openclaw channel discord guilds list

# 查看服务器详情
openclaw channel discord guilds info <guild-id>

# 添加服务器
openclaw channel discord guilds add <guild-id>

# 移除服务器
openclaw channel discord guilds remove <guild-id>
```

---

## 高级功能

### Slash 命令

```json
{
  "channels": {
    "discord": {
      "commands": {
        "registerSlashCommands": true,
        "slashCommands": {
          "ask": {
            "type": 1,  // CHAT_INPUT
            "description": "Ask the AI a question",
            "options": [
              {
                "name": "question",
                "description": "Your question",
                "type": 3,  // STRING
                "required": true
              }
            ]
          }
        }
      }
    }
  }
}
```

### 按钮和菜单

```markdown
# AGENTS.md

## 交互组件

### 按钮示例
当用户点击按钮时：
- **确认** - 继续执行操作
- **取消** - 取消操作
- **更多信息** - 提供更多细节

### 下拉菜单
提供选项让用户选择：
- 模型选择
- 语言偏好
- 输出格式
```

### 模态框

```json
{
  "channels": {
    "discord": {
      "modals": {
        "enabled": true,
        "forms": {
          "feedback": {
            "title": "提交反馈",
            "fields": [
              {
                "label": "你的反馈",
                "style": "paragraph",
                "required": true
              }
            ]
          }
        }
      }
    }
  }
}
```

---

## 故障排查

### 常见问题

#### Q1: Bot 无法连接

**检查清单**：

```bash
# 1. 验证 Bot Token
openclaw config get channels.discord.botToken

# 2. 检查 Gateway 状态
openclaw status

# 3. 查看连接日志
openclaw logs | grep "Discord"

# 4. 测试 Token
curl -H "Authorization: Bot <YOUR_TOKEN>" \
  https://discord.com/api/v10/users/@me
```

#### Q2: Bot 无响应

**解决方案**：

```bash
# 1. 检查 Bot 权限
# 确保 Bot 有 "Read Messages" 和 "Send Messages" 权限

# 2. 检查频道白名单
openclaw config get channels.discord.allowedChannels

# 3. 检查 mention 要求
openclaw config get channels.discord.message.mentionRequired

# 4. 查看 Bot 意图
openclaw channel discord intents
```

#### Q3: Slash 命令不工作

**解决方案**：

```bash
# 1. 确保启用了 applications.commands scope
# 重新生成邀请链接并添加 Bot

# 2. 重新注册命令
openclaw channel discord commands register --global

# 3. 检查命令权限
openclaw channel discord commands list

# 4. 同步命令
openclaw channel discord commands sync
```

#### Q4: 嵌入消息显示异常

**解决方案**：

```json
{
  "channels": {
    "discord": {
      "embeds": {
        "enabled": true,
        "color": 5814783,
        "timestamp": true,
        "footer": {
          "text": "OpenClaw AI Assistant",
          "iconUrl": "https://example.com/icon.png"
        },
        "author": {
          "name": "OpenClaw",
          "url": "https://openclaw.ai"
        }
      }
    }
  }
}
```

### 调试技巧

```bash
# 启用 Discord 详细日志
openclaw config set logging.channels.discord true
openclaw config set logging.level debug

# 查看实时日志
openclaw logs -f

# 查看 Discord 事件
openclaw channel discord events

# 测试 Bot 连接
openclaw channel discord test
```

---

## 小结

本节我们学习了：

1. ✅ Discord 通道概述 - 特性和支持情况
2. ✅ 准备条件 - 账号、服务器、权限
3. ✅ 创建 Discord 应用 - 通过开发者门户创建
4. ✅ Bot 配置 - Token 和完整配置
5. ✅ Gateway 连接 - 邀请和连接
6. ✅ 权限配置 - Bot 权限、角色权限、频道权限
7. ✅ 服务器管理 - 多服务器和服务器特定配置
8. ✅ 高级功能 - Slash 命令、按钮、模态框
9. ✅ 故障排查 - 常见问题和解决方案

## 测试你的配置

```bash
# 1. 在 Discord 中找到你的 Bot
# 使用 / 命令调用 Bot

# 2. 测试基本命令
# /ask Hello, how are you?

# 3. 测试前缀命令（如果配置）
# !ask What can you do?

# 4. 测试提及
# @OpenClaw Bot help me
```

## 下一步

[其他通道](./10-other-channels.md) - 学习 iMessage, Slack, Signal 等其他通道
