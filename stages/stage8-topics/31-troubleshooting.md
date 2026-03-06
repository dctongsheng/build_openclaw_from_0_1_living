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
