# 实战项目 2-4

## 实战项目 2：团队知识库机器人

> 功能需求：Slack 集成、文档检索、问答、知识管理

### 项目概述

构建一个团队知识库机器人，能够：

- 集成到 Slack 工作区
- 智能检索文档
- 回答常见问题
- 管理团队知识

### 技术要点

```json
{
  "agents": {
    "list": [
      {
        "id": "knowledge-bot",
        "name": "知识库助手",
        "model": "anthropic/claude-sonnet-4-20250514",
        "workspace": "~/.openclaw/workspace-knowledge",
        "tools": {
          "allow": ["read", "web_search", "message"],
          "deny": ["exec", "write"]
        },
        "routing": {
          "channels": ["slack"],
          "keywords": ["docs", "知识", "help"]
        }
      }
    ]
  },
  "channels": {
    "slack": {
      "enabled": true,
      "workspaces": {
        "allow": ["T12345678"]
      },
      "appHome": true
    }
  }
}
```

### Memory 系统使用

```markdown
# AGENTS.md - 知识管理 Agent

## 知识存储

### 短期记忆
- 当前对话上下文
- 用户最近查询

### 中期记忆
- 会话历史摘要
- 常见问题库

### 长期记忆
- 文档索引
- 知识图谱
- 团队经验库

## 知识检索

1. **关键词匹配**
   - 提取查询关键词
   - 搜索文档库
   - 返回相关片段

2. **语义搜索**
   - 理解查询意图
   - 向量相似度搜索
   - 排序结果

3. **上下文增强**
   - 结合用户历史
   - 考虑团队背景
   - 提供相关链接
```

---

## 实战项目 3：自动化工作流引擎

> 功能需求：Cron 任务、多 Agent 协作、事件驱动、审批流程

### 项目概述

构建一个自动化工作流引擎，支持：

- 定时任务调度
- 多 Agent 协作
- 事件驱动响应
- 审批流程

### 技术要点

```json
{
  "plugins": {
    "lobster": {
      "enabled": true,
      "workflows": {
        "daily-report": {
          "schedule": "0 9 * * *",
          "steps": [
            {
              "agent": "research",
              "task": "收集日报数据"
            },
            {
              "agent": "writing",
              "task": "生成日报摘要"
            },
            {
              "tool": "message",
              "task": "发送到 Slack"
            }
          ]
        }
      }
    }
  }
}
```

### Lobster 工作流

```yaml
# approval-workflow.yaml
name: Code Approval Workflow
description: 代码审批流程

triggers:
  - type: webhook
    event: pull_request.opened

steps:
  - name: code-review
    agent: code-reviewer
    task: 审查代码

  - name: check-tests
    agent: tester
    task: 运行测试

  - name: approve
    condition: tests_passed == true
    agent: manager
    task: 审批通过

  - name: reject
    condition: tests_passed == false
    agent: manager
    task: 拒绝并说明原因

on:
  success:
    - message:
        channel: "#dev"
        text: "PR 已批准"

  failure:
    - message:
        channel: "#dev"
        text: "PR 被拒绝"
```

---

## 实战项目 4：客户服务系统

> 功能需求：多渠道客服、工单系统、问答、人工转接

### 项目概述

构建一个客户服务系统，支持：

- 多渠道接入
- 智能问答
- 工单管理
- 人工转接

### 技术要点

```json
{
  "agents": {
    "list": [
      {
        "id": "support-bot",
        "name": "客服助手",
        "model": "openai/gpt-4o",
        "workspace": "~/.openclaw/workspace-support",
        "tools": {
          "profile": "messaging"
        },
        "routing": {
          "channels": ["slack", "telegram", "whatsapp"]
        }
      }
    ]
  },
  "sessions": {
    "escalation": {
      "enabled": true,
      "triggers": [
        "frustration",
        "complex_issue",
        "request_human"
      ],
      "action": "transfer_to_human"
    }
  }
}
```

### 会话管理

```markdown
# AGENTS.md - 客服 Agent

## 会话处理

### 自动处理
- 常见问题
- 标准流程
- 信息查询

### 转接条件
1. 用户明确要求人工
2. 连续 3 次无效回答
3. 检测到负面情绪
4. 复杂技术问题

### 转接流程
1. 总结对话
2. 收集信息
3. 创建工单
4. 通知人工客服
5. 无缝转接
```

---

## 项目对比

| 项目 | 难度 | 时间 | 核心技能 |
|-----|------|------|---------|
| **编程助手** | ⭐⭐⭐ | 3-5 天 | 代码分析、测试生成 |
| **知识库** | ⭐⭐ | 2-3 天 | 文档检索、Slack 集成 |
| **工作流引擎** | ⭐⭐⭐⭐ | 5-7 天 | Lobster、多 Agent 协作 |
| **客服系统** | ⭐⭐⭐ | 4-6 天 | 会话管理、转接逻辑 |

---

## 第七阶段完成！

恭喜！你已经完成了项目实战学习：

- ✅ 个人 AI 编程助手
- ✅ 团队知识库机器人
- ✅ 自动化工作流引擎
- ✅ 客户服务系统

## 项目实践建议

1. **从简单开始** - 先完成知识库机器人
2. **逐步增加复杂度** - 然后做编程助手
3. **整合功能** - 最后做工作流和客服系统
4. **持续优化** - 根据使用反馈改进

## 下一步

[第八阶段：高级主题](../stage8-topics/30-performance.md) - 性能优化
