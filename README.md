# 🦞 OpenClaw 搭建手记

> 一个人、一台Mac mini、一场关于自托管AI助手的探索之旅

---

## 📖 缘起

2026年3月，一个Mac mini的订单，开启了一场新的探索...

**这一切是怎么开始的？**

最近越来越多的人问我："小龙虾怎么安装？"、"我的小龙虾配置好后怎么用？"

问题来得多了，我就在想：为什么不把整个搭建过程记录下来呢？一方面自己以后可以回顾，另一方面也方便有需要的朋友参考。

正好，我刚下单了一台Mac mini，专门用来跑这些自托管服务。于是，我开始了这场**OpenClaw的搭建探索之旅**。

---

## 🤔 什么是OpenClaw？

OpenClaw（我们亲切地称它为"小龙虾"）是一个**自托管的AI Agent Gateway**。简单来说，它让你能够：

- 🏠 **在自己的机器上运行AI助手** - 数据隐私完全掌控
- 🌐 **连接15+种聊天平台** - WhatsApp、Telegram、Discord...
- 🤖 **定制专属AI Agent** - 工具、技能、行为全可配
- 👥 **多 Agent 协作系统** - 配置多个专业化 agent 协同工作 ✨
- 🔓 **完全开源免费** - 没有厂商锁定，想怎么改就怎么改

**当前配置：**
- 🦐 **小虾米（主 agent）** - 全能协调者，使用 GLM-4.7
- 📚 **研究助手** - 信息收集、数据分析专家
- 🌟 **生活助手** - 日常对话、生活建议顾问

---

## 🛠️ 搭建进度

### 已完成

- [x] Mac mini 到货并配置
- [x] OpenClaw 基础安装
- [x] 第一个AI消息发送成功
- [x] Telegram 和 Feishu 通道配置
- [x] **多 Agent 系统配置**（2026年3月7日）
  - [x] 主 agent（小虾米）
  - [x] 研究助手
  - [x] 生活助手
  - [x] Agent 间通信协作

### 进行中

- [ ] Agent 定制化优化
- [ ] 技能系统扩展

### 计划中

- [ ] WhatsApp 通道配置
- [ ] 生产环境部署
- [ ] 实战项目

> 无固定时间表，慢慢探索，记录点滴

---

## 🕳️ 坑点记录

搭建过程中遇到的问题和解决方案，都在这里记录：

- [安装相关](./PITFALLS.md#安装相关) - Node.js版本、权限问题等
- [配置相关](./PITFALLS.md#配置相关) - API配置、模型选择等
- [通道集成](./PITFALLS.md#通道集成相关) - 各平台配置的坑

> 更多坑点持续更新中...

---

## 📝 经验分享

### 关于自托管AI

搭建OpenClaw的过程让我重新思考了数据隐私的重要性。把AI助手跑在自己的机器上，所有聊天记录、配置信息都在本地掌控，这种安全感是云端服务无法替代的。

### 关于踩坑

完美的教程很多，但真实的搭建过程往往充满各种坑。我选择把这些坑都记录下来，不隐藏任何问题，因为这才是真实的过程。

### 关于节奏

没有"Day 1-3"的时间表，没有"打卡"的压力。想折腾就折腾，累了就停一停。这才是个人项目的乐趣。

---

## 📚 参考文档

这里记录了完整的搭建过程，供参考：

### 核心文档
- [📖 搭建指南](./stages.md) - 从零开始搭建 OpenClaw
- [🚀 多 Agent 配置完整指南](./multi-agent-setup-guide.md) - **[NEW]** 配置多个专业化 agent
- [⚡ 多 Agent 快速参考](./multi-agent-quick-reference.md) - **[NEW]** 快速查阅多 agent 使用

### 经验分享
- [🕳️ 坑点记录](./PITFALLS.md) - 安装、配置、集成的各种坑
- [❓ 常见问题](./FAQ.md) - 问题排查和解决方案
- [📝 探索之旅](./JOURNEY.md) - 整个探索过程的思考

---

## 🤝 共建邀请

如果你也在搭建OpenClaw，欢迎：

- 🐛 报告你遇到的坑
- 💡 分享你的解决方案
- 📝 补充遗漏的内容
- 🌟 分享你的使用案例

详见 [共建指南](./CONTRIBUTING.md)

---

## 🔗 相关链接

- 📖 [OpenClaw官方文档](https://docs.openclaw.ai/)
- 🐙 [OpenClaw GitHub仓库](https://github.com/openclaw/openclaw)
- 💬 [OpenClaw社区论坛](https://github.com/openclaw/openclaw/discussions)
- 📮 [问题反馈](https://github.com/dctongsheng/build_openclaw_from_0_1_living/issues)

---

## 📜 许可证

本项目采用 [CC BY-NC-SA 4.0](./LICENSE) 许可证

- ✅ 允许：分享、修改、用于学习
- ❌ 禁止：商业化使用

---

## 💖 致谢

- **OpenClaw开源项目** - 提供了如此优秀的AI Agent Gateway
- **所有贡献者** - 每一个PR、Issue都让这个项目更好

---

<div align="center">

**记录探索过程，分享踩坑经验**

**开源共建 · 轻松记录 · 持续更新**

[⭐ Star](https://github.com/dctongsheng/build_openclaw_from_0_1_living) |
[🍴 Fork](https://github.com/dctongsheng/build_openclaw_from_0_1_living/fork)

</div>
