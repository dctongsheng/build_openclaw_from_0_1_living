# 常见问题解答

这里汇总了关于 OpenClaw 从零到一 直播教学项目的常见问题。

---

## 📖 关于课程

### Q: 这个课程收费吗？

**A: 完全免费！**

这个直播教学项目是**完全免费**的，所有内容都会开源到GitHub，永久可访问。我们不接受任何形式的付费，拒绝割韭菜。

### Q: 零基础可以学吗？

**A: 需要一定的基础**。

建议具备以下基础知识：
- 基本的命令行操作能力（Terminal/CMD）
- 了解JSON格式
- 基本的网络概念
- 有使用聊天应用的经验

如果你是完全零基础，建议先学习一下基础命令行操作。

### Q: 需要什么硬件配置？

**A: 最低配置要求**：

- **CPU**: 2核心以上
- **内存**: 4GB以上（推荐8GB）
- **存储**: 20GB可用空间
- **网络**: 稳定的互联网连接

### Q: 支持哪些操作系统？

**A**: macOS、Linux、Windows均可。

OpenClaw是跨平台的，但在macOS和Linux上的体验会更好一些。Windows用户建议使用WSL2。

### Q: 可以回看吗？

**A: 可以！**

每场直播结束后24小时内，回放视频会发布到：
- 站定：GitHub仓库（录播地址）
- B站账号 [待填写]

---

## 🦞 关于OpenClaw

### Q: OpenClaw是什么？

**A**: OpenClaw是一个**自托管的AI Agent Gateway**。

简单来说，它让你能够：
- 在自己的机器上运行AI助手
- 连接15+种聊天平台（WhatsApp、Telegram、Discord等）
- 定制专属的AI Agent行为
- 完全掌控数据隐私

详细说明请查看：[认识OpenClaw](./stages/stage1-stage2/01-what-is-openclaw.md)

### Q: 为什么要用OpenClaw？

**A**: 主要优势：

1. **数据隐私** - 所有数据在本地，不上传第三方
2. **完全掌控** - 想怎么改就怎么改
3. **无厂商锁定** - 开源免费，随时迁移
4. **多平台统一** - 一个网关连接所有聊天平台
5. **高度可定制** - Agent行为、工具、技能全可配

### Q: OpenClaw和ChatGPT/Claude官网有什么区别？

**A**:

| 对比项 | OpenClaw | ChatGPT/Claude官网 |
|--------|----------|-------------------|
| 数据隐私 | 本地存储，完全掌控 | 上传到服务器 |
| 定制能力 | 高度可定制 | 有限定制 |
| 部署方式 | 自托管 | SaaS服务 |
| 费用 | 仅API调用费 | 订阅费+API费 |
| 平台整合 | 统一15+平台 | 单一平台 |

---

## 💰 关于费用

### Q: 使用OpenClau要花钱吗？

**A**: OpenClaw本身免费，但需要支付AI模型API费用。

- OpenClaw软件：完全免费开源
- AI模型API：按使用量计费
  - Claude API: 按token计费
  - OpenAI API: 按token计费

个人使用成本通常在几元到几十元/月，取决于使用频率。

### Q: 如何控制API成本？

**A**:

1. 选择合适的模型（如Haiku比Opus便宜）
2. 设置使用限额
3. 优化提示词减少token消耗
4. 使用缓存机制

---

## 🔧 技术问题

### Q: 安装失败怎么办？

**A**: 常见原因和解决方法：

1. **Node.js版本过低** - 升级到Node.js 22+
2. **权限问题** - 使用sudo或管理员权限
3. **网络问题** - 检查网络连接，尝试使用镜像
4. **端口占用** - 检查3000端口是否被占用

详细排查方法：[故障排查](./stages/stage8-topics/31-troubleshooting.md)

### Q: 配置文件在哪里？

**A**:

- **macOS**: `~/Library/Application Support/openclaw/openclaw.json`
- **Linux**: `~/.config/openclaw/openclaw.json`
- **Windows**: `%APPDATA%\openclaw\openclaw.json`

### Q: 如何启动OpenClaw？

**A**:

```bash
# 开发模式
openclaw dev

# 生产模式
openclaw start
```

详细说明：[快速开始](./stages/stage1-stage2/03-quick-start.md)

### Q: 支持哪些AI模型？

**A**:

- Claude系列（Claude 3 Opus/Sonnet/Haiku）
- OpenAI系列（GPT-4/GPT-3.5）
- 本地模型（通过兼容接口）
- 其他兼容OpenAI协议的模型

---

## 🌐 通道集成

### Q: 支持哪些聊天平台？

**A**: 15+种平台，包括：

- WhatsApp
- Telegram
- Discord
- Slack
- iMessage
- Signal
- 微信（需额外配置）
- 等...

详细列表：[其他通道](./stages/stage3-channel/10-other-channels.md)

### Q: 最简单的通道是哪个？

**A**: **Telegram** 是最简单的。

只需要：
1. 在Telegram中创建Bot
2. 获取Bot Token
3. 在配置文件中添加即可

推荐新手先从Telegram开始。

### Q: 可以同时使用多个通道吗？

**A**: 可以！

OpenClaw支持多通道同时工作，可以：
- 统一的Agent配置
- 独立的通道配置
- 灵活的路由规则

---

## 📺 关于直播

### Q: 错过直播怎么办？

**A**:

1. 观看回放（24小时内发布）
2. 阅读对应教程文档
3. 在Issues中提问

### Q: 直播时可以互动吗？

**A**: 非常欢迎！

- 在直播间弹幕提问
- 在GitHub Issues提前提交问题
- 在微信群中讨论

每场直播预留15-20分钟Q&A时间。

### Q: 直播有回放吗？

**A**: 有的！

每场直播后24小时内发布回放到：
- B站 [待填写]
- GitHub仓库

---

## 🤝 参与方式

### Q: 如何参与学习？

**A**:

1. ⭐ Star本仓库
2. 📺 观看直播或回放
3. 📖 阅读教程文档
4. 💻 跟着动手操作
5. 📝 在Issues中打卡学习进度

### Q: 如何贡献内容？

**A**:

详见 [贡献指南](./CONTRIBUTING.md)

简而言之：
- 修正文档错误
- 补充遗漏内容
- 分享使用案例
- 提交PR

### Q: 如何加入交流群？

**A**:

- 微信群：[待填写二维码]
- GitHub Discussions
- 直播评论区

---

## ❓ 还有问题？

如果在FAQ中没有找到答案：

1. 查看 [完整教程](./stages.md)
2. 查看 [故障排查](./stages/stage8-topics/31-troubleshooting.md)
3. 在 [GitHub Issues](https://github.com/dctongsheng/build_openclaw_from_0_1_living/issues) 提问
4. 在直播时互动提问
5. 在微信交流群中咨询

---

<div align="center">

**没有愚蠢的问题，只有未提出的问题**

**随时欢迎提问！** 🙋

</div>
