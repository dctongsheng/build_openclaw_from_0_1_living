# Workspace 使用说明

这个目录用于存放OpenClaw的workspace配置。

## 📂 目录用途

OpenClaw的workspace包含：
- Agent配置文件
- Skills技能定义
- 工具配置
- 自定义提示词

## ⚠️ 重要提示

### 敏感信息处理

workspace中的**敏感信息不会被提交到Git**，包括：
- API密钥
- Token
- 密码
- 私有配置

这些文件已被 `.gitignore` 排除。

### 跨机器使用

在新机器上使用时：

1. **拉取项目**
   ```bash
   git clone <repo-url>
   cd build_openclaw_from_0_to_1
   ```

2. **创建软链接（推荐）**
   ```bash
   # macOS/Linux
   ln -s $(pwd)/workspace ~/Library/Application\ Support/openclaw/workspace

   # 或者复制到OpenClaw默认位置
   cp -r workspace/ ~/Library/Application\ Support/openclaw/
   ```

3. **配置敏感信息**
   - 手动添加API密钥到 `openclaw.json`
   - 或使用环境变量

## 📝 建议的同步内容

**可以安全提交**：
- Agent提示词模板
- Skills定义
- 工具配置（不含密钥）
- 文档说明

**不要提交**：
- API密钥
- Token
- 个人凭据

## 🔧 配置示例

创建一个 `workspace.example/` 目录存放示例配置，方便他人参考。

## 📖 更多信息

- [返回项目首页](./README.md)
- [坑点记录](./PITFALLS.md#配置相关)
