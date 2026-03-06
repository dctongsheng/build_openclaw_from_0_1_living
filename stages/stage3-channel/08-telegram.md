# Telegram 集成

> 学习目标：快速设置最简单的通道

## 目录

- [Telegram 通道概述](#telegram-通道概述)
- [准备条件](#准备条件)
- [创建 Bot](#创建-bot)
- [基础配置](#基础配置)
- [Webhook 配置](#webhook-配置)
- [群组支持](#群组支持)
- [命令设置](#命令设置)
- [高级功能](#高级功能)
- [故障排查](#故障排查)
- [小结](#小结)

---

## Telegram 通道概述

### 为什么选择 Telegram？

- **配置简单** - 最容易设置的通道
- **API 强大** - 功能丰富的 Bot API
- **无消息限制** - 没有消息数量限制
- **支持广泛** - 跨平台支持
- **隐私保护** - 端到端加密选项

### Telegram 通道特性

| 特性 | 支持状态 | 说明 |
|-----|---------|------|
| **文本消息** | ✅ | 完全支持 |
| **图片** | ✅ | 支持各种格式 |
| **视频** | ✅ | 支持视频文件 |
| **文档** | ✅ | 支持各种文件 |
| **语音** | ✅ | 支持语音消息 |
| **群组** | ✅ | 支持超级群组 |
| **频道** | ✅ | 支持频道 |
| **内联模式** | ✅ | 支持内联查询 |
| **自定义键盘** | ✅ | 支持自定义键盘 |
| **深链接** | ✅ | 支持深度链接 |

---

## 准备条件

### 1. Telegram 账号

- 确保你有有效的 Telegram 账号
- 安装 Telegram 应用（可选）

### 2. 与 BotFather 对话

Telegram 的所有 Bot 都通过 [@BotFather](https://t.me/botfather) 创建。

### 3. 选择用户标识

Telegram 使用以下标识：

- **用户名** - @username（公开）
- **用户 ID** - 数字 ID（私有）
- **群组 ID** - 负数 ID（群组/频道）

---

## 创建 Bot

### 步骤 1：启动 BotFather

在 Telegram 中搜索并启动 [@BotFather](https://t.me/botfather)。

### 步骤 2：创建新 Bot

发送 `/newbot` 命令：

```
You: /newbot

BotFather: Alright, a new bot. How are we going to call it? Please choose a name for your bot.

You: MyOpenClawBot

BotFather: Good. Now let's choose a username for your bot. It must end in `bot`. Like this, for example: TetrisBot or tetris_bot.

You: MyOpenClawBot_bot

BotFather: Done! Congratulations on your new bot. You will find it at t.me/MyOpenClawBot_bot. You can now add a description, about section and profile picture for your bot, see /help for a list of commands.

Use this token to access the HTTP API:
1234567890:ABCdefGHIjklMNOpqrsTUVwxyz

Keep your token secure and store it safely, it can be used by anyone to control your bot.
```

### 步骤 3：保存 API Token

复制并保存你的 API Token：

```
1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
```

### 步骤 4：获取你的用户信息

发送 `/start` 给 [@userinfobot](https://t.me/userinfobot) 获取你的用户 ID：

```
You: /start

UserInfoBot:
Id: 123456789
First: Your Name
Last: Your Last Name
Username: your_username
Lang: en
```

---

## 基础配置

### 环境变量配置（推荐）

```bash
# 设置 Telegram Bot Token
export TELEGRAM_BOT_TOKEN="1234567890:ABCdefGHIjklMNOpqrsTUVwxyz"

# 添加到 shell 配置
echo 'export TELEGRAM_BOT_TOKEN="1234567890:ABCdefGHIjklMNOpqrsTUVwxyz"' >> ~/.bashrc
source ~/.bashrc
```

### 配置文件配置

编辑 `~/.openclaw/openclaw.json`:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "${TELEGRAM_BOT_TOKEN}",
      "allowFrom": ["@your_username", 123456789]
    }
  }
}
```

### 完整配置示例

```json
{
  "channels": {
    "telegram": {
      "enabled": true,

      // Bot 认证
      "botToken": "${TELEGRAM_BOT_TOKEN}",

      // 白名单
      "allowFrom": [
        "@your_username",
        "@friend_username",
        123456789,
        987654321
      ],
      "denyFrom": ["@spam_user"],

      // 群组配置
      "groups": {
        "allow": [-1001234567890, -1001234567891],
        "deny": [-1009999999999]
      },

      // 消息配置
      "message": {
        "parseMode": "Markdown",
        "disableWebPagePreview": false,
        "disableNotification": false
      },

      // Webhook 配置
      "webhook": {
        "enabled": true,
        "url": "https://your-domain.com/telegram/webhook",
        "secret": "your-webhook-secret"
      },

      // 命令配置
      "commands": {
        "help": "显示帮助信息",
        "start": "开始使用",
        "ask": "向 AI 提问"
      }
    }
  }
}
```

---

## Webhook 配置

### 什么是 Webhook？

Webhook 允许 Telegram 将消息推送到你的服务器，而不是轮询获取。

### 使用 ngrok 测试（开发环境）

```bash
# 安装 ngrok
brew install ngrok  # macOS
# 或从 https://ngrok.com 下载

# 启动 ngrok 隧道
ngrok http 18789

# 你会看到:
# Forwarding https://abc123.ngrok.io -> http://localhost:18789
```

配置 webhook：

```bash
# 使用 OpenClaw 设置 webhook
openclaw channel telegram webhook set https://abc123.ngrok.io/telegram/webhook

# 验证 webhook
openclaw channel telegram webhook info
```

### 生产环境 Webhook

使用 Nginx 反向代理：

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location /telegram/webhook {
        proxy_pass http://localhost:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

配置：

```json
{
  "channels": {
    "telegram": {
      "webhook": {
        "enabled": true,
        "url": "https://your-domain.com/telegram/webhook",
        "secret": "your-random-secret-string",
        "maxConnections": 40
      }
    }
  }
}
```

### 手动设置 Webhook

```bash
# 使用 curl 设置 webhook
curl -X POST "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-domain.com/telegram/webhook",
    "secret_token": "your-secret-token"
  }'
```

---

## 群组支持

### 允许群组

```json
{
  "channels": {
    "telegram": {
      "groups": {
        "allow": [-1001234567890, -1001234567891]
      }
    }
  }
}
```

### 获取群组 ID

1. 将 Bot 添加到群组
2. 在群组中发送消息
3. 使用日志查看群组 ID：

```bash
openclaw logs | grep "chat_id"
```

或使用 BotFather：

```
You: /setgroup

BotFather: Choose a bot to change group messages settings.

You: @MyOpenClawBot_bot

BotFather: Select what you want to update:
  — Enable groups — Turn on groups for your bot
  — Disable groups — Turn off groups for your bot

You: Enable groups
```

### 群组权限

```json
{
  "channels": {
    "telegram": {
      "groups": {
        "allow": ["*"],  // 允许所有群组
        "adminOnly": false,  // 所有用户都可以使用
        "requireCommand": false  // 不需要命令
      }
    }
  }
}
```

---

## 命令设置

### 内置命令

```json
{
  "channels": {
    "telegram": {
      "commands": {
        "start": "开始使用 OpenClaw",
        "help": "显示帮助信息",
        "ask": "向 AI 提问 /ask <your question>",
        "status": "查看系统状态",
        "clear": "清除对话历史"
      }
    }
  }
}
```

### 自定义命令处理器

在 `workspace/` 中创建命令处理逻辑：

```markdown
# AGENTS.md

## 自定义命令

### /help 命令
当用户发送 /help 时：
1. 显示可用的命令列表
2. 简要说明每个命令的用途

### /ask 命令
当用户发送 /ask <question> 时：
1. 将 <question> 发送给 AI
2. 返回 AI 的回答

### /status 命令
当用户发送 /status 时：
1. 检查 Gateway 状态
2. 检查 API 连接
3. 返回状态摘要
```

### 设置命令菜单

```bash
# 使用 BotFather 设置命令菜单
# 发送 /setcommands

BotFather: Choose a bot to change the list of commands.

You: @MyOpenClawBot_bot

BotFather: OK. Send me a list of commands for your bot. Please use this format:

command1 - Description
command2 - Another description

You:
start - 开始使用 OpenClaw
help - 显示帮助信息
ask - 向 AI 提问
status - 查看系统状态

BotFather: Success! Command list updated. /help
```

---

## 高级功能

### 内联模式

允许用户在任何聊天中使用你的 Bot：

```json
{
  "channels": {
    "telegram": {
      "inline": {
        "enabled": true,
        "placeholder": "Ask AI...",
        "cacheTime": 300
      }
    }
  }
}
```

### 自定义键盘

```json
{
  "channels": {
    "telegram": {
      "keyboard": {
        "enabled": true,
        "keyboard": [
          ["Help", "Status"],
          ["Ask", "Clear"]
        ],
        "resizeKeyboard": true,
        "oneTimeKeyboard": false
      }
    }
  }
}
```

### 深链接

创建带有参数的链接：

```
https://t.me/MyOpenClawBot_bot?start=user_123
```

处理参数：

```markdown
# AGENTS.md

## 深链接处理

当用户通过深链接启动时：
1. 提取 start 参数
2. 根据参数提供个性化欢迎消息
3. 记录用户来源
```

---

## 故障排查

### 常见问题

#### Q1: Bot 无响应

**检查清单**：

```bash
# 1. 检查 Bot Token
openclaw config get channels.telegram.botToken

# 2. 验证 Token
curl "https://api.telegram.org/bot<YOUR_TOKEN>/getMe"

# 3. 检查 Gateway 状态
openclaw status

# 4. 查看日志
openclaw logs --channel telegram
```

#### Q2: Webhook 不工作

**解决方案**：

```bash
# 1. 检查 webhook 信息
curl "https://api.telegram.org/bot<YOUR_TOKEN>/getWebhookInfo"

# 2. 删除现有 webhook
curl "https://api.telegram.org/bot<YOUR_TOKEN>/deleteWebhook"

# 3. 重新设置 webhook
openclaw channel telegram webhook set <your-url>
```

#### Q3: 群组中 Bot 不响应

**检查清单**：

```bash
# 1. 确认 Bot 在群组中
# 在群组设置中检查成员列表

# 2. 检查群组权限
openclaw config get channels.telegram.groups

# 3. 确保 Bot 有权限发送消息
# 在群组设置中给予 Bot 管理员权限
```

#### Q4: 命令不工作

**解决方案**：

```bash
# 1. 检查命令是否在 BotFather 中设置
# 发送 /help 给 @BotFather

# 2. 重新设置命令
# 发送 /setcommands 给 @BotFather

# 3. 确保命令格式正确
# 命令必须以 / 开头，如 /help, /ask
```

### 调试技巧

```bash
# 启用详细日志
openclaw config set logging.level debug

# 实时查看日志
openclaw logs -f

# 过滤 Telegram 日志
openclaw logs | grep telegram

# 测试 Bot 连接
curl "https://api.telegram.org/bot<YOUR_TOKEN>/getMe"
```

---

## 小结

本节我们学习了：

1. ✅ Telegram 通道概述 - 特性和优势
2. ✅ 准备条件 - 账号、BotFather、用户标识
3. ✅ 创建 Bot - 通过 BotFather 创建和配置
4. ✅ 基础配置 - Token 和白名单设置
5. ✅ Webhook 配置 - 开发和生产环境
6. ✅ 群组支持 - 群组 ID 和权限
7. ✅ 命令设置 - 内置和自定义命令
8. ✅ 高级功能 - 内联模式、自定义键盘、深链接
9. ✅ 故障排查 - 常见问题和解决方案

## 测试你的配置

```bash
# 1. 启动与 Bot 的对话
# 在 Telegram 中搜索 @YourBotName 并点击 /start

# 2. 测试命令
# 发送 /help

# 3. 测试 AI 对话
# 发送: "Hello, can you help me?"

# 4. 测试群组
# 将 Bot 添加到群组并 @提及它
```

## 下一步

[Discord 集成](./09-discord.md) - 学习如何集成 Discord 通道
