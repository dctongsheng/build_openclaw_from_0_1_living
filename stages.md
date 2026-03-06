> 📖 正在查看搭建手记？[返回项目首页](./README.md)

# OpenClaw 搭建指南

> 这里记录了从零开始搭建OpenClaw的完整过程，供参考

---

## 指南简介

本指南分为8个章节，记录了OpenClaw的完整搭建过程，从基础安装到生产部署。

### 适合谁

- 开发者和高级用户
- 希望拥有个人 AI 助手但注重数据隐私
- 需要多平台 AI 集成的技术团队

### 搭建路径

```
入门 → 基础配置 → 通道集成 → Agent 定制 → 高级功能 → 生产部署 → 项目实战 → 高级主题
```

---

## 章节目录

### 第一章：快速入门

**主要内容**：完成安装并发送第一条 AI 消息

- [认识 OpenClaw](./stages/stage1-stage2/01-what-is-openclaw.md) - 理解核心价值和应用场景
- [环境准备与安装](./stages/stage1-stage2/02-installation.md) - 完成安装和基础配置
- [快速开始](./stages/stage1-stage2/03-quick-start.md) - 发送第一条 AI 消息

### 第二章：基础配置

**主要内容**：掌握配置文件和模型配置

- [配置文件详解](./stages/stage1-stage2/04-configuration.md) - openclaw.json 核心配置
- [模型配置](./stages/stage1-stage2/05-model-configuration.md) - 配置和优化 AI 模型
- [Agent Workspace 配置](./stages/stage1-stage2/06-agent-workspace.md) - 定制 Agent 行为

### 第三章：通道集成

**主要内容**：集成各种消息通道

- [WhatsApp 集成](./stages/stage3-channel/07-whatsapp.md) - 配置最受欢迎的通道
- [Telegram 集成](./stages/stage3-channel/08-telegram.md) - 快速设置最简单的通道
- [Discord 集成](./stages/stage3-channel/09-discord.md) - 集成到 Discord 社区
- [其他通道](./stages/stage3-channel/10-other-channels.md) - iMessage, Slack, Signal 等
- [多通道管理](./stages/stage3-channel/11-multi-channel.md) - 统一管理多个通道

### 第四章：Agent 定制

**主要内容**：深入理解和定制 Agent

- [Agent 深入理解](./stages/stage4-agent/12-agent-loop.md) - 理解 Agent Loop 工作原理
- [工具系统](./stages/stage4-agent/13-tools-system.md) - 掌握核心工具的使用
- [Skills 系统](./stages/stage4-agent/14-skills-system.md) - 扩展 Agent 能力
- [多 Agent 配置](./stages/stage4-agent/15-multi-agent.md) - 配置专门的 Agent

### 第五章：高级功能

**主要内容**：掌握高级功能和扩展

- [沙箱与安全](./stages/stage5-advanced/16-security.md) - 理解并配置安全沙箱
- [插件系统](./stages/stage5-advanced/17-plugins.md) - 通过插件扩展功能
- [Hooks 系统](./stages/stage5-advanced/18-hooks.md) - 使用 Hooks 实现自定义逻辑
- [内存与上下文管理](./stages/stage5-advanced/19-memory.md) - 优化 Agent 内存和上下文
- [流式传输与队列](./stages/stage5-advanced/20-streaming.md) - 理解响应流和队列机制

### 第六章：生产部署

**主要内容**：部署和管理生产环境

- [部署选项](./stages/stage6-deployment/21-deployment.md) - 选择合适的部署方案
- [监控与日志](./stages/stage6-deployment/22-monitoring.md) - 建立完善的监控体系
- [备份与恢复](./stages/stage6-deployment/23-backup.md) - 建立数据保护策略
- [远程访问](./stages/stage6-deployment/24-remote-access.md) - 安全地从外部访问
- [多 Gateway 配置](./stages/stage6-deployment/25-multi-gateway.md) - 管理多个 Gateway 实例

### 第七章：项目实战

**主要内容**：通过实战项目巩固所学

- [实战项目 1：个人 AI 编程助手](./stages/stage7-projects/26-project-coding-assistant.md)
- [实战项目 2：团队知识库机器人](./stages/stage7-projects/27-project-knowledge-base.md)
- [实战项目 3：自动化工作流引擎](./stages/stage7-projects/28-project-workflow.md)
- [实战项目 4：客户服务系统](./stages/stage7-projects/29-project-customer-service.md)

### 第八章：高级主题

**主要内容**：深入探索高级主题

- [性能优化](./stages/stage8-topics/30-performance.md) - 提升系统性能
- [故障排查](./stages/stage8-topics/31-troubleshooting.md) - 快速定位和解决问题
- [扩展开发](./stages/stage8-topics/32-extension-dev.md) - 开发自定义扩展
- [社区与贡献](./stages/stage8-topics/33-community.md) - 参与 OpenClaw 社区

---

## 参考资源

### 官方资源

- [官方文档](https://docs.openclaw.ai/)
- [GitHub 仓库](https://github.com/openclaw/openclaw)
- [社区论坛](https://github.com/openclaw/openclaw/discussions)

---

## 进阶方向

完成基础搭建后，可以选择以下方向深入：

### 方向一：架构师
- 大规模部署设计
- 多租户架构
- 高可用性设计
- 安全架构

### 方向二：开发者
- 自定义 Channel 开发
- Plugin 开发
- 核心代码贡献
- 工具生态扩展

### 方向三：运维专家
- 容器化部署
- 监控体系
- 自动化运维
- 性能调优

### 方向四：解决方案专家
- 行业解决方案
- 定制化服务
- 咨询服务

---

## 开始搭建

建议从 [第一章：快速入门](./stages/stage1-stage2/01-what-is-openclaw.md) 开始。

祝搭建顺利！
