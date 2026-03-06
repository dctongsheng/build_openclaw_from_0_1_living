# 坑点记录

搭建OpenClaw过程中遇到的问题和解决方案汇总。

> 持续更新中...

---

## 安装相关

### Node.js 版本问题

**现象**: 安装时报错 `Unsupported Node.js version`

**原因**: OpenClaw 要求 Node.js 22+，系统安装的是旧版本

**解决方案**:
```bash
# 使用 nvm 安装最新 LTS 版本
nvm install 22
nvm use 22

# 验证版本
node -v  # 应显示 v22.x.x
```

---

### 权限问题

**现象**: 运行 `openclaw init` 时报错 `EACCES: permission denied`

**原因**: 全局安装需要管理员权限

**解决方案**:
```bash
# macOS/Linux
sudo npm install -g openclaw

# 或者使用用户目录安装
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH
```

---

## 配置相关

### API 密钥配置错误

**现象**: 配置后无法调用 AI，返回 401 错误

**原因**: API 密钥格式错误或使用了错误的密钥类型

**解决方案**:
- 确认使用的是正确的 API 密钥（Claude API Key 或 OpenAI API Key）
- 检查配置文件中 `apiKey` 字段格式
- 确保密钥没有多余的空格或换行

---

### 配置文件找不到

**现象**: 报错 `Configuration file not found`

**原因**: 配置文件路径错误或文件不存在

**解决方案**:
```bash
# 查看配置文件位置
openclaw config path

# 重新初始化配置
openclaw init
```

默认配置文件位置：
- macOS: `~/Library/Application Support/openclaw/openclaw.json`
- Linux: `~/.config/openclaw/openclaw.json`
- Windows: `%APPDATA%\openclaw\openclaw.json`

---

## 通道集成相关

### Telegram Bot Token 无效

**现象**: Telegram 通道无法连接

**原因**: Bot Token 格式错误或 Bot 未正确创建

**解决方案**:
1. 在 Telegram 中找到 @BotFather
2. 发送 `/newbot` 创建新 Bot
3. 复收到的 Token 格式应为 `123456:ABC-DEF1234...`
4. 确保复制时没有多余空格

---

### WhatsApp Business API 申请

**现象**: WhatsApp 通道配置后无法使用

**原因**: WhatsApp Business API 需要单独申请，流程较复杂

**解决方案**:
1. 访问 Meta Developers 注册开发者账号
2. 创建 WhatsApp Business App
3. 完成验证流程
4. 获取 Access Token 和 Phone Number ID
5. 在 OpenClaw 配置中填入正确信息

*注意：这个过程可能需要几天时间*

---

## 运行相关

### 端口被占用

**现象**: 启动时报错 `Port 3000 is already in use`

**原因**: 3000 端口已被其他程序占用

**解决方案**:
```bash
# 查找占用端口的进程
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# 终止进程或更改 OpenClaw 端口
```

---

### 内存不足

**现象**: 运行一段时间后进程崩溃

**原因**: 系统内存不足，特别是在处理大量对话时

**解决方案**:
- 增加系统内存
- 限制并发对话数量
- 定期重启 OpenClaw
- 考虑使用更轻量的模型

---

## 更多坑点？

如果你在搭建过程中遇到了其他问题，欢迎：

- 🐛 [提交 Issue](https://github.com/dctongsheng/build_openclaw_from_0_1_living/issues) 报告坑点
- 💡 [分享解决方案](https://github.com/dctongsheng/build_openclaw_from_0_1_living/issues/new/choose) 帮助他人

---

## 🔗 相关链接

- [返回项目首页](./README.md)
- [查看搭建指南](./stages.md)
- [搭建历程](./JOURNEY.md)

---

<div align="center">

**踩坑是成长的阶梯，记录是避坑的地图** 🦞

</div>
