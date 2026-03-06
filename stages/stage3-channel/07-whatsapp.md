# WhatsApp 集成

> 学习目标：配置最受欢迎的通道

## 目录

- [WhatsApp 通道概述](#whatsapp-通道概述)
- [准备条件](#准备条件)
- [QR 配对流程](#qr-配对流程)
- [配置详解](#配置详解)
- [消息类型支持](#消息类型支持)
- [群组配置](#群组配置)
- [媒体处理](#媒体处理)
- [故障排查](#故障排查)
- [小结](#小结)

---

## WhatsApp 通道概述

### 为什么选择 WhatsApp？

- **用户基数大** - 全球超过 20 亿用户
- **高互动率** - 用户活跃度高
- **多媒体支持** - 支持文字、图片、视频、文件
- **群组功能** - 支持群组对话
- **端到端加密** - 通信安全

### WhatsApp 通道特性

| 特性 | 支持状态 | 说明 |
|-----|---------|------|
| **文本消息** | ✅ | 完全支持 |
| **图片** | ✅ | 自动下载和处理 |
| **视频** | ✅ | 自动下载和处理 |
| **文档** | ✅ | 支持各种文件格式 |
| **语音** | ✅ | 转录为文本 |
| **群组** | ✅ | 支持 @ 提及触发 |
| **广播** | ✅ | 支持广播列表 |
| **状态** | ❌ | 不支持 |

---

## 准备条件

### 1. WhatsApp 账号

- 确保你有有效的 WhatsApp 账号
- 建议使用单独的号码（不是个人号码）

### 2. 手机号格式

WhatsApp 使用国际电话号码格式：

```
+[国家码][手机号]

示例:
+8613800138000  (中国)
+15555555555    (美国)
+447700900123   (英国)
```

### 3. 网络连接

- 稳定的互联网连接
- WhatsApp Web 可访问

---

## QR 配对流程

### 步骤 1：启用 WhatsApp 通道

编辑 `~/.openclaw/openclaw.json`:

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+8613800138000"]
    }
  }
}
```

### 步骤 2：启动 Gateway

```bash
# 启动 Gateway
openclaw start

# 如果已经在运行，重启
openclaw restart
```

### 步骤 3：获取 QR 码

```bash
# 查看日志获取 QR 码
openclaw logs | grep QR

# 或在 Control UI 中查看
openclaw dashboard
# 导航到 Channels -> WhatsApp -> Pair
```

### 步骤 4：扫描 QR 码

1. 打开手机上的 WhatsApp
2. 点击 **设置** -> **关联设备**
3. 点击 **关联设备**
4. 扫描显示的 QR 码

### 步骤 5：确认配对

配对成功后，你会看到：

```
[INFO] WhatsApp: Connected successfully
[INFO] WhatsApp: Phone +8613800138000 connected
```

---

## 配置详解

### 基础配置

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+8613800138000"],
      "groupMentionTrigger": "@ai"
    }
  }
}
```

### 完整配置

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,

      // 白名单配置
      "allowFrom": [
        "+8613800138000",
        "+8613800138001"
      ],
      "denyFrom": [
        "+8613800139999"
      ],

      // 群组配置
      "groupMentionTrigger": "@ai",
      "groups": {
        "allow": ["Team Group", "Project Updates"],
        "deny": ["Family Group"]
      },

      // 商业配置（可选）
      "businessProfile": {
        "name": "AI Assistant",
        "photo": "/path/to/photo.jpg",
        "description": "Your AI Assistant",
        "email": "support@example.com",
        "website": "https://example.com"
      },

      // 消息配置
      "message": {
        "autoDownload": {
          "images": true,
          "videos": false,
          "documents": true,
          "maxSize": 10485760  // 10MB
        },
        "retryAttempts": 3,
        "retryDelay": 1000
      },

      // Webhook 配置（可选）
      "webhook": {
        "url": "https://your-server.com/webhook",
        "secret": "your-webhook-secret"
      }
    }
  }
}
```

---

## 消息类型支持

### 文本消息

完全支持，包括：

- 纯文本
- 格式化文本（粗体、斜体、删除线）
- 链接
- 表情符号

```
You: Hello @ai, can you help me?

OpenClaw: Of course! What can I help you with?
```

### 图片消息

自动下载和处理：

```json
{
  "channels": {
    "whatsapp": {
      "message": {
        "autoDownload": {
          "images": true,
          "maxSize": 5242880  // 5MB
        }
      }
    }
  }
}
```

### 视频消息

支持视频下载（注意文件大小）：

```json
{
  "channels": {
    "whatsapp": {
      "message": {
        "autoDownload": {
          "videos": true,
          "maxSize": 10485760  // 10MB
        }
      }
    }
  }
}
```

### 文档消息

支持各种文档格式：

- PDF (.pdf)
- Word (.doc, .docx)
- Excel (.xls, .xlsx)
- PowerPoint (.ppt, .pptx)
- 文本文件 (.txt, .md)
- 代码文件 (.py, .js, 等)

```json
{
  "channels": {
    "whatsapp": {
      "message": {
        "autoDownload": {
          "documents": true,
          "maxSize": 10485760  // 10MB
        }
      }
    }
  }
}
```

### 语音消息

语音消息会被转录为文本：

```
[Voice message transcribed]
"请帮我分析这个数据"
```

---

## 群组配置

### @ 提及触发

在群组中，只有被 @ 提及时才响应：

```json
{
  "channels": {
    "whatsapp": {
      "groupMentionTrigger": "@ai"
    }
  }
}
```

使用示例：

```
In group "Team Group":

Alice: Anyone know how to fix this bug?

Bob: @ai can you help with the bug?

OpenClaw: I'd be happy to help! Please share the error message and code.
```

### 群组白名单

只响应特定群组：

```json
{
  "channels": {
    "whatsapp": {
      "groups": {
        "allow": ["Team Group", "Dev Team"],
        "deny": ["Marketing", "Random Chat"]
      }
    }
  }
}
```

### 群组管理

```bash
# 查看已加入的群组
openclaw channel whatsapp groups list

# 添加群组到白名单
openclaw channel whatsapp groups add "Team Group"

# 移除群组
openclaw channel whatsapp groups remove "Random Chat"
```

---

## 媒体处理

### 自动下载配置

```json
{
  "channels": {
    "whatsapp": {
      "message": {
        "autoDownload": {
          "enabled": true,
          "images": true,
          "videos": false,
          "documents": true,
          "audio": false,
          "maxSize": 10485760,
          "allowedTypes": ["jpg", "png", "pdf", "doc", "docx"]
        }
      }
    }
  }
}
```

### 媒体存储位置

```
~/.openclaw/channels/whatsapp/media/
├── images/
├── videos/
├── documents/
└── audio/
```

### 媒体文件清理

```bash
# 查看媒体文件大小
openclaw channel whatsapp media stats

# 清理旧媒体文件
openclaw channel whatsapp media clean --older-than 30d

# 清理所有媒体
openclaw channel whatsapp media clean --all
```

---

## 故障排查

### 常见问题

#### Q1: QR 码无法加载

**解决方案**：

```bash
# 检查 WhatsApp 通道状态
openclaw channel whatsapp status

# 重启通道
openclaw channel whatsapp restart

# 查看详细日志
openclaw logs --channel whatsapp
```

#### Q2: 配对后立即断开

**可能原因**：
- 网络不稳定
- WhatsApp 账号被限制
- 多设备同时登录

**解决方案**：

```bash
# 检查网络连接
ping -c 4 web.whatsapp.com

# 重新配对
openclaw channel whatsapp unpair
openclaw channel whatsapp pair
```

#### Q3: 消息发送失败

**检查清单**：

```bash
# 1. 检查白名单
openclaw config get channels.whatsapp.allowFrom

# 2. 检查号码格式（必须是国际格式）
openclaw channel whatsapp validate +8613800138000

# 3. 检查 Gateway 状态
openclaw status

# 4. 查看错误日志
openclaw logs --level error
```

#### Q4: 群组中无响应

**检查清单**：

```bash
# 1. 检查群组触发配置
openclaw config get channels.whatsapp.groupMentionTrigger

# 2. 检查群组白名单
openclaw channel whatsapp groups list

# 3. 确保使用正确的触发词
# 例如: "@ai help" 而不是 "@aihelp"
```

### 调试模式

启用调试日志：

```json
{
  "logging": {
    "level": "debug",
    "channels": {
      "whatsapp": true
    }
  }
}
```

---

## 小结

本节我们学习了：

1. ✅ WhatsApp 通道概述 - 特性和支持情况
2. ✅ 准备条件 - 账号、号码格式、网络
3. ✅ QR 配对流程 - 5 步完成配对
4. ✅ 配置详解 - 基础和完整配置
5. ✅ 消息类型支持 - 文本、图片、视频、文档、语音
6. ✅ 群组配置 - @ 提及触发和群组白名单
7. ✅ 媒体处理 - 自动下载和存储管理
8. ✅ 故障排查 - 常见问题和解决方案

## 测试你的配置

```bash
# 1. 发送测试消息
openclaw message send --channel whatsapp --target +8613800138000 --message "Hello from OpenClaw!"

# 2. 测试群组响应
# 在已配置的群组中发送: "@ai hello"

# 3. 测试图片处理
# 发送一张图片到 WhatsApp

# 4. 测试文档处理
# 发送一个文档到 WhatsApp
```

## 下一步

[Telegram 集成](./08-telegram.md) - 学习如何集成 Telegram 通道
