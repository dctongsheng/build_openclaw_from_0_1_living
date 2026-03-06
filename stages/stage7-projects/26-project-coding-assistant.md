# 实战项目 1：个人 AI 编程助手

> 功能需求：多通道支持、代码审查、自动化测试、文档生成

## 目录

- [项目概述](#项目概述)
- [需求分析](#需求分析)
- [系统设计](#系统设计)
- [实现步骤](#实现步骤)
- [配置指南](#配置指南)
- [测试验证](#测试验证)
- [部署上线](#部署上线)
- [扩展功能](#扩展功能)
- [小结](#小结)

---

## 项目概述

### 项目目标

构建一个个人 AI 编程助手，能够：

- 通过多个聊天平台访问
- 代码审查和优化建议
- 自动化测试执行
- 自动生成文档

### 技术栈

```
后端:
- OpenClaw Gateway
- 多 Agent 架构
- 自定义 Skills

通道:
- WhatsApp (个人)
- Discord (开发团队)

工具:
- Git
- Docker
- 测试框架
```

---

## 需求分析

### 功能需求

```
核心功能:
├── 代码审查
│   ├── 静态分析
│   ├── 最佳实践检查
│   └── 安全性检查
├── 测试自动化
│   ├── 单元测试
│   ├── 集成测试
│   └── 测试报告
├── 文档生成
│   ├── API 文档
│   ├── 代码注释
│   └── README 生成
└── 代码优化
    ├── 性能优化
    ├── 重构建议
    └── 代码格式化
```

### 非功能需求

```
性能:
├── 响应时间 < 10s
├── 并发支持 > 5 会话
└── 准确率 > 90%

可用性:
├── 7x24 可用
├── 故障恢复 < 5min
└── 数据持久化

安全:
├── 代码隔离
├── 沙箱执行
└── 敏感信息过滤
```

---

## 系统设计

### 架构图

```
┌─────────────────────────────────────────────────────────┐
│                     User Layer                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│  │ WhatsApp   │  │  Discord   │  │  Telegram  │       │
│  └────────────┘  └────────────┘  └────────────┘       │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│                  OpenClaw Gateway                       │
│  ┌─────────────────────────────────────────────────────┐│
│  │              Agent Router                            ││
│  │  - 代码审查 Agent                                      ││
│  │  - 测试 Agent                                          ││
│  │  - 文档 Agent                                          ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│                    Tools & Skills                        │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐       │
│  │  Git   │  │  Test  │  │  Docs  │  │ Analysis│      │
│  └────────┘  └────────┘  └────────┘  └────────┘       │
└─────────────────────────────────────────────────────────┘
```

### 数据流

```
用户发送代码
  ↓
通道接收消息
  ↓
Agent 分析代码
  ↓
使用工具:
  - read: 读取文件
  - web_search: 查找最佳实践
  - exec: 运行分析工具
  ↓
生成审查报告
  ↓
发送到用户
```

---

## 实现步骤

### 步骤 1：基础配置

```bash
# 1. 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 2. 初始化配置
openclaw onboard --install-daemon

# 3. 配置 API Key
export ANTHROPIC_API_KEY="sk-ant-..."
openclaw config set models.providers.anthropic.apiKey "$ANTHROPIC_API_KEY"
```

### 步骤 2：配置通道

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+8613800138000"]
    },
    "discord": {
      "enabled": true,
      "guilds": {
        "allow": ["YOUR_DISCORD_GUILD_ID"]
      }
    }
  }
}
```

### 步骤 3：创建 Coding Agent

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "代码审查专家",
        "model": "anthropic/claude-sonnet-4-20250514",
        "workspace": "~/.openclaw/workspace-coding",
        "tools": {
          "profile": "coding",
          "allow": ["read", "write", "edit", "exec", "web_search", "web_fetch"],
          "deny": ["message", "cron", "gateway"]
        },
        "skills": {
          "enabled": ["code-analyzer", "test-generator", "doc-generator"]
        },
        "routing": {
          "keywords": ["review", "代码", "审查", "test", "测试"],
          "channels": ["whatsapp", "discord"]
        }
      }
    ]
  }
}
```

### 步骤 4：创建 Workspace

```bash
# 创建 coding workspace
mkdir -p ~/.openclaw/workspace-coding

# 创建 AGENTS.md
cat > ~/.openclaw/workspace-coding/AGENTS.md << 'EOF'
# 代码审查 Agent

## 核心职责

你是一个专业的代码审查专家，帮助用户提高代码质量。

## 主要能力

1. **代码审查** - 全面审查代码质量
2. **测试生成** - 自动生成测试用例
3. **文档生成** - 生成代码文档
4. **优化建议** - 提供性能优化建议

## 代码审查流程

1. **读取代码**
   - 使用 read 工具读取文件
   - 或让用户提供代码片段

2. **分析代码**
   - 检查代码风格
   - 识别潜在问题
   - 评估安全性
   - 检查性能

3. **查找最佳实践**
   - 使用 web_search 查找语言特定最佳实践
   - 参考官方文档

4. **生成报告**
   - 总结发现
   - 列出问题（按严重性排序）
   - 提供改进建议
   - 给出示例代码

## 审查标准

### 代码质量
- [ ] 代码清晰易读
- [ ] 遵循语言惯例
- [ ] 适当的注释
- [ ] 有意义的命名

### 性能
- [ ] 无明显性能问题
- [ ] 合理的复杂度
- [ ] 适当的资源使用

### 安全性
- [ ] 无安全漏洞
- [ ] 输入验证
- [ ] 输出编码
- [ ] 权限检查

### 测试
- [ ] 单元测试覆盖
- [ ] 边界情况处理
- [ ] 错误处理

## 响应格式

```markdown
# 代码审查报告

## 总体评价
[评分] - [简要评价]

## 发现的问题

### 🔴 严重问题
1. [问题描述]
   - 文件: [文件名:行号]
   - 问题: [详细说明]
   - 修复: [修复建议]

### 🟡 警告
[类似格式]

### 🔵 建议
[类似格式]

## 改进建议
[具体建议和示例]

## 测试建议
[需要测试的场景]

## 最佳实践
[相关的最佳实践参考]
```
EOF
```

### 步骤 5：创建 Skills

```bash
# 创建 code-analyzer skill
mkdir -p ~/.openclaw/skills/code-analyzer

cat > ~/.openclaw/skills/code-analyzer/skill.md << 'EOF'
# code-analyzer

## 描述
深度分析代码质量、性能和安全性

## 触发条件
- "分析代码"
- "code analysis"
- "检查代码"

## 分析步骤

1. 静态分析
2. 安全检查
3. 性能评估
4. 最佳实践验证
EOF
```

### 步骤 6：配置工具权限

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "tools": {
          "confirm": {
            "exec": {
              "enabled": true,
              "message": "即将执行命令: {command}\n是否继续？"
            },
            "write": {
              "enabled": true,
              "message": "即将修改文件: {path}\n是否继续？"
            }
          }
        }
      }
    ]
  }
}
```

---

## 配置指南

### 完整配置文件

```json
{
  "$schema": "https://openclaw.ai/schema/config.json",
  "agents": {
    "defaults": {
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.3,
      "maxTokens": 4096
    },
    "list": [
      {
        "id": "code-reviewer",
        "name": "代码审查专家",
        "model": "anthropic/claude-sonnet-4-20250514",
        "workspace": "~/.openclaw/workspace-coding",
        "tools": {
          "allow": ["read", "write", "edit", "exec", "web_search", "web_fetch"],
          "deny": ["message", "cron"]
        },
        "routing": {
          "keywords": ["review", "代码", "test", "测试"]
        }
      }
    ]
  },
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+8613800138000"]
    },
    "discord": {
      "enabled": true,
      "guilds": {
        "allow": ["YOUR_GUILD_ID"]
      }
    }
  },
  "models": {
    "providers": {
      "anthropic": {
        "apiKey": "${ANTHROPIC_API_KEY}"
      }
    }
  },
  "tools": {
    "confirm": {
      "exec": true,
      "write": true
    }
  },
  "security": {
    "sandbox": {
      "enabled": true,
      "level": "basic",
      "restrictions": {
        "fileSystem": {
          "allowedPaths": ["~/projects", "/tmp"]
        },
        "commands": {
          "allowed": ["python", "node", "npm", "git", "pytest"],
          "denied": ["rm", "dd", "mkfs"]
        }
      }
    }
  }
}
```

---

## 测试验证

### 测试场景

#### 场景 1：代码审查

```
用户: "审查这个文件 /path/to/code.py"

Agent:
1. 读取文件
2. 分析代码
3. 查找最佳实践
4. 生成审查报告

预期输出:
- 总体评价
- 问题列表（严重性分级）
- 改进建议
- 测试建议
```

#### 场景 2：测试生成

```
用户: "为这个函数生成测试"

Agent:
1. 分析函数功能
2. 识别边界情况
3. 生成测试代码
4. 提供测试运行说明

预期输出:
- 测试代码
- 测试说明
- 运行命令
```

#### 场景 3：文档生成

```
用户: "为这个模块生成文档"

Agent:
1. 分析代码结构
2. 识别公共 API
3. 生成文档
4. 添加示例

预期输出:
- API 文档
- 使用示例
- 参数说明
```

### 验证清单

```bash
# 测试代码审查
echo "请审查这段 Python 代码" | openclaw message send --channel discord --message -

# 测试会话管理
openclaw sessions list

# 测试工具权限
openclaw tools test --tool read --path ~/.openclaw/workspace-coding

# 查看日志
openclaw logs --tail 50
```

---

## 部署上线

### Docker 部署

```dockerfile
# Dockerfile
FROM node:22-alpine

# 安装 OpenClaw
RUN npm install -g @openclaw/gateway

# 复制配置
COPY openclaw.json /root/.openclaw/
COPY workspace-coding /root/.openclaw/workspace-coding

# 暴露端口
EXPOSE 18789

# 启动
CMD ["openclaw", "start", "--foreground"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    build: .
    container_name: coding-assistant
    restart: unless-stopped
    ports:
      - "18789:18789"
    volumes:
      - ./projects:/projects
      - openclaw-data:/root/.openclaw
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - NODE_ENV=production
```

### 启动服务

```bash
# 构建镜像
docker build -t coding-assistant .

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

---

## 扩展功能

### 功能 1：GitHub 集成

```bash
# 创建 GitHub skill
mkdir -p ~/.openclaw/skills/github-integration

# 配置 GitHub Token
export GITHUB_TOKEN="your-github-token"

# 创建 pull request
echo "创建 PR" | openclaw message send --message "
GitHub PR for feature-branch
Title: Add new feature
Description: Implement XYZ feature
"
```

### 功能 2：CI/CD 集成

```bash
# CI/CD workflow
cat > ~/.openclaw/workflows/ci-cd.yaml << 'EOF'
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: |
          npm install
          npm test
      - name: Code review
        run: |
          openclaw review --branch ${{ github.ref }}
EOF
```

### 功能 3：性能分析

```bash
# 创建 profiling skill
mkdir -p ~/.openclaw/skills/profiler

cat > ~/.openclaw/skills/profiler/skill.md << 'EOF'
# profiler

## 描述
分析代码性能并提供优化建议

## 分析内容
1. 时间复杂度
2. 空间复杂度
3. 瓶颈识别
4. 优化建议
EOF
```

---

## 小结

本节我们完成了：

1. ✅ 项目概述 - 目标和技术栈
2. ✅ 需求分析 - 功能和非功能需求
3. ✅ 系统设计 - 架构图和数据流
4. ✅ 实现步骤 - 6 个详细步骤
5. ✅ 配置指南 - 完整配置文件
6. ✅ 测试验证 - 3 个测试场景
7. ✅ 部署上线 - Docker 部署
8. ✅ 扩展功能 - GitHub、CI/CD、性能分析

## 项目成果

通过这个项目，你将：

- ✅ 掌握多通道配置
- ✅ 理解 Agent 定制
- ✅ 学习 Skill 开发
- ✅ 实践 Docker 部署
- ✅ 建立完整的 AI 编程助手

## 下一步

[实战项目 2：团队知识库机器人](./27-project-knowledge-base.md)
