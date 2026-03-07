# 故障排查指南

> 学习目标：掌握 OpenClaw 常见问题的诊断和解决方法

## 目录

- [诊断工具](#诊断工具)
- [工具权限问题](#工具权限问题)
- [连接问题](#连接问题)
- [模型配置问题](#模型配置问题)
- [通道配置问题](#通道配置问题)
- [性能问题](#性能问题)
- [安全审计警告](#安全审计警告)
- [日志分析](#日志分析)
- [常见错误代码](#常见错误代码)
- [获取帮助](#获取帮助)

---

## 诊断工具

### openclaw doctor

`openclaw doctor` 是你的首要诊断工具，它会全面检查系统状态：

```bash
# 基础诊断
openclaw doctor

# 自动修复问题
openclaw doctor --fix

# 深度诊断
openclaw doctor --deep
```

输出内容包括：
- Gateway 状态
- 认证配置
- 状态完整性
- 安全问题
- 技能状态
- 插件状态
- 内存搜索状态

### openclaw status

查看实时运行状态：

```bash
# 基础状态
openclaw status

# 详细状态
openclaw status --deep

# 包含所有信息
openclaw status --all
```

### openclaw logs

查看实时日志：

```bash
# 跟踪日志
openclaw logs --follow

# 查看特定级别
openclaw logs --level error

# 查看最近日志
tail -f ~/.openclaw/logs/gateway.log
```

---

## 工具权限问题

### 🔴 问题：Agent 无法使用工具

**症状**：
- Agent 返回 "I don't have access to that tool"
- Agent 拒绝执行文件操作、命令执行等
- 工具调用失败或被拒绝

**原因分析**：

最常见的原因是 `tools.profile` 配置不正确。某些配置向导或模板会默认使用 `"messaging"` profile，这会限制大部分工具的访问权限。

**诊断步骤**：

```bash
# 1. 检查当前工具配置
openclaw config get tools.profile

# 2. 查看完整的工具配置
openclaw config get tools

# 3. 检查 agent 的工具配置
openclaw config get agents.defaults.tools
```

**解决方案**：

#### 方案一：修改为完整工具配置（推荐）

```bash
# 设置为 full 模式，启用所有工具
openclaw config set tools.profile full

# 重启 Gateway 以应用更改
openclaw gateway restart
```

#### 方案二：使用特定的工具配置文件

```bash
# 编程相关工具
openclaw config set tools.profile coding

# 或者创建自定义配置
openclaw config set tools.profiles.custom.allow '["read","write","exec","web_search"]'
openclaw config set tools.profile custom
```

#### 方案三：为特定 Agent 配置工具

```bash
# 为特定 agent 设置工具配置
openclaw agents edit <agent-id> --tools-profile full
```

**验证修复**：

```bash
# 1. 检查配置是否生效
openclaw config get tools.profile

# 2. 查看 agent 状态
openclaw agents list

# 3. 测试工具访问
# 在对话中尝试使用工具，看是否能正常工作
```

**不同工具配置文件对比**：

| 配置文件 | 允许的工具 | 适用场景 |
|---------|-----------|----------|
| `messaging` | read, write, message, sessions | 纯消息机器人，不需要执行能力 |
| `coding` | read, write, edit, exec, web_search, web_fetch | 编程助手，需要代码操作能力 |
| `full` | 所有工具 (*) | 完整功能，需要所有能力 |
| `minimal` | read, write | 最基础的文件操作 |

**预防措施**：

1. **明确需求**：配置前明确你的 Agent 需要哪些能力
2. **安全考虑**：只授予必要的工具权限
3. **定期审计**：使用 `openclaw security audit` 检查配置
4. **测试验证**：配置后测试工具是否正常工作

---

### 🔴 问题：exec/curl 等命令执行工具无法使用

**症状**：
- Agent 返回 "I don't have access to exec tool"
- 无法执行 shell 命令（curl, ls, grep 等）
- 命令执行功能被拒绝

**原因分析**：

即使 `tools.profile` 设置为 "full"，OpenClaw 默认不会启用"提升工具"（elevated tools）。这些工具包括：
- `exec` - 执行 shell 命令
- `process` - 进程管理
- `runtime` - 运行时控制

这是为了安全考虑，因为这些工具具有系统级访问权限。

**诊断步骤**：

```bash
# 1. 检查当前工具配置
openclaw config get tools.profile
# 应该显示 "full"

# 2. 检查 elevated tools 是否启用
openclaw config get tools.elevated
# 如果显示 "Config path not found"，说明未启用

# 3. 查看完整的安全审计
openclaw status
# 查看 Security audit 部分，确认 runtime 工具状态
```

**解决方案**：

#### 方案一：启用 Elevated Tools（推荐）

```bash
# 1. 编辑配置文件
nano ~/.openclaw/openclaw.json

# 2. 在 tools 部分添加 elevated 配置
{
  "tools": {
    "profile": "full",
    "elevated": {
      "enabled": true
    }
  }
}

# 3. 保存并重启 Gateway
openclaw gateway restart

# 4. 验证修复
openclaw status | grep runtime
# 应该看到: runtime=[exec, process]
```

#### 方案二：使用命令行配置

```bash
# 创建 elevated 配置
cat > /tmp/elevated-config.json << 'EOF'
{
  "tools": {
    "profile": "full",
    "elevated": {
      "enabled": true
    }
  }
}
EOF

# 合并到主配置
jq -s '.[0] * .[1]' ~/.openclaw/openclaw.json /tmp/elevated-config.json > ~/.openclaw/openclaw.json.tmp
mv ~/.openclaw/openclaw.json.tmp ~/.openclaw/openclaw.json

# 重启 Gateway
openclaw gateway restart
```

**验证修复**：

```bash
# 1. 检查 elevated 状态
openclaw config get tools.elevated.enabled
# 应该显示: true

# 2. 查看安全审计报告
openclaw status | grep -A 5 "runtime"
# 应该看到: runtime=[exec, process]

# 3. 测试命令执行
# 在 OpenClaw 对话中尝试执行简单命令
```

**⚠️ 安全警告**：

启用 elevated tools 后，安全审计会显示警告：

```
CRITICAL Open groupPolicy with runtime/filesystem tools exposed
```

**安全建议**：

1. **限制群组访问**：
   ```bash
   # 将飞书群组改为 allowlist
   openclaw config set channels.feishu.groupPolicy "allowlist"
   openclaw config set channels.feishu.groupAllowFrom '["trusted_group_id"]'
   ```

2. **启用沙箱**（如果可能）：
   ```bash
   openclaw config set agents.defaults.sandbox.mode "all"
   ```

3. **限制文件系统访问**：
   ```bash
   openclaw config set tools.fs.workspaceOnly true
   ```

4. **使用白名单**：
   - 只在受信任的私聊中启用这些工具
   - 避免在开放群组中使用 elevated tools

**不同配置的安全级别对比**：

| 配置 | 工具权限 | 安全级别 | 适用场景 |
|------|---------|---------|----------|
| `messaging` + no elevated | 仅消息工具 | ⭐⭐⭐⭐⭐ | 纯聊天机器人 |
| `full` + no elevated | 基础工具（无命令执行） | ⭐⭐⭐⭐ | 一般助手 |
| `full` + elevated + private | 所有工具（私聊） | ⭐⭐⭐ | 个人助手 |
| `full` + elevated + open groups | 所有工具（开放群组） | ⭐ | 极高风险，不推荐 |

**最佳实践**：

1. ✅ **默认使用最小权限**：从 `messaging` 开始，按需提升
2. ✅ **私聊启用 elevated**：只在个人聊天中启用命令执行
3. ✅ **群组使用 allowlist**：严格控制哪些群组可以使用高级工具
4. ✅ **定期安全审计**：每周运行 `openclaw security audit`
5. ✅ **监控日志**：关注异常的命令执行行为
6. ❌ **避免开放群组 + elevated**：这是最危险的组合

---

## 连接问题

### Gateway 无法连接

**症状**：
- `openclaw status` 显示 Gateway unreachable
- WebSocket 连接失败
- 控制面板无法打开

**诊断**：

```bash
# 检查 Gateway 状态
openclaw gateway status

# 检查端口占用
lsof -i :18789

# 检查防火墙
# macOS
sudo pfctl -s rules | grep 18789
# Linux
sudo ufw status
```

**解决方案**：

```bash
# 1. 确保 Gateway 正在运行
openclaw gateway start

# 2. 如果已运行但无法连接，重启
openclaw gateway restart

# 3. 检查配置
openclaw config get gateway

# 4. 查看错误日志
tail -50 ~/.openclaw/logs/gateway.log
```

### 通道连接失败

**症状**：
- 通道显示 disconnected
- 消息发送失败
- Webhook 不工作

**诊断**：

```bash
# 检查所有通道状态
openclaw status --deep

# 检查特定通道
openclaw channels status telegram
openclaw channels status feishu
```

**常见原因**：

1. **认证信息错误**
   ```bash
   # 检查认证配置
   openclaw config get channels.telegram.botToken
   openclaw config get channels.feishu.appId
   ```

2. **网络问题**
   ```bash
   # 测试网络连接
   curl https://api.telegram.org
   ping open.feishu.cn
   ```

3. **Webhook 配置错误**
   ```bash
   # 检查 webhook 配置
   openclaw config get channels.telegram.webhook
   ```

---

### 🔴 Telegram 群组无响应

**症状**：
- ✅ Telegram 私聊工作正常
- ❌ Telegram 群组中 @bot 无任何反应
- ❌ 群组消息被完全忽略

**原因分析**：

这是**第三常见的配置问题**！

Telegram 有两种独立的访问控制策略：
1. **私聊策略（dmPolicy）**：控制私聊访问
2. **群组策略（groupPolicy）**：控制群组访问

当群组策略设置为 `allowlist`（白名单模式）但白名单列表为空时，所有群组消息会被**静默丢弃**。

**诊断步骤**：

```bash
# 1. 检查群组策略
openclaw config get channels.telegram.groupPolicy
# 如果显示 "allowlist"，说明是白名单模式

# 2. 检查白名单列表
openclaw config get channels.telegram.groupAllowFrom
# 如果显示 "Config path not found"，说明白名单为空

# 3. 查看安全审计
openclaw status | grep -A 5 "telegram"
# 应该能看到警告：groupAllowFrom is empty
```

**解决方案**：

#### 方案一：开放所有群组（简单但不推荐）

```bash
# 修改群组策略为 open
openclaw config set channels.telegram.groupPolicy open

# 重启 Gateway
openclaw gateway restart

# 验证配置
openclaw config get channels.telegram.groupPolicy
# 应该显示: open
```

**优点**：
- ✅ 配置简单，一条命令搞定
- ✅ 任何群组都可以使用

**缺点**：
- ⚠️ 安全风险较高
- ⚠️ 任何群组都可以访问 bot
- ⚠️ 如果启用了 elevated tools，极度危险

#### 方案二：使用白名单（推荐，更安全）

```bash
# 步骤 1：获取群组 ID
# 方法 A：从日志中查找
grep "telegram" ~/.openclaw/logs/gateway.log | grep "group" | tail -10

# 方法 B：使用 Telegram API
# 访问：https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates

# 步骤 2：添加群组到白名单（假设群组 ID 是 -1001234567890）
openclaw config set channels.telegram.groupAllowFrom '["-1001234567890"]'

# 添加多个群组
openclaw config set channels.telegram.groupAllowFrom '["-1001234567890", "-1009876543210"]'

# 步骤 3：（可选）设置提及触发
openclaw config set channels.telegram.groupMentionTrigger "@ai"

# 步骤 4：重启 Gateway
openclaw gateway restart

# 验证配置
openclaw config get channels.telegram.groupAllowFrom
```

**优点**：
- ✅ 安全可控
- ✅ 只允许指定的群组访问
- ✅ 适合与 elevated tools 一起使用

**缺点**：
- 需要先获取群组 ID
- 需要手动添加每个群组

**验证修复**：

```bash
# 1. 检查配置
openclaw config get channels.telegram.groupPolicy

# 2. 检查白名单
openclaw config get channels.telegram.groupAllowFrom

# 3. 查看日志
tail -f ~/.openclaw/logs/gateway.log

# 4. 在 Telegram 群组中测试
# @your_bot_name 你好
```

**⚠️ 安全警告**：

如果启用了 elevated tools，请务必：

1. ✅ **使用白名单模式**
2. ✅ **只添加受信任的群组**
3. ✅ **设置提及触发**（groupMentionTrigger）
4. ✅ **定期审查白名单**
5. ❌ **避免在公开群组中使用**

**不同配置的安全级别对比**：

| 私聊 | 群组 | Elevated | 安全级别 | 适用场景 |
|------|------|----------|----------|----------|
| pairing | allowlist（空） | ❌ | ⭐⭐⭐⭐⭐ | 仅私聊 |
| pairing | open | ❌ | ⭐⭐⭐ | 私聊+公开群组 |
| pairing | allowlist | ✅ | ⭐⭐⭐⭐ | 私聊+指定群组 |
| pairing | open | ✅ | ⭐ | ❌ 极度危险 |

**最佳实践**：

1. ✅ **默认使用白名单**：从 allowlist 开始
2. ✅ **按需添加群组**：只添加受信任的群组
3. ✅ **设置提及触发**：避免无意中触发
4. ✅ **定期安全审计**：每周检查白名单
5. ✅ **监控日志**：关注异常的群组活动

---

### 🔴 Telegram 群组配置了白名单但仍无响应

**症状**：
- ✅ Telegram 私聊工作正常
- ✅ 已配置 `groupPolicy: "allowlist"`
- ✅ 已配置 `groupAllowFrom: [...]`（包含正确的群组ID）
- ❌ Telegram 群组中 @bot 仍然无任何反应

**原因分析**：

这是一个**更隐蔽的配置问题**！

虽然你配置了全局的 `groupPolicy` 和 `groupAllowFrom`，但**缺少了 `channels.telegram.groups` 对象的配置**。

根据 OpenClaw 的配置体系，群组访问控制有三个层次：

1. **全局群组策略**（`channels.telegram.groupPolicy`）
   - 控制默认的群组访问行为
   - 可选值：`open`、`allowlist`、`deny`

2. **全局白名单**（`channels.telegram.groupAllowFrom`）
   - 当 `groupPolicy` 为 `allowlist` 时生效
   - 列出允许的群组 ID

3. **⭐ 每个群组的详细配置**（`channels.telegram.groups.<群组ID>`）**【容易被忽略】**
   - 为每个群组单独配置行为
   - 包括：`requireMention`、`groupPolicy`、`enabled` 等
   - **即使设置了全局白名单，也必须配置此项！**

**官方文档说明**：

> Group replies require mention by default. Mention can come from native @botusername mention, or mention patterns in agents.list[].groupChat.mentionPatterns.

如果不配置 `channels.telegram.groups`，系统无法确定：
- 是否需要 @bot 才能触发
- 该群组的策略是什么
- 该群组是否启用

**诊断步骤**：

```bash
# 1. 检查全局群组策略
openclaw config get channels.telegram.groupPolicy
# 应该显示: allowlist

# 2. 检查全局白名单
openclaw config get channels.telegram.groupAllowFrom
# 应该显示: ["-5002529238", "-5092528016", ...]

# 3. 检查群组详细配置（关键！）
openclaw config get channels.telegram.groups
# 如果显示 "Config path not found" 或 "{}"，说明没有配置！

# 4. 查看会话文件获取群组 ID
cat ~/.openclaw/agents/main/sessions/sessions.json | grep "telegram:group" | head -5
# 应该看到类似：agent:main:telegram:group:-5002529238
```

**解决方案**：

为每个群组 ID 添加详细配置：

```bash
# 为群组 -5002529238 配置
openclaw config set channels.telegram.groups.'-5002529238'.groupPolicy open
openclaw config set channels.telegram.groups.'-5002529238'.requireMention false

# 为群组 -5092528016 配置
openclaw config set channels.telegram.groups.'-5092528016'.groupPolicy open
openclaw config set channels.telegram.groups.'-5092528016'.requireMention false

# 为群组 -5082337987 配置
openclaw config set channels.telegram.groups.'-5082337987'.groupPolicy open
openclaw config set channels.telegram.groups.'-5082337987'.requireMention false

# 为群组 -5146706488 配置
openclaw config set channels.telegram.groups.'-5146706488'.groupPolicy open
openclaw config set channels.telegram.groups.'-5146706488'.requireMention false

# 重启 Gateway
openclaw gateway restart

# 验证配置
openclaw config get channels.telegram.groups
```

**配置说明**：

每个群组的配置选项：

```json
{
  "channels": {
    "telegram": {
      "groups": {
        "-5002529238": {
          "groupPolicy": "open",           // 该群组的策略（可覆盖全局设置）
          "requireMention": false,         // 是否需要 @bot 才能触发
          "enabled": true,                 // 是否启用该群组（默认 true）
          "groupMentionTrigger": "@ai"     // 自定义提及触发字符串（可选）
        }
      }
    }
  }
}
```

**参数说明**：

- `groupPolicy`: 该群组的访问策略
  - `"open"` - 开放，所有成员都可以与 bot 交互
  - `"allowlist"` - 白名单模式（需要配合 `allowFrom`）
  - `"deny"` - 禁用该群组

- `requireMention`: 是否需要 @bot 才能触发
  - `false` - 任何消息都会被处理（适合专用 bot 群组）
  - `true` - 只有 @bot 的消息才会被处理（适合有其他对话的群组）

- `enabled`: 是否启用该群组
  - `true` - 启用
  - `false` - 禁用（bot 不会处理该群组的任何消息）

**验证修复**：

```bash
# 1. 检查 groups 配置
openclaw config get channels.telegram.groups
# 应该看到所有群组的配置

# 2. 检查 Gateway 状态
openclaw status | grep -A 10 "Sessions"
# 应该看到 group 类型的会话

# 3. 查看日志
tail -f ~/.openclaw/logs/gateway.log

# 4. 在 Telegram 群组中测试
# 发送消息（不需要 @bot，如果 requireMention: false）
# 你好

# 或发送（如果 requireMention: true）
# @your_bot_name 你好
```

**配置完整示例**：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "botToken": "YOUR_BOT_TOKEN",
      "groupPolicy": "allowlist",
      "groupAllowFrom": [
        "-5002529238",
        "-5092528016",
        "-5082337987",
        "-5146706488"
      ],
      "groups": {
        "-5002529238": {
          "groupPolicy": "open",
          "requireMention": false
        },
        "-5092528016": {
          "groupPolicy": "open",
          "requireMention": true,
          "groupMentionTrigger": "@ai"
        },
        "-5082337987": {
          "groupPolicy": "open",
          "requireMention": false
        },
        "-5146706488": {
          "groupPolicy": "open",
          "requireMention": false
        }
      }
    }
  }
}
```

**常见配置场景**：

#### 场景 1：专用 AI 群组（推荐）

```bash
# 不需要 @bot，任何消息都由 AI 处理
openclaw config set channels.telegram.groups.'-5002529238'.requireMention false
openclaw config set channels.telegram.groups.'-5002529238'.groupPolicy open
```

**适用**：专门为 AI bot 创建的群组，所有消息都是给 bot 的。

#### 场景 2：混合群组（需要 @bot）

```bash
# 只有 @bot 才触发，其他正常聊天不受影响
openclaw config set channels.telegram.groups.'-5092528016'.requireMention true
openclaw config set channels.telegram.groups.'-5092528016'.groupPolicy open
```

**适用**：普通聊天群组，偶尔需要 AI 协助。

#### 场景 3：禁用特定群组

```bash
# 临时禁用某个群组
openclaw config set channels.telegram.groups.'-5082337987'.enabled false
```

**适用**：不想让 bot 处理某个群组的消息。

**⚠️ 重要提示**：

1. **必须配置 `groups` 对象**：即使设置了全局白名单，也必须为每个群组配置 `groups` 对象！

2. **群组 ID 格式**：Telegram 群组 ID 通常是负数，如 `-5002529238` 或 `-1001234567890`

3. **配置优先级**：群组级别的配置（`groups.<群组ID>`）优先于全局配置（`groupPolicy`）

4. **安全性**：如果启用了 elevated tools，建议：
   - 使用 `requireMention: true`
   - 设置自定义的 `groupMentionTrigger`
   - 只在受信任的群组中使用

**排查清单**：

- [ ] 检查 `channels.telegram.groupPolicy` 是否为 `allowlist`
- [ ] 检查 `channels.telegram.groupAllowFrom` 是否包含目标群组 ID
- [ ] **检查 `channels.telegram.groups` 是否配置**（最容易被忽略！）
- [ ] 为每个群组 ID 配置 `groupPolicy` 和 `requireMention`
- [ ] 重启 Gateway：`openclaw gateway restart`
- [ ] 在群组中测试功能

---

**诊断**：

```bash
# 检查所有通道状态
openclaw status --deep

# 检查特定通道
openclaw channels status telegram
openclaw channels status feishu
```

**常见原因**：

1. **认证信息错误**
   ```bash
   # 检查认证配置
   openclaw config get channels.telegram.botToken
   openclaw config get channels.feishu.appId
   ```

2. **网络问题**
   ```bash
   # 测试网络连接
   curl https://api.telegram.org
   ping open.feishu.cn
   ```

3. **Webhook 配置错误**
   ```bash
   # 检查 webhook 配置
   openclaw config get channels.telegram.webhook
   ```

---

## 模型配置问题

### API 密钥问题

**症状**：
- 模型调用失败
- 认证错误
- 401 Unauthorized

**诊断**：

```bash
# 检查认证配置
openclaw config get auth

# 检查模型配置
openclaw config get agents.defaults.model

# 检查认证文件
cat ~/.openclaw/agents/main/agent/auth-profiles.json
```

**解决方案**：

```bash
# 重新配置认证
openclaw configure --section model

# 或手动设置 API 密钥
export ANTHROPIC_API_KEY="your-key-here"
export OPENAI_API_KEY="your-key-here"

# 重启 Gateway
openclaw gateway restart
```

### 模型不兼容

**症状**：
- 模型调用失败
- 参数不支持
- 功能缺失

**解决方案**：

```bash
# 检查模型是否支持
openclaw models list

# 更换兼容的模型
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4-20250514"
```

---

## 通道配置问题

### Telegram 机器人无响应

**诊断**：

```bash
# 检查机器人状态
openclaw channels status telegram

# 查看日志
grep telegram ~/.openclaw/logs/gateway.log | tail -20
```

**常见问题**：

1. **Bot Token 错误**
   ```bash
   # 验证 Token
   curl https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getMe
   ```

2. **Webhook 未设置**
   ```bash
   # 设置 webhook
   openclaw channels set-webhook telegram --url https://your-domain.com/webhook
   ```

3. **allowfrom 配置**
   ```bash
   # 确保你的用户 ID 在 allowlist 中
   openclaw config get channels.telegram.allowFrom
   ```

### 飞书连接问题

**诊断**：

```bash
# 检查飞书状态
openclaw channels status feishu

# 查看日志
grep feishu ~/.openclaw/logs/gateway.log | tail -20
```

**常见问题**：

1. **App ID/Secret 错误**
2. **权限不足**
3. **加密配置错误**

---

## 性能问题

### 响应缓慢

**诊断**：

```bash
# 检查系统资源
top -p $(pgrep -f openclaw)

# 检查网络延迟
ping api.anthropic.com

# 查看会话大小
openclaw sessions list
```

**优化方案**：

```bash
# 1. 启用缓存
openclaw config set agents.defaults.cache.enabled true

# 2. 限制上下文大小
openclaw config set agents.defaults.maxTokens 4096

# 3. 使用更快的模型
openclaw config set agents.defaults.model "anthropic/claude-haiku-4-20250514"
```

### 内存占用高

**解决方案**：

```bash
# 1. 清理会话
openclaw sessions prune

# 2. 限制并发会话
openclaw config set gateway.concurrency.maxSessions 5

# 3. 调整内存限制
openclaw config set agents.defaults.memory.maxSize 512
```

---

## 安全审计警告

### 常见警告及修复

#### 1. Gateway 认证缺失

**警告**：`Gateway auth missing on loopback`

**修复**：

```bash
# 设置认证 token
openclaw config set gateway.auth.token $(openssl rand -hex 32)

# 或设置密码
openclaw config set gateway.auth.password "your-secure-password"

# 重启 Gateway
openclaw gateway restart
```

#### 2. 插件安全

**警告**：`Extensions exist but plugins.allow is not set`

**修复**：

```bash
# 设置允许的插件
openclaw config set plugins.allow '["feishu","telegram"]'

# 或禁用未使用的插件
openclaw config set plugins.entries.feishu.enabled false
```

#### 3. 群组策略开放

**警告**：`Open groupPolicy with elevated tools enabled`

**修复**：

```bash
# 使用 allowlist 而不是 open
openclaw config set channels.feishu.groupPolicy "allowlist"

# 添加允许的群组
openclaw config set channels.feishu.groupAllowFrom '["group_id_1","group_id_2"]'
```

#### 4. 文件系统权限

**警告**：`State dir is readable by others`

**修复**：

```bash
# 修改目录权限
chmod 700 ~/.openclaw
```

---

## 日志分析

### 日志位置

```bash
# Gateway 日志
~/.openclaw/logs/gateway.log

# 临时日志
/tmp/openclaw/openclaw-*.log

# 系统日志（macOS）
log show --predicate 'process == "openclaw"'
```

### 查看特定类型日志

```bash
# 错误日志
grep ERROR ~/.openclaw/logs/gateway.log

# 特定通道日志
grep telegram ~/.openclaw/logs/gateway.log

# 最近一小时的日志
grep "$(date +%Y-%m-%d-%H)" ~/.openclaw/logs/gateway.log
```

---

## 常见错误代码

### HTTP 状态码

| 状态码 | 含义 | 解决方案 |
|-------|------|----------|
| 401 | 认证失败 | 检查 API 密钥 |
| 403 | 权限不足 | 检查 allowFrom 配置 |
| 404 | 资源不存在 | 检查路径和 ID |
| 429 | 请求过多 | 降低请求频率 |
| 500 | 服务器错误 | 查看日志并重试 |
| 503 | 服务不可用 | 检查网络连接 |

### OpenClaw 特定错误

| 错误信息 | 原因 | 解决方案 |
|---------|------|----------|
| `Gateway unreachable` | Gateway 未运行 | `openclaw gateway start` |
| `Tool not allowed` | 工具权限不足 | 修改 `tools.profile` |
| `Channel disconnected` | 通道连接失败 | 检查网络和认证 |
| `Session not found` | 会话不存在 | 创建新会话 |
| `Agent not found` | Agent 不存在 | 检查 Agent ID |

---

## 获取帮助

### 内置帮助

```bash
# 通用帮助
openclaw --help

# 特定命令帮助
openclaw gateway --help
openclaw agents --help
```

### 文档资源

- [官方文档](https://docs.openclaw.ai)
- [FAQ](../../FAQ.md)
- [配置指南](../stage1-stage2/04-configuration.md)

### 社区支持

1. **GitHub Issues**：提交问题和 bug 报告
2. **GitHub Discussions**：讨论和问题解答
3. **直播互动**：直播时提问
4. **微信交流群**：实时讨论

### 提交问题时请包含

1. **系统信息**
   ```bash
   openclaw status
   ```

2. **错误日志**
   ```bash
   openclaw logs --tail 100 > logs.txt
   ```

3. **配置文件**（删除敏感信息）
   ```bash
   openclaw config get > config.json
   ```

4. **复现步骤**
   - 你在做什么
   - 期望的结果
   - 实际的结果
   - 错误信息

---

## 小结

本节我们学习了：

1. ✅ **诊断工具**：doctor、status、logs
2. ✅ **工具权限问题**：最常见的配置错误及解决方法
3. ✅ **连接问题**：Gateway 和通道连接故障排查
4. ✅ **模型配置问题**：API 密钥和模型兼容性
5. ✅ **性能问题**：响应速度和内存优化
6. ✅ **安全审计**：常见安全警告的修复方法
7. ✅ **日志分析**：如何解读日志
8. ✅ **错误代码**：常见错误的含义和解决方案
9. ✅ **获取帮助**：内置帮助、文档、社区支持

## 最佳实践

1. **定期运行诊断**：`openclaw doctor`
2. **保持配置备份**：定期备份 `openclaw.json`
3. **查看日志**：遇到问题首先查看日志
4. **安全审计**：定期运行 `openclaw security audit`
5. **版本更新**：保持 OpenClaw 更新到最新版本

## 相关资源

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [官方 FAQ](https://docs.openclaw.ai/faq)
- [故障排查](https://docs.openclaw.ai/troubleshooting)
