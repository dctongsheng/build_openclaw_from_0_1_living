# 快速开始

> 学习目标：发送你的第一条 AI 消息

## 目录

- [初始化配置](#初始化配置)
- [配置 API Key](#配置-api-key)
- [启动 Gateway](#启动-gateway)
- [使用 Control UI](#使用-control-ui)
- [发送测试消息](#发送测试消息)
- [第一个对话](#第一个对话)
- [下一步](#下一步)
- [小结](#小结)

---

## 初始化配置

首次使用时，需要初始化 OpenClaw 配置。

### 运行入门向导

```bash
openclaw onboard
```

### 向导流程

向导会引导你完成以下步骤：

```
? 欢迎使用 OpenClaw！让我们开始配置吧。

? 选择你的使用方式:
  ❯ 个人使用
    团队协作
    开发测试

? 选择要配置的通道:
  ❯ WhatsApp
    Telegram
    Discord
    稍后配置

? 是否安装后台服务 (daemon)?
  ❯ 是 (推荐)
    否

✅ 配置完成！你的 OpenClaw 已准备就绪。
```

### 自动安装 Daemon

推荐安装 daemon，让 Gateway 在后台运行：

```bash
openclaw onboard --install-daemon
```

**Daemon 的优势**：
- 自动启动
- 崩溃自动重启
- 系统集成更好

---

## 配置 API Key

OpenClaw 需要访问 LLM Provider 的 API。

### 支持的 Provider

| Provider | 环境变量 | 获取地址 |
|---------|---------|---------|
| **OpenAI** | `OPENAI_API_KEY` | https://platform.openai.com/api-keys |
| **Anthropic** | `ANTHROPIC_API_KEY` | https://console.anthropic.com/ |
| **OpenRouter** | `OPENROUTER_API_KEY` | https://openrouter.ai/keys |
| **Google AI** | `GOOGLE_API_KEY` | https://makersuite.google.com/app/apikey |

### 配置方式 1：环境变量（推荐）

```bash
# macOS/Linux
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.bashrc
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc
source ~/.bashrc

# Windows PowerShell
[System.Environment]::SetEnvironmentVariable('OPENAI_API_KEY', 'sk-...', 'User')
```

### 配置方式 2：配置文件

编辑 `~/.openclaw/openclaw.json`:

```json
{
  "models": {
    "providers": {
      "openai": {
        "apiKey": "sk-...",
        "baseURL": "https://api.openai.com/v1"
      },
      "anthropic": {
        "apiKey": "sk-ant-..."
      }
    }
  }
}
```

### 配置方式 3：命令行

```bash
# 设置 OpenAI API Key
openclaw config set models.providers.openai.apiKey "sk-..."

# 设置 Anthropic API Key
openclaw config set models.providers.anthropic.apiKey "sk-ant-..."

# 查看配置
openclaw config get models
```

### 验证 API Key

```bash
# 测试 API 连接
openclaw doctor --test-api

# 输出示例:
# ✓ OpenAI API: Connected
# ✓ Anthropic API: Connected
```

---

## 启动 Gateway

### 方式 1：使用 Daemon（推荐）

如果安装了 daemon，Gateway 会自动启动：

```bash
# 检查状态
openclaw status

# 启动
openclaw start

# 停止
openclaw stop

# 重启
openclaw restart
```

### 方式 2：手动启动

```bash
# 前台运行（可以看到日志）
openclaw start --foreground

# 或直接运行
openclaw gateway
```

### 启动日志

正常启动后，你会看到类似输出：

```
[INFO] OpenClaw Gateway v1.x.x
[INFO] Loading configuration from /Users/username/.openclaw/openclaw.json
[INFO] Workspace: /Users/username/.openclaw/workspace
[INFO] Agent: Default agent loaded
[INFO] Channels: whatsapp, telegram
[INFO] Control UI: http://127.0.0.1:18789/
[INFO] Gateway ready and listening
```

---

## 使用 Control UI

OpenClaw 提供了一个 Web 界面，方便管理和交互。

### 访问 Control UI

```bash
# 打开 Control UI
openclaw dashboard

# 自动在浏览器打开 http://127.0.0.1:18789/
```

### Control UI 功能

#### 1. 概览页面 (Dashboard)

- Gateway 状态
- 活跃会话
- Token 使用统计
- 通道状态

#### 2. 会话管理 (Sessions)

- 查看所有会话
- 搜索和过滤
- 查看会话详情
- 删除会话

#### 3. 通道配置 (Channels)

- 查看已配置通道
- 通道连接状态
- 快速测试通道

#### 4. Agent 配置 (Agents)

- 查看已配置 Agent
- Agent 状态
- 工具配置

#### 5. 日志查看 (Logs)

- 实时日志
- 日志搜索
- 日志级别过滤

---

## 发送测试消息

### 方式 1：使用命令行

```bash
# 发送消息到指定通道
openclaw message send \
  --channel whatsapp \
  --target +8613800138000 \
  --message "Hello from OpenClaw!"

# 使用简短形式
openclaw msg send -c wa -t +8613800138000 -m "你好"
```

### 方式 2：使用 Control UI

1. 打开 `http://127.0.0.1:18789/`
2. 导航到 **Sessions** 页面
3. 点击 **New Session**
4. 选择通道和目标
5. 输入消息并发送

### 方式 3：直接从聊天应用发送

配置好通道后，直接在聊天应用中发送消息：

```
WhatsApp: +8613800138000
发送: "Hello OpenClaw, can you help me?"
```

---

## 第一个对话

让我们来体验一个完整的对话场景。

### 示例 1：简单问答

```
You: What is the capital of France?

OpenClaw: The capital of France is Paris. It's known for its
iconic landmarks like the Eiffel Tower, the Louvre Museum,
and its rich history as a center of art, culture, and fashion.
```

### 示例 2：代码辅助

```
You: Write a Python function to calculate fibonacci numbers

OpenClaw: Here's a Python function to calculate Fibonacci numbers:

```python
def fibonacci(n):
    """
    Calculate the nth Fibonacci number.

    Args:
        n (int): The position in the Fibonacci sequence

    Returns:
        int: The nth Fibonacci number
    """
    if n <= 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibonacci(n - 1) + fibonacci(n - 2)

# More efficient version using memoization
def fibonacci_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 0:
        return 0
    elif n == 1:
        return 1
    else:
        memo[n] = fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo)
        return memo[n]
```

The first version is simple but has exponential time complexity.
The second version uses memoization for linear time complexity.
```

### 示例 3：使用工具

OpenClaw Agent 可以使用工具来增强能力：

```
You: What's the weather like in Tokyo?

OpenClaw: I'll search for the current weather in Tokyo.

[Agent uses web_search tool]

OpenClaw: According to the search results, Tokyo currently has
mild temperatures around 18°C (64°F) with partly cloudy skies.
The humidity is around 65%. Perfect weather for sightseeing!
```

### 查看会话历史

```bash
# 列出所有会话
openclaw sessions list

# 查看特定会话
openclaw sessions history <session-id>

# 搜索会话
openclaw sessions list --search "python"
```

---

## 下一步

现在你已经成功发送了第一条消息，接下来可以：

### 立即尝试

1. **配置更多通道** - 连接 Telegram、Discord 等
2. **尝试不同的任务** - 代码、写作、研究
3. **探索工具** - 查看 Agent 可以使用哪些工具

### 深入学习

- [配置文件详解](./04-configuration.md) - 了解完整配置选项
- [模型配置](./05-model-configuration.md) - 配置多个模型
- [Agent Workspace](./06-agent-workspace.md) - 定制 Agent 行为

### 实践建议

- **每天使用** - 在日常任务中使用 OpenClaw
- **实验** - 尝试不同类型的请求
- **记录** - 记录有用的提示词和工作流
- **加入社区** - 与其他用户交流经验

---

## 常用命令速查

| 命令 | 说明 |
|-----|------|
| `openclaw doctor` | 健康检查 |
| `openclaw status` | 查看状态 |
| `openclaw start` | 启动 Gateway |
| `openclaw stop` | 停止 Gateway |
| `openclaw dashboard` | 打开 Control UI |
| `openclaw message send` | 发送消息 |
| `openclaw sessions list` | 列出会话 |
| `openclaw config get` | 查看配置 |
| `openclaw config set` | 设置配置 |

---

## 小结

恭喜！你已经完成了 OpenClaw 的快速入门。本节我们学习了：

1. ✅ 初始化配置 - 使用 `openclaw onboard`
2. ✅ 配置 API Key - 连接 LLM Provider
3. ✅ 启动 Gateway - 使用 daemon 或手动启动
4. ✅ 使用 Control UI - Web 管理界面
5. ✅ 发送测试消息 - 第一个 AI 对话

## 下一步

[配置文件详解](./04-configuration.md) - 深入了解 OpenClaw 的配置选项
