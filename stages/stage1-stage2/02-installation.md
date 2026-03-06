# 环境准备与安装

> 学习目标：完成 OpenClaw 的安装和基础配置

## 目录

- [系统要求](#系统要求)
- [安装方式选择](#安装方式选择)
- [方式一：安装脚本（推荐）](#方式一安装脚本推荐)
- [方式二：npm/pnpm 安装](#方式二npmpnpm-安装)
- [方式三：从源码构建](#方式三从源码构建)
- [方式四：Docker 部署](#方式四docker-部署)
- [验证安装](#验证安装)
- [常见问题](#常见问题)
- [小结](#小结)

---

## 系统要求

### 必需环境

| 组件 | 最低版本 | 推荐版本 | 检查命令 |
|-----|---------|---------|---------|
| **Node.js** | 22.0.0 | 最新 LTS | `node --version` |
| **npm** | 10.0.0 | 最新版 | `npm --version` |
| **操作系统** | 见下表 | - | - |

### 支持的操作系统

| 操作系统 | 支持状态 | 说明 |
|---------|---------|-----|
| **macOS** | ✅ 完全支持 | 11+ (Big Sur 及以上) |
| **Linux** | ✅ 完全支持 | Ubuntu 20.04+, Debian 11+, CentOS 8+ |
| **Windows** | ✅ 完全支持 | Windows 10/11 with WSL2 推荐 |

### 环境检查

在安装前，请检查你的环境：

```bash
# 检查 Node 版本（需要 22+）
node --version

# 检查 npm 版本
npm --version

# 检查操作系统
uname -a  # Linux/macOS
systeminfo | findstr /B /C:"OS"  # Windows

# 检查可用磁盘空间（建议至少 1GB）
df -h  # Linux/macOS
```

**如果 Node 版本低于 22**，请先升级：

```bash
# 使用 nvm（推荐）
nvm install 22
nvm use 22

# 或使用 n（npm）
npm install -g n
sudo n 22

# macOS (Homebrew)
brew install node@22

# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

---

## 安装方式选择

OpenClaw 提供多种安装方式，请根据你的需求选择：

| 安装方式 | 难度 | 适用场景 | 推荐度 |
|---------|-----|---------|-------|
| **安装脚本** | ⭐ 简单 | 大多数用户 | ⭐⭐⭐⭐⭐ |
| **npm/pnpm** | ⭐⭐ 中等 | Node.js 开发者 | ⭐⭐⭐⭐ |
| **从源码构建** | ⭐⭐⭐ 复杂 | 需要修改源码 | ⭐⭐⭐ |
| **Docker** | ⭐⭐ 中等 | 容器化部署 | ⭐⭐⭐⭐ |

---

## 方式一：安装脚本（推荐）

这是最简单的安装方式，适合大多数用户。

### macOS / Linux

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Windows (PowerShell)

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### 安装脚本做了什么？

1. **检查环境** - 验证 Node 版本
2. **下载安装** - 从 npm 获取最新版本
3. **配置路径** - 添加到系统 PATH
4. **初始化配置** - 创建基础配置文件
5. **安装 Daemon** - 可选，用于后台运行

### 自定义安装选项

```bash
# 安装到指定目录
curl -fsSL https://openclaw.ai/install.sh | INSTALL_DIR=/opt/openclaw bash

# 不安装 daemon
curl -fsSL https://openclaw.ai/install.sh | SKIP_DAEMON=1 bash

# 安装特定版本
curl -fsSL https://openclaw.ai/install.sh | OPENCLAW_VERSION=1.2.3 bash
```

---

## 方式二：npm/pnpm 安装

如果你熟悉 Node.js，可以直接使用 npm 或 pnpm 安装。

### 使用 npm

```bash
# 全局安装
npm install -g @openclaw/gateway

# 安装特定版本
npm install -g @openclaw/gateway@1.2.3

# 查看安装位置
npm root -g
npm bin -g
```

### 使用 pnpm（推荐）

```bash
# 全局安装
pnpm add -g @openclaw/gateway

# 安装特定版本
pnpm add -g @openclaw/gateway@1.2.3
```

### 验证 PATH 配置

安装后，确保 `openclaw` 命令在 PATH 中：

```bash
# 检查命令是否可用
which openclaw

# 如果不可用，添加到 PATH
export PATH="$PATH:$(npm bin -g)"
# 或
export PATH="$PATH:$(pnpm bin -g)"

# 添加到 shell 配置文件（永久生效）
echo 'export PATH="$PATH:'$(npm bin -g)'"' >> ~/.bashrc
echo 'export PATH="$PATH:'$(npm bin -g)'"' >> ~/.zshrc
source ~/.bashrc  # 或 source ~/.zshrc
```

---

## 方式三：从源码构建

适合需要修改源码或贡献代码的开发者。

### 前置要求

- Node.js 22+
- Git
- pnpm（推荐）或 npm

### 构建步骤

```bash
# 1. 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 2. 安装依赖
pnpm install

# 3. 构建
pnpm build

# 4. 链接全局命令
pnpm link --global

# 5. 验证
openclaw --version
```

### 开发模式

```bash
# 运行开发服务器
pnpm dev

# 运行测试
pnpm test

# 代码检查
pnpm lint
```

---

## 方式四：Docker 部署

适合容器化部署和隔离环境。

### 使用 Docker Hub 镜像

```bash
# 拉取最新镜像
docker pull openclaw/gateway:latest

# 运行容器
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/.openclaw:/root/.openclaw \
  openclaw/gateway:latest

# 查看日志
docker logs -f openclaw
```

### 使用 Docker Compose（推荐）

创建 `docker-compose.yml`:

```yaml
version: '3.8'

services:
  openclaw:
    image: openclaw/gateway:latest
    container_name: openclaw
    restart: unless-stopped
    ports:
      - "18789:18789"
    volumes:
      - ./openclaw-data:/root/.openclaw
      - /var/run/docker.sock:/var/run/docker.sock  # 如果需要使用 Docker 工具
    environment:
      - NODE_ENV=production
      - OPENCLAW_LOG_LEVEL=info
```

启动服务：

```bash
# 启动
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 停止
docker-compose down
```

### 构建自定义镜像

```dockerfile
FROM node:22-alpine

# 安装 pnpm
RUN npm install -g pnpm

# 设置工作目录
WORKDIR /app

# 复制 package 文件
COPY package.json pnpm-lock.yaml ./

# 安装依赖
RUN pnpm install --frozen-lockfile

# 复制源码
COPY . .

# 构建
RUN pnpm build

# 暴露端口
EXPOSE 18789

# 启动
CMD ["pnpm", "start"]
```

构建和运行：

```bash
# 构建镜像
docker build -t my-openclaw .

# 运行
docker run -d \
  --name my-openclaw \
  -p 18789:18789 \
  -v ~/.openclaw:/root/.openclaw \
  my-openclaw
```

---

## 验证安装

安装完成后，运行以下命令验证：

### 1. 版本检查

```bash
openclaw --version
# 输出: openclaw/1.x.x
```

### 2. 健康检查

```bash
# 综合诊断
openclaw doctor

# 输出示例:
# ✓ Node.js version: 22.x.x
# ✓ Platform: darwin/arm64
# ✓ Installation: /usr/local/lib/node_modules/@openclaw/gateway
# ✓ Config file: /Users/username/.openclaw/openclaw.json
# ✓ Workspace: /Users/username/.openclaw/workspace
# ! API keys: No API keys configured
```

### 3. 查看状态

```bash
# 查看 Gateway 状态
openclaw status

# 输出示例:
# Gateway: running (PID: 12345)
# Uptime: 2 hours 30 minutes
# Channels: whatsapp, telegram
# Active sessions: 5
```

### 4. 测试命令帮助

```bash
# 查看所有可用命令
openclaw --help

# 查看特定命令帮助
openclaw message --help
openclaw agent --help
```

---

## 常见问题

### Q1: 安装后提示命令找不到

**A**: 检查 PATH 配置：

```bash
# 临时添加
export PATH="$PATH:$(npm bin -g)"

# 永久添加到 shell 配置
echo 'export PATH="$PATH:'$(npm bin -g)'"' >> ~/.zshrc
source ~/.zshrc
```

### Q2: Node 版本不兼容

**A**: 升级 Node.js 到 22+：

```bash
# 使用 nvm
nvm install 22
nvm use 22

# 或使用 n
sudo n 22
```

### Q3: macOS 提示无法验证开发者

**A**: 允许来自未知开发者的应用：

```bash
# 系统偏好设置 -> 安全性与隐私 -> 通用
# 点击"仍要打开"
```

或卸载后重新安装：

```bash
sudo npm uninstall -g @openclaw/gateway
npm install -g @openclaw/gateway
```

### Q4: 权限问题

**A**: 使用 sudo 或修复权限：

```bash
# 使用 sudo 安装
sudo npm install -g @openclaw/gateway

# 或修复 npm 目录权限
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Q5: Docker 容器无法访问宿主机文件

**A**: 正确挂载卷：

```bash
docker run -v ~/.openclaw:/root/.openclaw ...
```

---

## 小结

本节我们学习了：

1. ✅ 系统要求：Node 22+, 支持主流操作系统
2. ✅ 四种安装方式：
   - 安装脚本（推荐，最简单）
   - npm/pnpm（适合 Node.js 开发者）
   - 从源码构建（适合开发者）
   - Docker（适合容器化部署）
3. ✅ 验证安装：`openclaw doctor` 和 `openclaw status`
4. ✅ 常见问题解决方案

## 下一步

[快速开始](./03-quick-start.md) - 发送你的第一条 AI 消息
