# Skills 系统

> 学习目标：扩展 Agent 能力

## 目录

- [Skills 概述](#skills-概述)
- [Skills 位置](#skills-位置)
- [Skill 结构](#skill-结构)
- [创建自定义 Skill](#创建自定义-skill)
- [Skill 配置](#skill-配置)
- [Skill 最佳实践](#skill-最佳实践)
- [常用 Skills 示例](#常用-skills-示例)
- [小结](#小结)

---

## Skills 概述

### 什么是 Skill？

Skill 是可重用的 Agent 能力模块，类似于函数或插件，可以扩展 Agent 的功能。

### Skills 层级

```
Skills 层级
├── Bundled Skills  (内置)
│   ├── coding
│   ├── writing
│   ├── research
│   └── analysis
├── Local Skills   (本地)
│   └── ~/.openclaw/skills/
│       ├── my-custom-skill/
│       └── another-skill/
└── Workspace Skills (项目)
    └── /workspace/skills/
        ├── project-specific/
        └── team-skill/
```

### 优先级

```
Workspace Skills > Local Skills > Bundled Skills
```

---

## Skills 位置

### 1. Bundled Skills

内置在 OpenClaw 中，自动可用：

```bash
# 查看内置 Skills
openclaw skills list --bundled

# 输出:
# - coding: 代码编写和审查
# - writing: 写作和编辑
# - research: 信息收集和分析
# - analysis: 数据分析
```

### 2. Local Skills

用户级别，所有 Agent 共享：

```bash
~/.openclaw/skills/
├── my-custom-skill/
│   ├── skill.md       # Skill 定义
│   ├── config.json    # 配置（可选）
│   └── files/         # 附加文件（可选）
└── another-skill/
    ├── skill.md
    └── config.json
```

### 3. Workspace Skills

项目级别，特定 Agent 使用：

```bash
/workspace/skills/
├── project-coding/
│   ├── skill.md
│   ├── templates/     # 代码模板
│   └── examples/      # 示例
└── team-workflow/
    ├── skill.md
    └── checklists.md
```

---

## Skill 结构

### 最小 Skill

只有一个 `skill.md` 文件：

```markdown
# my-skill

## 描述

这是一个简单的 Skill 示例。

## 触发条件

当用户询问关于 X 的话题时，使用此 Skill。

## 使用说明

1. 执行步骤 A
2. 执行步骤 B
3. 返回结果
```

### 完整 Skill

包含所有可选文件：

```
my-skill/
├── skill.md          # 必需：Skill 定义
├── config.json       # 可选：配置
├── environment.md    # 可选：环境说明
├── examples.md       # 可选：使用示例
└── files/            # 可选：附加文件
    ├── template.txt
    └── data.json
```

### skill.md 结构

```markdown
# Skill 名称

## 描述
简短描述这个 Skill 的功能。

## 触发条件
描述何时应该使用这个 Skill。

## 使用说明
详细的操作步骤。

## 输入格式
描述预期的输入格式。

## 输出格式
描述输出格式。

## 限制
任何限制或注意事项。

## 示例
使用示例。
```

---

## 创建自定义 Skill

### 步骤 1：创建 Skill 目录

```bash
# Local Skill
mkdir -p ~/.openclaw/skills/my-skill

# Workspace Skill
mkdir -p /workspace/skills/my-skill
```

### 步骤 2：编写 skill.md

```markdown
# code-reviewer

## 描述

自动审查代码并提供改进建议。

## 触发条件

当用户请求：
- "审查这个代码"
- "code review"
- "检查代码质量"
- "改进建议"

## 使用说明

1. **获取代码**
   - 使用 read 工具读取文件
   - 或让用户提供代码片段

2. **分析代码**
   - 检查代码风格
   - 识别潜在问题
   - 评估可读性
   - 检查安全性

3. **搜索最佳实践**
   - 使用 web_search 查找语言特定的最佳实践
   - 查找官方文档

4. **生成报告**
   - 总结发现
   - 列出具体问题
   - 提供改进建议
   - 给出示例代码

## 输入格式

- 文件路径
- 或代码片段

## 输出格式

```markdown
# 代码审查报告

## 总体评价
[评分和简要评论]

## 发现的问题
1. [问题描述]
   - 位置: [文件:行号]
   - 严重性: [高/中/低]
   - 建议: [改进建议]

## 改进建议
[具体建议和示例]

## 最佳实践
[相关的最佳实践]
```

## 限制

- 支持的语言：Python, JavaScript, TypeScript, Java, Go
- 最大文件大小：1MB
- 不支持多文件项目（暂定）

## 示例

用户: "审查 src/utils.js"

Agent:
[执行审查流程]
```

### 步骤 3：创建配置（可选）

`config.json`:

```json
{
  "enabled": true,
  "priority": 1,
  "languages": ["python", "javascript", "typescript"],
  "maxFileSize": 1048576,
  "rules": {
    "style": true,
    "security": true,
    "performance": true,
    "bestPractices": true
  }
}
```

### 步骤 4：测试 Skill

```bash
# 验证 Skill
openclaw skills validate my-skill

# 测试 Skill
openclaw skills test my-skill

# 列出可用 Skills
openclaw skills list
```

---

## Skill 配置

### Agent 级别配置

```json
{
  "agents": {
    "list": [
      {
        "id": "coder",
        "skills": {
          "enabled": ["code-reviewer", "test-generator"],
          "disabled": ["general-chat"],
          "config": {
            "code-reviewer": {
              "strict": true,
              "autoFix": false
            }
          }
        }
      }
    ]
  }
}
```

### 全局配置

```json
{
  "skills": {
    "defaults": {
      "enabled": true,
      "priority": 0
    },
    "searchPaths": [
      "~/.openclaw/skills",
      "/workspace/skills"
    ],
    "autoLoad": true
  }
}
```

### Skill 环境变量

```json
{
  "skills": {
    "my-skill": {
      "environment": {
        "API_KEY": "${MY_API_KEY}",
        "ENDPOINT": "https://api.example.com",
        "TIMEOUT": 30000
      }
    }
  }
}
```

---

## Skill 最佳实践

### 1. 单一职责

每个 Skill 应该只做一件事：

```markdown
# ❌ 不好的例子 - 做太多事情
# all-in-one-skill
## 描述
这个 Skill 可以做所有事情：编码、写作、分析...

# ✅ 好的例子 - 专注单一功能
# python-import-sorter
## 描述
专门用于排序 Python 导入语句的 Skill
```

### 2. 明确触发条件

```markdown
## 触发条件

✅ 明确:
- 用户说 "format my python imports"
- 检测到 Python 文件且导入混乱
- 用户明确请求导入排序

❌ 模糊:
- 当需要时
- 自动触发
```

### 3. 提供示例

```markdown
## 示例

### 示例 1：基本用法
用户: "format my imports"
Agent: [执行导入排序]

### 示例 2：指定文件
用户: "sort imports in utils.py"
Agent: [排序 utils.py 的导入]

### 示例 3：自定义配置
用户: "format imports with isort style"
Agent: [使用 isort 风格排序]
```

### 4. 错误处理

```markdown
## 错误处理

### 文件不存在
- 检查文件路径
- 提示用户提供正确路径
- 建议使用 find 命令定位文件

### 权限问题
- 检查文件权限
- 建议使用 chmod
- 提供替代方案

### 语法错误
- 识别错误位置
- 提供修复建议
- 示例正确语法
```

### 5. 文档化

```markdown
# 技能文档

## 概述
简要说明

## 何时使用
使用场景

## 如何使用
步骤说明

## 配置选项
可选配置

## 限制
已知限制

## 故障排查
常见问题
```

---

## 常用 Skills 示例

### Skill 1: git-helper

```markdown
# git-helper

## 描述

帮助用户进行 Git 操作的最佳实践和自动化。

## 触发条件

- 用户询问 Git 相关问题
- 需要创建分支、提交、合并
- 需要解决冲突

## 使用说明

### 创建分支
1. 确认当前分支
2. 创建新分支
3. 切换到新分支

### 提交更改
1. 检查状态
2. 添加文件
3. 创建提交
4. 推送到远程

### 解决冲突
1. 识别冲突文件
2. 逐个解决冲突
3. 标记为已解决
4. 完成合并

## 示例

用户: "创建一个新的功能分支"
Agent: [执行分支创建流程]
```

### Skill 2: docker-generator

```markdown
# docker-generator

## 描述

为应用程序自动生成 Dockerfile 和 docker-compose.yml。

## 触发条件

- 用户请求容器化应用
- 项目需要 Docker 配置

## 使用说明

1. **分析项目**
   - 识别语言/框架
   - 检测依赖
   - 确定端口

2. **生成 Dockerfile**
   - 选择基础镜像
   - 设置工作目录
   - 复制依赖文件
   - 安装依赖
   - 复制应用代码
   - 暴露端口
   - 设置启动命令

3. **生成 docker-compose.yml**
   - 定义服务
   - 设置网络
   - 配置卷

4. **提供说明**
   - 构建命令
   - 运行命令
   - 其他有用命令

## 模板

### Python 应用
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

### Node.js 应用
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```
```

### Skill 3: test-generator

```markdown
# test-generator

## 描述

为代码自动生成单元测试。

## 触发条件

- 用户请求为代码生成测试
- 代码文件没有对应的测试文件

## 使用说明

1. **分析代码**
   - 识别函数和类
   - 理解功能
   - 确定测试框架

2. **生成测试**
   - 为每个函数生成测试
   - 包括正常情况
   - 包括边界情况
   - 包括错误情况

3. **提供示例**
   - 如何运行测试
   - 如何添加新测试
   - 如何调试

## 测试框架

- Python: pytest
- JavaScript: Jest
- TypeScript: Jest + ts-jest
- Java: JUnit
- Go: testing

## 示例

### Python 测试
```python
import pytest
from mymodule import add

def test_add_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-2, -3) == -5

def test_add_zero():
    assert add(5, 0) == 5

def test_add_floats():
    assert add(0.1, 0.2) == pytest.approx(0.3)
```
```

### Skill 4: api-client-generator

```markdown
# api-client-generator

## 描述

根据 OpenAPI/Swagger 规范生成 API 客户端代码。

## 触发条件

- 提供 OpenAPI 规范
- 需要与 API 交互

## 使用说明

1. **获取规范**
   - 下载 OpenAPI 文档
   - 或提供 URL

2. **分析规范**
   - 识别端点
   - 了解参数
   - 了解响应格式

3. **生成客户端**
   - 选择语言
   - 生成代码
   - 提供文档

## 支持的语言

- Python (requests)
- JavaScript (fetch)
- TypeScript (axios)
- Java (OkHttp)
- Go (net/http)

## 输出示例

```python
# Generated Python client

class APIClient:
    def __init__(self, base_url, api_key):
        self.base_url = base_url
        self.api_key = api_key

    def get_user(self, user_id):
        url = f"{self.base_url}/users/{user_id}"
        headers = {"Authorization": f"Bearer {self.api_key}"}
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        return response.json()

    def create_user(self, data):
        url = f"{self.base_url}/users"
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }
        response = requests.post(url, json=data, headers=headers)
        response.raise_for_status()
        return response.json()
```
```

---

## 小结

本节我们学习了：

1. ✅ Skills 概述 - 可重用的能力模块
2. ✅ Skills 位置 - Bundled、Local、Workspace
3. ✅ Skill 结构 - skill.md 和可选文件
4. ✅ 创建自定义 Skill - 完整步骤和示例
5. ✅ Skill 配置 - Agent 和全局配置
6. ✅ Skill 最佳实践 - 单一职责、明确触发、示例、错误处理、文档
7. ✅ 常用 Skills 示例 - git-helper, docker-generator, test-generator, api-client-generator

## 实践练习

1. 创建一个代码格式化 Skill
2. 创建一个日志分析 Skill
3. 创建一个数据库迁移 Skill
4. 创建一个部署自动化 Skill

## 下一步

[多 Agent 配置](./15-multi-agent.md) - 配置专门的 Agent
