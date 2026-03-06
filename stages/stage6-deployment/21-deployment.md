# 部署选项

> 学习目标：选择合适的部署方案

## 目录

- [部署场景分析](#部署场景分析)
- [本地开发机器](#本地开发机器)
- [家庭服务器](#家庭服务器)
- [VPS 云主机](#vps-云主机)
- [容器化部署](#容器化部署)
- [Kubernetes 集群](#kubernetes-集群)
- [部署对比](#部署对比)
- [选择指南](#选择指南)
- [小结](#小结)

---

## 部署场景分析

### 需求评估矩阵

```
部署场景:
├── 个人使用 → 本地机器
├── 小团队 → 家庭服务器/VPS
├── 中型团队 → VPS/容器
└── 企业级 → Kubernetes/云服务
```

### 评估因素

| 因素 | 权重 | 说明 |
|-----|------|------|
| **成本** | ⭐⭐⭐⭐⭐ | 硬件、软件、维护成本 |
| **性能** | ⭐⭐⭐⭐ | 响应时间、并发能力 |
| **可靠性** | ⭐⭐⭐⭐ | 可用性、故障恢复 |
| **可扩展性** | ⭐⭐⭐ | 水平/垂直扩展能力 |
| **安全性** | ⭐⭐⭐⭐⭐ | 数据保护、访问控制 |
| **维护性** | ⭐⭐⭐ | 部署、更新、监控 |

---

## 本地开发机器

### 适用场景

- ✅ 个人开发测试
- ✅ 学习和实验
- ✅ 本地服务集成
- ❌ 生产环境

### 优缺点

```
优点:
✅ 成本低（已有硬件）
✅ 完全控制
✅ 快速迭代
✅ 数据本地化

缺点:
❌ 不可靠（电脑关机）
❌ 不可访问（离开时）
❌ 性能限制
❌ 安全风险
```

### 配置建议

```bash
# 系统要求
CPU: 4 核心以上
内存: 8GB 以上
存储: 50GB 以上
网络: 稳定的互联网连接

# 推荐配置
CPU: 8 核心
内存: 16GB
存储: 500GB SSD
```

### 部署步骤

```bash
# 1. 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 2. 配置自动启动
openclaw onboard --install-daemon

# 3. 配置端口转发（可选）
# 使用 ngrok 或类似服务
ngrok http 18789

# 4. 验证
openclaw status
openclaw doctor
```

---

## 家庭服务器

### 适用场景

- ✅ 个人长期使用
- ✅ 家庭成员共享
- ✅ 小项目托管
- ⚠️ 需要一定技术能力

### 优缺点

```
优点:
✅ 低成本（一次投入）
✅ 完全控制
✅ 数据隐私
✅ 可扩展

缺点:
❌ 需要维护
❌ 电力成本
❌ 网络依赖
❌ 物理空间
```

### 硬件选择

#### 树莓派

```
型号: Raspberry Pi 4 或 5
CPU: 4 核心 ARM Cortex
内存: 4GB 或 8GB
存储: 128GB SD 卡
成本: 约 $50-80
```

#### 迷你 PC

```
型号: Intel NUC, Mini PC
CPU: Intel i5 或 i7
内存: 16GB
存储: 512GB SSD
成本: 约 $300-500
```

#### 旧电脑

```
利用旧电脑或笔记本
确保:
- 功耗合理
- 散热良好
- 静音运行
```

### 网络配置

#### 端口转发

```bash
# 路由器配置
外部端口: 443 (HTTPS)
内部端口: 18789
内部 IP: 192.168.1.100 (服务器 IP)

# 或使用 DMZ (不推荐)
```

#### 动态 DNS

```bash
# 使用 ddclient 或类似服务
# 配置动态 DNS 服务

# 安装 ddclient
sudo apt install ddclient

# 配置文件
/etc/ddclient.conf:
protocol=dyndns2
use=web, web=http://checkip.dyndns.org
server=members.dyndns.org
login=your-username
password=your-password
your-domain.dyndns.org
```

### 部署步骤

```bash
# 1. 设置操作系统
# Ubuntu Server 推荐
sudo apt update && sudo apt upgrade -y

# 2. 安装 Node.js
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 3. 安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 4. 配置自动启动
sudo systemctl enable openclaw
sudo systemctl start openclaw

# 5. 配置防火墙
sudo ufw allow 443/tcp
sudo ufw enable

# 6. 配置 HTTPS
# 使用 Let's Encrypt 和 Certbot
sudo apt install certbot
sudo certbot certonly --standalone -d your-domain.com

# 7. 配置反向代理 (Nginx)
sudo apt install nginx
sudo nano /etc/nginx/sites-available/openclaw
```

---

## VPS 云主机

### 适用场景

- ✅ 小型团队
- ✅ 需要外部访问
- ✅ 生产环境
- ✅ 可靠性要求

### 优缺点

```
优点:
✅ 高可用性
✅ 专业维护
✅ 快速部署
✅ 可扩展

缺点:
❌ 持续成本
❌ 数据隐私
❌ 依赖服务商
❌ 网络延迟
```

### VPS 提供商

| 提供商 | 起步价格 | 特点 |
|--------|---------|------|
| **DigitalOcean** | $6/月 | 简单易用 |
| **Linode** | $5/月 | 性能好 |
| **Vultr** | $5/月 | 全球节点 |
| **AWS Lightsail** | $3.5/月 | AWS 生态 |
| **Google Cloud** | $6/月 | GCP 集成 |
| **阿里云** | ¥60/月 | 国内优化 |

### 推荐配置

```
最小配置:
CPU: 1 核心
内存: 1GB
存储: 25GB SSD
带宽: 1TB
成本: $5-6/月

推荐配置:
CPU: 2 核心
内存: 4GB
存储: 80GB SSD
带宽: 4TB
成本: $20-24/月

高性能配置:
CPU: 4 核心
内存: 8GB
存储: 160GB SSD
带宽: 5TB
成本: $40-48/月
```

### 部署步骤

```bash
# 1. 创建 VPS 实例
# 在提供商控制台创建

# 2. SSH 连接
ssh root@your-vps-ip

# 3. 更新系统
apt update && apt upgrade -y

# 4. 安装 Docker (推荐)
curl -fsSL https://get.docker.com | sh

# 5. 运行 OpenClaw
docker run -d \
  --name openclaw \
  --restart unless-stopped \
  -p 18789:18789 \
  -v openclaw-data:/root/.openclaw \
  openclaw/gateway:latest

# 6. 配置域名和 HTTPS
# 使用 Cloudflare 或 Let's Encrypt

# 7. 监控和维护
docker logs -f openclaw
docker stats openclaw
```

---

## 容器化部署

### Docker 部署

#### Dockerfile

```dockerfile
FROM node:22-alpine

# 安装依赖
RUN npm install -g @openclaw/gateway

# 创建数据目录
RUN mkdir -p /root/.openclaw

# 复制配置
COPY openclaw.json /root/.openclaw/

# 暴露端口
EXPOSE 18789

# 启动
CMD ["openclaw", "start", "--foreground"]
```

#### Docker Compose

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
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - NODE_ENV=production
      - OPENCLAW_LOG_LEVEL=info
    networks:
      - openclaw-network

  nginx:
    image: nginx:alpine
    container_name: openclaw-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - openclaw
    networks:
      - openclaw-network

networks:
  openclaw-network:
    driver: bridge
```

#### 部署命令

```bash
# 构建镜像
docker build -t openclaw-custom .

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 更新服务
docker-compose pull
docker-compose up -d

# 备份数据
docker run --rm \
  -v openclaw-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/openclaw-backup.tar.gz /data
```

---

## Kubernetes 集群

### 适用场景

- ✅ 大规模部署
- ✅ 高可用性要求
- ✅ 复杂架构
- ✅ 企业环境

### Kubernetes 清单

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openclaw
  labels:
    app: openclaw
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openclaw
  template:
    metadata:
      labels:
        app: openclaw
    spec:
      containers:
      - name: openclaw
        image: openclaw/gateway:latest
        ports:
        - containerPort: 18789
        env:
        - name: NODE_ENV
          value: "production"
        - name: OPENCLAW_LOG_LEVEL
          value: "info"
        volumeMounts:
        - name: openclaw-data
          mountPath: /root/.openclaw
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 18789
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 18789
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: openclaw-data
        persistentVolumeClaim:
          claimName: openclaw-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: openclaw
spec:
  selector:
    app: openclaw
  ports:
  - protocol: TCP
    port: 80
    targetPort: 18789
  type: LoadBalancer

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: openclaw-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### 部署命令

```bash
# 部署
kubectl apply -f deployment.yaml

# 扩展
kubectl scale deployment openclaw --replicas=5

# 更新
kubectl set image deployment/openclaw \
  openclaw=openclaw/gateway:v1.2.3

# 回滚
kubectl rollout undo deployment/openclaw

# 查看状态
kubectl get pods
kubectl logs -f deployment/openclaw
```

---

## 部署对比

| 方案 | 成本 | 性能 | 可靠性 | 可维护性 | 推荐度 |
|-----|------|------|--------|----------|-------|
| **本地机器** | $ | ⭐⭐ | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **家庭服务器** | $$ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **VPS** | $$$ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Docker** | $$$ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Kubernetes** | $$$$ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 选择指南

### 决策树

```
开始
  ↓
是否需要外部访问?
  ├─ 否 → 本地机器
  └─ 是 → 是否需要 24/7 可用?
           ├─ 否 → 家庭服务器
           └─ 是 → 用户规模?
                    ├─ < 10 用户 → VPS
                    ├─ 10-100 用户 → VPS + Docker
                    └─ > 100 用户 → Kubernetes
```

### 场景推荐

```
个人学习:
→ 本地机器

个人项目:
→ 家庭服务器 或 VPS (基本)

小型团队 (< 10 人):
→ VPS (推荐配置)

中型团队 (10-50 人):
→ VPS + Docker Compose

大型团队 (50+ 人):
→ Kubernetes + 云服务

企业级:
→ Kubernetes + 多云部署
```

---

## 小结

本节我们学习了：

1. ✅ 部署场景分析 - 需求评估和因素
2. ✅ 本地开发机器 - 优缺点和配置
3. ✅ 家庭服务器 - 硬件选择和网络配置
4. ✅ VPS 云主机 - 提供商和部署
5. ✅ 容器化部署 - Docker 和 Compose
6. ✅ Kubernetes 集群 - 清单和部署
7. ✅ 部署对比 - 各方案对比
8. ✅ 选择指南 - 决策树和场景推荐

## 部署检查清单

- [ ] 确定部署场景
- [ ] 选择部署方案
- [ ] 准备硬件/云资源
- [ ] 配置网络和域名
- [ ] 设置 HTTPS
- [ ] 配置防火墙
- [ ] 设置监控
- [ ] 配置备份
- [ ] 测试部署
- [ ] 文档化配置

## 下一步

[监控与日志](./22-monitoring.md) - 建立完善的监控体系
