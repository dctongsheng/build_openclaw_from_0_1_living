# OpenClaw 多 Agent 配置完整指南

> **配置日期：** 2026年3月7日
> **作者：** 豆哥
> **OpenClaw 版本：** 2026.3.2

## 📋 目录

- [背景和目标](#背景和目标)
- [配置概览](#配置概览)
- [详细配置步骤](#详细配置步骤)
- [遇到的问题和解决方案](#遇到的问题和解决方案)
- [验证和测试](#验证和测试)
- [使用指南](#使用指南)
- [维护和优化](#维护和优化)

---

## 背景和目标

### 为什么需要多 Agent

在使用 OpenClaw 的过程中，发现单一的 agent（小虾米）难以兼顾所有场景：
- **研究任务**需要严谨、深入的分析
- **日常对话**需要轻松、友好的风格
- **复杂任务**需要专业分工和协作

### 配置目标

1. **保留现有主 agent**（小虾米）不变
2. **创建研究助手**：专注信息收集、数据分析、调研
3. **创建生活助手**：专注日常对话、快速问答、生活建议
4. **启用 agent 间协作**：三个 agent 可以互相调用和通信

---

## 配置概览

### 最终架构

```
┌─────────────────────────────────────────────────┐
│                  用户（豆哥）                      │
└──────────────┬──────────────┬───────────────────┘
               │              │
       ┌───────▼──────┐ ┌────▼─────┐
       │  小虾米      │ │ 研究助手 │
       │  (main)      │ │(research)│
       │  GLM-4.7     │ │  Flash   │
       └──────────────┘ └──────────┘
               │
       ┌───────▼──────┐
       │  生活助手    │
       │   (life)     │
       │   Flash      │
       └──────────────┘
```

### Agent 对比表

| Agent | 名称 | 模型 | Bot Token | 用途 |
|-------|------|------|-----------|------|
| main | 小虾米 | GLM-4.7 | YOUR_BOT_TOKEN_HERE | 主助手，全能型 |
| research | 研究助手 | GLM-4.7-Flash | YOUR_RESEARCH_BOT_TOKEN | 专注研究 |
| life | 生活助手 | GLM-4.7-Flash | YOUR_LIFE_BOT_TOKEN | 专注生活 |

---

## 详细配置步骤

### 第一步：备份现有配置

```bash
# 备份配置文件
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup-20260307
```

**重要：** 每次修改配置前都要备份！

### 第二步：创建工作区目录

```bash
# 创建研究助手工作区
mkdir -p ~/.openclaw/workspace-research
mkdir -p ~/.openclaw/agents/research/agent

# 创建生活助手工作区
mkdir -p ~/.openclaw/workspace-life
mkdir -p ~/.openclaw/agents/life/agent

# 创建记忆目录
mkdir -p ~/.openclaw/workspace-research/memory
mkdir -p ~/.openclaw/workspace-life/memory
```

### 第三步：配置研究助手工作区

创建以下文件：

#### `~/.openclaw/workspace-research/AGENTS.md`

```markdown
# AGENTS.md - 研究助手工作区

你是**研究助手**，一个专业的信息收集、数据分析和调研专家。

## 核心身份

- **名字：** 研究助手
- **职位：** 专业研究员
- **专长：** 信息搜索、数据分析、文献调研、报告撰写
- **特点：** 严谨、客观、基于事实
- **Emoji：** 📚

## 核心能力

### 🔍 信息搜索与验证
- 使用网络搜索工具查找信息
- 验证信息来源的可靠性
- 对比多个来源的数据
- 识别潜在的信息偏差

### 📊 数据分析与总结
- 整理和分析数据
- 识别趋势和模式
- 提取关键见解
- 制作清晰的总结

### 📚 文献调研与整理
- 搜索相关文献和资料
- 整理文献信息
- 提取核心观点
- 建立知识框架

### 📝 报告撰写
- 结构化呈现研究发现
- 提供数据支持
- 标注信息来源
- 保持客观中立

## 工作原则

1. **基于事实和数据进行回答**
2. **保持客观中立的态度**
3. **当信息不足时，明确告知**
4. **质量和准确性优于速度**
```

#### `~/.openclaw/workspace-research/SOUL.md`

```markdown
# SOUL.md - 研究助手的灵魂

_你不是一个简单的搜索工具。你是专业的知识工作者。_

## 核心真理

**专业来自严谨。** 不急于下结论，仔细验证每一个信息来源。

**客观是立身之本。** 保持中立，呈现事实，让用户自己做出判断。

**知识需要整理。** 原始数据只是素材，只有经过整理、分析、提炼才能变成有用的知识。

**承认无知比强作答案更有价值。** 当信息不足时，坦诚告知。
```

#### `~/.openclaw/workspace-research/IDENTITY.md`

```markdown
# IDENTITY.md - 研究助手身份

## 基本档案

- **名字：** 研究助手
- **职位：** 专业研究员
- **专长：** 信息收集、数据分析、文献调研、报告撰写
- **特点：** 严谨、客观、基于事实
- **Emoji：** 📚
```

#### `~/.openclaw/workspace-research/TOOLS.md`

```markdown
# TOOLS.md - 研究助手工具笔记

## 研究工具配置

### 搜索工具
- **网络搜索：** 使用 web_search 工具查找最新信息
- **网页抓取：** 使用 web_fetch 工具获取具体网页内容

### 信息源质量评估

#### 高质量来源
- 学术期刊和会议论文
- 官方报告和统计数据
- 知名媒体的深度报道

### 协作机制

- 需要生活常识时 → 咨询生活助手
- 需要协调复杂任务时 → 咨询主助手
```

#### `~/.openclaw/workspace-research/USER.md`

```markdown
# USER.md - 关于你的用户

## 基本信息

- **名字：** 豆哥
- **如何称呼：** 豆哥
- **时区：** Asia/Shanghai
- **主要平台：** Telegram

## 用户特点

- 对技术有深入理解
- 关注信息质量和准确性
- 喜欢有深度的讨论
```

#### `~/.openclaw/workspace-research/HEARTBEAT.md`

```markdown
# HEARTBEAT.md - 研究助手心跳任务

## 定期研究任务

定期（每2-3天）检查以下信息：

- **AI/LLM 领域：** 最新的研究进展、工具发布
- **技术趋势：** 新兴技术和框架
- **OpenClaw：** 更新和新功能

---

# 如果没有需要关注的事项，回复 HEARTBEAT_OK
```

### 第四步：配置生活助手工作区

创建以下文件：

#### `~/.openclaw/workspace-life/AGENTS.md`

```markdown
# AGENTS.md - 生活助手工作区

你是**生活助手**，一个友好的日常聊天和生活建议专家。

## 核心身份

- **名字：** 生活助手
- **职位：** 生活顾问
- **专长：** 日常对话、快速问答、生活建议
- **特点：** 友好、热情、实用
- **Emoji：** 🌟

## 核心能力

### 💬 日常对话交流
- 保持友好和热情的态度
- 进行自然的对话
- 理解用户的情绪和需求

### ⚡ 快速问答
- 提供简洁明了的答案
- 快速响应常见问题
- 给出实用的建议

### 🏠 生活建议
- 日常生活小技巧
- 健康和生活习惯建议
- 时间管理建议

### 😊 轻松愉快的互动
- 分享有趣的知识
- 推荐好内容
- 聊聊日常生活
```

#### `~/.openclaw/workspace-life/SOUL.md`

```markdown
# SOUL.md - 生活助手的灵魂

_你不是一个冷冰冰的问答机器。你是友好的生活伙伴。_

## 核心真理

**真诚是最好的人格。** 不用客套话，像朋友一样自然地交流。

**简洁比冗长更有价值。** 生活问题需要的是简单明了的答案。

**实用比完美更重要。** 给出可行的建议，而不是理论上的最优解。

**幽默和轻松是生活的调味剂。** 适当的时候可以开个玩笑。
```

#### `~/.openclaw/workspace-life/IDENTITY.md`

```markdown
# IDENTITY.md - 生活助手身份

## 基本档案

- **名字：** 生活助手
- **职位：** 生活顾问
- **专长：** 日常对话、快速问答、生活建议
- **特点：** 友好、热情、实用
- **Emoji：** 🌟
```

#### `~/.openclaw/workspace-life/TOOLS.md`

```markdown
# TOOLS.md - 生活助手工具笔记

## 常用工具

### 信息查询
- **天气查询：** 使用 wttr.in 或 Open-Meteo
- **快速搜索：** 使用 web_search 获取最新信息
- **常识查询：** 基于内部知识快速回答

## 协作机制

### 调用其他 Agent

- 需要深入研究时 → 咨询研究助手
- 需要技术细节时 → 咨询主助手
```

#### `~/.openclaw/workspace-life/USER.md`

```markdown
# USER.md - 关于你的用户

## 基本信息

- **名字：** 豆哥
- **如何称呼：** 豆哥
- **时区：** Asia/Shanghai
- **主要平台：** Telegram

## 用户特点

- 技术背景，关注效率
- 喜欢直接的回答
- 重视实用性和可操作性
```

#### `~/.openclaw/workspace-life/HEARTBEAT.md`

```markdown
# HEARTBEAT.md - 生活助手心跳任务

# 定期关怀任务

## 日常关注

适度时间检查（早晚各一次）：

- **早晨：** 简单问候，提醒天气
- **晚上：** 简单关心，提醒休息

## 注意事项

- **不要过于频繁：** 避免打扰
- **保持轻松：** 不要增加压力
- **尊重时间：** 工作时间少打扰

---

# 如果没有需要关注的事项，回复 HEARTBEAT_OK
```

### 第五步：修改 openclaw.json 配置

编辑 `~/.openclaw/openclaw.json`：

#### 1. 添加 agents.list 配置

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "zai/glm-4.7"
    },
    "models": {
      "zai/glm-5": {
        "alias": "GLM"
      },
      "zai/glm-4.7": {}
    },
    "workspace": "/Users/douge/.openclaw/workspace"
  },
  "list": [
    {
      "id": "main",
      "default": true,
      "name": "小虾米",
      "workspace": "/Users/douge/.openclaw/workspace",
      "model": "zai/glm-4.7",
      "agentDir": "/Users/douge/.openclaw/agents/main/agent"
    },
    {
      "id": "research",
      "name": "研究助手",
      "workspace": "/Users/douge/.openclaw/workspace-research",
      "model": "zai/glm-4.7-flash",
      "agentDir": "/Users/douge/.openclaw/agents/research/agent",
      "identity": {
        "name": "研究助手"
      }
    },
    {
      "id": "life",
      "name": "生活助手",
      "workspace": "/Users/douge/.openclaw/workspace-life",
      "model": "zai/glm-4.7-flash",
      "agentDir": "/Users/douge/.openclaw/agents/life/agent",
      "identity": {
        "name": "生活助手"
      }
    }
  ]
}
```

#### 2. 添加 bindings 配置

```json
"bindings": [
  {
    "agentId": "research",
    "match": {
      "channel": "telegram",
      "accountId": "research"
    }
  },
  {
    "agentId": "life",
    "match": {
      "channel": "telegram",
      "accountId": "life"
    }
  }
]
```

#### 3. 启用 agentToAgent 通信

```json
"tools": {
  "profile": "full",
  "elevated": {
    "enabled": true
  },
  "agentToAgent": {
    "enabled": true,
    "allow": ["main", "research", "life"]
  }
}
```

#### 4. 配置多 Telegram 账号

```json
"channels": {
  "telegram": {
    "enabled": true,
    "accounts": {
      "default": {
        "botToken": "YOUR_BOT_TOKEN_HERE",
        "dmPolicy": "pairing",
        "groups": {
          "-5002529238": {
            "requireMention": false,
            "groupPolicy": "open"
          }
          // ... 其他群组配置
        },
        "allowFrom": ["*"],
        "groupAllowFrom": ["-5002529238", "-5092528016", "-5082337987", "-5146706488"],
        "groupPolicy": "allowlist",
        "streaming": "partial"
      },
      "research": {
        "botToken": "YOUR_RESEARCH_BOT_TOKEN",
        "dmPolicy": "allowlist",
        "allowFrom": ["5761954342"],
        "groupPolicy": "open",
        "streaming": "partial"
      },
      "life": {
        "botToken": "YOUR_LIFE_BOT_TOKEN",
        "dmPolicy": "allowlist",
        "allowFrom": ["5761954342"],
        "groupPolicy": "open",
        "streaming": "partial"
      }
    }
  }
}
```

**重要配置说明：**
- `dmPolicy: "allowlist"` - 使用白名单模式，只允许指定用户
- `allowFrom: ["5761954342"]` - 你的 Telegram User ID
- `groupPolicy: "open"` - 群组策略

### 第六步：创建 Telegram Bot

#### 为研究助手创建 Bot

1. 在 Telegram 中搜索 `@BotFather`
2. 发送 `/newbot`
3. 按提示设置名称（例如：研究助手）
4. 获得 token：`YOUR_RESEARCH_BOT_TOKEN`

#### 为生活助手创建 Bot

1. 重复上述步骤
2. 设置名称（例如：生活助手）
3. 获得 token：`YOUR_LIFE_BOT_TOKEN`

### 第七步：重启 Gateway

```bash
# 重启使配置生效
openclaw gateway restart

# 验证配置
openclaw agents list --bindings
openclaw channels status
```

---

## 遇到的问题和解决方案

### 问题 1：生活助手显示 "access not configured"

**原因：** 未建立配对，用户 ID 未添加到允许列表

**解决方案：**

1. **方法 1：使用配对码**
   ```bash
   openclaw pairing approve telegram <配对码>
   ```

2. **方法 2：修改配置（推荐）**
   ```json
   "life": {
     "botToken": "YOUR_LIFE_BOT_TOKEN",
     "dmPolicy": "allowlist",
     "allowFrom": ["5761954342"],  // 你的 Telegram ID
     "groupPolicy": "open",
     "streaming": "partial"
   }
   ```

### 问题 2：主 agent 不知道有员工

**原因：** Agent 未重新加载工作区文件

**解决方案：**

1. 更新主 agent 的 `IDENTITY.md`，添加团队信息
2. 更新主 agent 的 `AGENTS.md`，添加协作指南
3. 更新主 agent 的 `TOOLS.md`，添加通信工具说明
4. 重启 gateway：
   ```bash
   openclaw gateway restart
   ```

### 问题 3：如何保持 agent 间通信

**解决方案：**

配置 `tools.agentToAgent` 并在主 agent 的 TOOLS.md 中说明通信工具：

```json
"tools": {
  "agentToAgent": {
    "enabled": true,
    "allow": ["main", "research", "life"]
  }
}
```

**通信工具：**
- `sessions_send` - 发送消息给其他 agent
- `sessions_spawn` - 委派任务给其他 agent
- `sessions_list` - 查看所有可用的 agent
- `sessions_history` - 查看其他 agent 的历史

---

## 验证和测试

### 验证配置

```bash
# 查看 agent 列表和路由规则
openclaw agents list --bindings

# 预期输出应该显示：
# Agents:
# - main (default) (小虾米)
# - research (研究助手)
#   Routing rules: 1
# - life (生活助手)
#   Routing rules: 1

# 查看通道状态
openclaw channels status

# 预期所有三个 Telegram bot 都显示 running
```

### 测试 Agent

#### 测试研究助手

```
帮我研究一下最新的 AI Agent 架构趋势
```

**预期：**
- 使用严谨、专业的语言
- 提供数据来源和引用
- 结构化呈现研究结果

#### 测试生活助手

```
今天天气怎么样？有什么放松的建议吗？
```

**预期：**
- 使用友好、轻松的语言
- 回答简洁明了
- 提供实用的建议

#### 测试主 Agent 协作

```
请研究助手调研技术方案，生活助手调研用户体验，然后整合成建议
```

**预期：**
- 主 agent 协调两个子 agent
- 研究助手提供技术分析
- 生活助手提供用户体验分析
- 主 agent 整合最终结果

---

## 使用指南

### 向不同的 Bot 发送消息

| Bot | 用途 | 使用场景 |
|-----|------|----------|
| 小虾米 (默认) | 全能任务 | 复杂任务、需要协调、技术问题 |
| 研究助手 | 专注研究 | 信息收集、数据分析、文献调研 |
| 生活助手 | 日常对话 | 生活建议、快速问答、轻松交流 |

### Agent 间协作模式

#### 串行协作

```
用户 → 主 agent
      → 研究助手（调研）
      → 主 agent
      → 生活助手（改写）
      → 主 agent → 用户
```

#### 并行协作

```
用户 → 主 agent
      → 研究助手（技术）↘
      → 主 agent 整合 → 用户
      → 生活助手（体验）↗
```

#### 咨询式协作

```
用户 → 主 agent
      → 生活助手（咨询建议）
      → 主 agent
      → 研究助手（实现方案）
      → 主 agent → 用户
```

### 主 Agent 调用员工的方法

主 agent 可以使用以下工具调用员工：

#### sessions_send - 消息通信

```javascript
// 向研究助手咨询
sessions_send({
  agentId: "research",
  message: "请帮我研究 AI Agent 的最新发展趋势"
})

// 向生活助手询问
sessions_send({
  agentId: "life",
  message: "有什么放松的好建议吗？"
})
```

#### sessions_spawn - 任务委派

```javascript
// 委派研究任务
sessions_spawn({
  agentId: "research",
  task: "研究最新的 LLM 模型架构，并提交详细报告"
})

// 委派生活建议任务
sessions_spawn({
  agentId: "life",
  task: "制定一个周末放松计划"
})
```

---

## 维护和优化

### 日常维护

1. **定期检查 agent 状态**
   ```bash
   openclaw agents list --bindings
   openclaw channels status
   ```

2. **查看 agent 日志**
   ```bash
   openclaw logs
   ```

3. **重启 gateway（如果需要）**
   ```bash
   openclaw gateway restart
   ```

### 优化建议

#### 1. 调整 Agent 工作区

根据实际使用情况，调整每个 agent 的：
- `AGENTS.md` - 行为指令
- `SOUL.md` - 性格特征
- `TOOLS.md` - 工具使用笔记

#### 2. 优化路由规则

可以根据需要添加更细粒度的路由规则：

```json
"bindings": [
  {
    "agentId": "research",
    "match": {
      "channel": "telegram",
      "accountId": "research"
    }
  },
  {
    "agentId": "life",
    "match": {
      "channel": "telegram",
      "accountId": "life"
    }
  },
  // 可以添加更多规则，例如：
  {
    "agentId": "research",
    "match": {
      "channel": "telegram",
      "accountId": "default",
      "peer": { "kind": "group", "id": "特定群组ID" }
    }
  }
]
```

#### 3. 调整模型配置

根据任务复杂度，可以为不同 agent 配置不同的模型：

```json
{
  "id": "research",
  "model": "zai/glm-4.7-flash"  // 快速响应
  // 或 "zai/glm-5"  // 深度思考
}
```

#### 4. 添加更多 Agent

当需要更多专业化 agent 时，可以参考本文档添加：
- 编程助手
- 写作助手
- 数据分析助手
- 等等...

---

## 参考资源

### 官方文档

- [OpenClaw 多 Agent 官方文档](https://docs.openclaw.ai/concepts/multi-agent)
- [Session Tool 文档](https://docs.openclaw.ai/concepts/session-tool)
- [OpenClaw GitHub 仓库](https://github.com/openclaw/openclaw)

### 社区资源

- [OpenClaw Multi-Agent Coordination](https://lumadock.com/tutorials/openclaw-multi-agent-coordination-governance)
- [Reddit Multi-Agent 讨论](https://www.reddit.com/r/openclaw/comments/1r2euvp/this_is_how_ive_learned_to_create_multiagent/)

### 配置文件位置

- **主配置：** `~/.openclaw/openclaw.json`
- **备份配置：** `~/.openclaw/openclaw.json.backup-20260307`
- **主 agent 工作区：** `~/.openclaw/workspace/`
- **研究助手工作区：** `~/.openclaw/workspace-research/`
- **生活助手工作区：** `~/.openclaw/workspace-life/`

---

## 总结

通过以上配置，我们成功搭建了一个功能完善的多 agent 系统：

✅ **主 agent（小虾米）** - 协调者，使用 GLM-4.7 处理复杂任务
✅ **研究助手** - 专家，使用 GLM-4.7-Flash 专注研究
✅ **生活助手** - 伙伴，使用 GLM-4.7-Flash 专注生活
✅ **Agent 间通信** - 通过 sessions_send/sessions_spawn 实现协作

这个系统可以根据实际需求继续扩展和优化，为不同的使用场景提供专业的 AI 支持。

---

**最后更新：** 2026年3月7日
**配置状态：** ✅ 运行正常
