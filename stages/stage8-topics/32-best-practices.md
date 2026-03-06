# OpenClaw 配置最佳实践

> 基于实际使用经验总结的配置建议和避坑指南

## 目录

- [核心配置原则](#核心配置原则)
- [工具权限配置](#工具权限配置)
- [安全配置建议](#安全配置建议)
- [常见陷阱及避免方法](#常见陷阱及避免方法)
- [性能优化建议](#性能优化建议)
- [部署建议](#部署建议)

---

## 核心配置原则

### 1. 最小权限原则

始终从最小权限开始，根据实际需求逐步提升：

```
初始配置 → messaging（仅消息）
     ↓
需要文件操作 → coding（编程工具）
     ↓
需要命令执行 → full + elevated（完整功能）
```

### 2. 安全第一原则

- ✅ 优先考虑安全性，而不是便利性
- ✅ 在开放环境中限制高级工具
- ✅ 定期审查配置和日志
- ✅ 使用白名单而不是黑名单

### 3. 渐进式配置

不要一次性启用所有功能，按以下顺序配置：

1. **基础配置**：模型、通道、认证
2. **工具配置**：从受限到完整
3. **安全配置**：白名单、沙箱、审计
4. **优化配置**：缓存、性能、监控

---

## 工具权限配置

### 配置层次

OpenClaw 的工具权限有三个层次：

#### 1️⃣ 全局 Profile（最粗粒度）

```json
{
  "tools": {
    "profile": "messaging"  // 或 "coding", "full"
  }
}
```

**选择建议**：
- `messaging`：纯聊天机器人，无系统访问
- `coding`：需要代码读写，但不需要命令执行
- `full`：需要完整功能

#### 2️⃣ Elevated Tools（中级粒度）

```json
{
  "tools": {
    "profile": "full",
    "elevated": {
      "enabled": true
    }
  }
}
```

**何时启用**：
- ✅ 需要执行 shell 命令（curl, git, npm 等）
- ✅ 需要进程管理
- ✅ 需要运行时控制
- ⚠️ 只在受信任的环境中使用

#### 3️⃣ 通道级配置（最细粒度）

```json
{
  "channels": {
    "telegram": {
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["trusted_group_1"]
    }
  }
}
```

**推荐配置**：
- **私聊（DM）**：`pairing` 或 `open`（根据需求）
- **群组**：始终使用 `allowlist`
- **从未设置**：`allowlist` + 添加受信任的群组

### 推荐配置组合

#### 场景 1：个人聊天助手（最安全）

```json
{
  "tools": {
    "profile": "full",
    "elevated": {
      "enabled": true
    }
  },
  "channels": {
    "telegram": {
      "dmPolicy": "pairing",
      "groupPolicy": "deny"
    }
  }
}
```

**特点**：
- ✅ 完整工具权限
- ✅ 仅私聊可用
- ✅ 无群组访问
- ✅ 最安全的 elevated 配置

#### 场景 2：团队协作助手（平衡）

```json
{
  "tools": {
    "profile": "coding",
    "elevated": {
      "enabled": false
    }
  },
  "channels": {
    "telegram": {
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["team_group_id"],
      "groupMentionTrigger": "@ai"
    }
  }
}
```

**特点**：
- ✅ 编程相关工具
- ✅ 无命令执行能力
- ✅ 白名单群组
- ✅ 需要 @ai 提及

#### 场景 3：开发环境助手（完全开放）

```json
{
  "tools": {
    "profile": "full",
    "elevated": {
      "enabled": true
    }
  },
  "channels": {
    "telegram": {
      "dmPolicy": "open",
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["dev_team_group"]
    },
    "agents": {
      "defaults": {
        "sandbox": {
          "mode": "none"
        }
      }
    }
  }
}
```

**特点**：
- ⚠️ 完整功能
- ⚠️ 需要沙箱配置
- ⚠️ 仅限开发环境
- ❌ 不推荐用于生产环境

---

## 安全配置建议

### 1. Gateway 认证

**问题**：默认配置可能没有认证

**解决方案**：
```bash
# 设置认证 token
openclaw config set gateway.auth.token $(openssl rand -hex 32)

# 或设置密码
openclaw config set gateway.auth.password "your-secure-password"

# 重启生效
openclaw gateway restart
```

### 2. 群组访问控制

**问题**：开放群组 + elevated tools = 极高风险

**解决方案**：
```bash
# 1. 使用 allowlist
openclaw config set channels.feishu.groupPolicy "allowlist"

# 2. 只添加受信任的群组
openclaw config set channels.feishu.groupAllowFrom '["group_1", "group_2"]'

# 3. 设置提及触发
openclaw config set channels.feishu.groupMentionTrigger "@ai"
```

### 3. 插件安全

**问题**：自动加载插件可能带来安全风险

**解决方案**：
```bash
# 明确指定允许的插件
openclaw config set plugins.allow '["telegram", "feishu"]'

# 禁用未使用的插件
openclaw config set plugins.entries.unwanted-plugin.enabled false
```

### 4. 文件系统权限

**问题**：配置目录权限过宽

**解决方案**：
```bash
# 限制配置目录权限
chmod 700 ~/.openclaw

# 限制会话目录权限
chmod 700 ~/.openclaw/agents/main/sessions
```

### 5. 日志和监控

**推荐配置**：
```json
{
  "logging": {
    "level": "info",
    "file": "~/.openclaw/gateway.log",
    "maxSize": "100M",
    "maxFiles": 10
  }
}
```

**监控建议**：
- 定期检查日志：`tail -f ~/.openclaw/logs/gateway.log`
- 关注错误和警告：`grep ERROR ~/.openclaw/logs/gateway.log`
- 监控资源使用：`top -p $(pgrep -f openclaw)`

---

## 常见陷阱及避免方法

### ❌ 陷阱 1：忽略 tools.profile

**问题**：
- 配置向导可能设置为 `messaging`
- Agent 无法使用基本工具
- 用户困惑为什么工具不工作

**避免方法**：
```bash
# 检查配置
openclaw config get tools.profile

# 如果是 messaging，根据需求修改
openclaw config set tools.profile full  # 或 coding
```

### ❌ 陷阱 2：忘记启用 elevated tools

**问题**：
- `tools.profile` 是 `full`
- 但 exec/curl 等命令仍然无法使用
- 用户困惑为什么"full"还不够

**避免方法**：
```bash
# 检查 elevated 状态
openclaw config get tools.elevated

# 如果未启用，添加配置
openclaw config set tools.elevated.enabled true
```

### ❌ 陷阱 3：开放群组 + elevated tools

**问题**：
- 群组设置为 `open`
- elevated tools 已启用
- 任何群组成员都可以执行命令
- **极高风险**

**避免方法**：
```bash
# 1. 检查群组策略
openclaw config get channels.feishu.groupPolicy

# 2. 如果是 open，改为 allowlist
openclaw config set channels.feishu.groupPolicy "allowlist"

# 3. 添加受信任的群组
openclaw config set channels.feishu.groupAllowFrom '["trusted_group"]'
```

### ❌ 陷阱 4：没有配置认证

**问题**：
- Gateway 没有认证
- 任何人都可以访问控制面板
- 本地服务被暴露到公网

**避免方法**：
```bash
# 始终设置认证
openclaw config set gateway.auth.mode "token"
openclaw config set gateway.auth.token $(openssl rand -hex 32)
```

### ❌ 陷阱 5：忽略安全审计

**问题**：
- 安全审计显示警告
- 用户忽略警告
- 配置存在安全隐患

**避免方法**：
```bash
# 定期运行安全审计
openclaw security audit

# 阅读并修复所有 CRITICAL 级别问题
# 考虑修复 WARN 级别问题
```

---

## 性能优化建议

### 1. 会话管理

```json
{
  "sessions": {
    "pruning": {
      "enabled": true,
      "maxTokens": 100000,
      "maxMessages": 100
    }
  }
}
```

### 2. 缓存配置

```json
{
  "agents": {
    "defaults": {
      "cache": {
        "enabled": true,
        "ttl": 3600
      }
    }
  }
}
```

### 3. 并发控制

```json
{
  "gateway": {
    "concurrency": {
      "maxSessions": 10,
      "maxMessagesPerSession": 5
    }
  }
}
```

---

## 部署建议

### 开发环境

- ✅ 使用 `full` profile
- ✅ 启用 elevated tools
- ✅ 详细日志（debug 级别）
- ✅ 本地 Gateway 绑定

### 测试环境

- ✅ 使用 `coding` profile
- ⚠️ 谨慎启用 elevated tools
- ✅ 信息日志（info 级别）
- ✅ 基本认证

### 生产环境

- ✅ 使用 `messaging` 或 `coding` profile
- ❌ 禁用 elevated tools（除非绝对必要）
- ✅ 严格的白名单配置
- ✅ 强认证（token + 密码）
- ✅ 限制日志级别（warn/error）
- ✅ 监控和告警

---

## 配置检查清单

### 初次配置后

- [ ] 运行 `openclaw doctor`
- [ ] 运行 `openclaw security audit`
- [ ] 检查 `tools.profile` 是否符合需求
- [ ] 如需 elevated tools，确认已启用
- [ ] 所有群组都使用 allowlist
- [ ] Gateway 已配置认证
- [ ] 文件权限正确（700）
- [ ] 测试基本功能

### 启用 elevated tools 后

- [ ] 确认群组策略不是 `open`
- [ ] 仅在受信任的群组/私聊中使用
- [ ] 设置会话和文件访问限制
- [ ] 启用日志监控
- [ ] 定期审查安全审计报告

### 定期维护（每周）

- [ ] 运行 `openclaw security audit`
- [ ] 检查日志中的异常行为
- [ ] 审查群组白名单
- [ ] 更新 OpenClaw 到最新版本
- [ ] 清理旧会话
- [ ] 检查磁盘空间使用

---

## 快速参考

### 常用命令

```bash
# 诊断
openclaw doctor
openclaw status
openclaw security audit

# 配置
openclaw config get <path>
openclaw config set <path> <value>
openclaw configure --section model

# Gateway
openclaw gateway start
openclaw gateway restart
openclaw gateway logs

# 工具权限
openclaw config get tools.profile
openclaw config get tools.elevated
```

### 配置文件位置

```
~/.openclaw/
├── openclaw.json              # 主配置文件
├── agents/main/agent/
│   ├── auth-profiles.json     # 认证配置
│   └── models.json            # 模型配置
├── workspace/                 # Agent 工作区
└── logs/gateway.log           # Gateway 日志
```

---

## 总结

### 核心要点

1. **最小权限**：从最受限的配置开始
2. **渐进配置**：按需逐步提升权限
3. **安全第一**：优先考虑安全性
4. **定期审计**：定期检查和更新配置
5. **监控日志**：关注异常行为

### 安全优先级

```
安全性 > 便利性
白名单 > 黑名单
allowlist > open > deny
私聊 > 群聊
```

### 记住

- 🔒 如果不需要 elevated tools，不要启用
- 📋 永远使用白名单控制群组访问
- 🔐 始终配置 Gateway 认证
- 📊 定期运行安全审计
- 📝 记录配置变更和原因

---

## 相关资源

- [故障排查指南](./31-troubleshooting.md)
- [配置文件详解](../stage1-stage2/04-configuration.md)
- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [安全最佳实践](https://docs.openclaw.ai/security)
