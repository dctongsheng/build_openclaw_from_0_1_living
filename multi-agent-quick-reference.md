# OpenClaw 多 Agent 快速参考

> **配置日期：** 2026年3月7日
> **状态：** ✅ 运行正常

## 🎯 三大 Agent 一览

| Agent | 名称 | 模型 | Bot | 专长 |
|-------|------|------|-----|------|
| main | 🦐 小虾米 | GLM-4.7 | 默认 bot | 全能协调、复杂任务 |
| research | 📚 研究助手 | GLM-4.7-Flash | Research bot | 信息收集、数据分析 |
| life | 🌟 生活助手 | GLM-4.7-Flash | Life bot | 日常对话、生活建议 |

## 📁 关键文件位置

```
~/.openclaw/
├── openclaw.json                      # 主配置文件
├── openclaw.json.backup-20260307      # 配置备份
├── workspace/                         # 小虾米工作区
├── workspace-research/                # 研究助手工作区
├── workspace-life/                    # 生活助手工作区
└── agents/
    ├── main/agent/                    # 小虾米状态目录
    ├── research/agent/                # 研究助手状态目录
    └── life/agent/                    # 生活助手状态目录
```

## 🔧 常用命令

### 查看 Agent 状态
```bash
openclaw agents list --bindings
```

### 查看通道状态
```bash
openclaw channels status
```

### 重启 Gateway
```bash
openclaw gateway restart
```

### 查看 Agent 日志
```bash
openclaw logs
```

## 🤝 Agent 间通信

### sessions_send - 消息通信
```javascript
sessions_send({
  agentId: "research",  // 或 "life"
  message: "请帮我研究XXX"
})
```

### sessions_spawn - 任务委派
```javascript
sessions_spawn({
  agentId: "life",
  task: "制定一个周末放松计划"
})
```

## 🧪 快速测试

### 测试研究助手
```
帮我研究一下最新的 AI Agent 架构趋势
```

### 测试生活助手
```
今天天气怎么样？有什么放松的建议吗？
```

### 测试主 agent 协作
```
请研究助手调研技术方案，生活助手调研用户体验，然后整合成建议
```

## ⚠️ 常见问题

### 问题：Bot 显示 "access not configured"

**解决：**
```bash
# 方法 1：批准配对
openclaw pairing approve telegram <配对码>

# 方法 2：修改配置添加用户 ID
# 编辑 openclaw.json，在对应 bot 的配置中添加：
"allowFrom": ["你的Telegram ID"]
```

### 问题：主 agent 不知道有员工

**解决：**
```bash
# 重启 gateway 让 agent 重新加载工作区
openclaw gateway restart
```

## 📊 Agent 选择指南

### 什么时候用小虾米（main）？
- ✅ 复杂任务需要协调
- ✅ 需要整合多个 agent 的结果
- ✅ 技术性较强的问题
- ✅ 涉及敏感信息的任务

### 什么时候用研究助手？
- ✅ 需要深入研究和分析
- ✅ 需要查找和验证信息
- ✅ 需要数据支持或引用来源
- ✅ 技术调研和竞品分析

### 什么时候用生活助手？
- ✅ 轻松简单的任务
- ✅ 需要友好的对话风格
- ✅ 生活相关的小问题
- ✅ 需要快速回应的日常询问

## 🔄 协作模式

### 串行协作
```
你 → 研究助手(调研) → 你 → 生活助手(改写) → 你 → 用户
```

### 并行协作
```
你 → 研究助手(技术) ↘
    → 你整合 → 用户
你 → 生活助手(体验) ↗
```

### 咨询式协作
```
你 → 生活助手(咨询) → 你 → 研究助手(实现) → 你 → 用户
```

## 💡 优化建议

### 调整 Agent 个性
编辑对应工作区的文件：
- `AGENTS.md` - 行为指令
- `SOUL.md` - 性格特征
- `TOOLS.md` - 工具笔记

### 添加更多路由规则
编辑 `openclaw.json` 的 `bindings` 部分：
```json
"bindings": [
  {
    "agentId": "research",
    "match": {
      "channel": "telegram",
      "accountId": "research",
      "peer": { "kind": "group", "id": "特定群组ID" }
    }
  }
]
```

## 📚 参考资源

- [完整配置指南](./multi-agent-setup-guide.md)
- [OpenClaw 官方文档](https://docs.openclaw.ai/)
- [多 Agent 文档](https://docs.openclaw.ai/concepts/multi-agent)

---

**快速上手完成！现在你有三个专业助手为你服务了！** 🎉
