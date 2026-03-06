# 远程访问

> 学习目标：安全地从外部访问

## 目录

- [远程访问概述](#远程访问概述)
- [Tailscale](#tailscale)
- [VPN](#vpn)
- [反向代理](#反向代理)
- [隧道服务](#隧道服务)
- [安全配置](#安全配置)
- [方案对比](#方案对比)
- [小结](#小结)

---

## 远程访问概述

### 访问方式

```
远程访问方式:
├── Tailscale (推荐)
│   └── 零配置 Mesh VPN
├── VPN
│   └── 传统 VPN 方案
├── 反向代理
│   └── Nginx/Caddy
└── 隧道服务
    └── ngrok/frp
```

### 安全考虑

```
安全措施:
├── 加密传输 (TLS)
├── 身份验证
├── 访问控制
├── 速率限制
└── 审计日志
```

---

## Tailscale

### 为什么选择 Tailscale？

```
优点:
✅ 零配置
✅ 点对点加密
✅ 自动 NAT 穿透
✅ 内网穿透
✅ 访问控制

缺点:
❌ 需要第三方服务
❌ 依赖 Tailscale 网络
```

### 安装 Tailscale

```bash
# macOS
brew install --cask tailscale

# Linux
curl -fsSL https://tailscale.com/install.sh | sh

# 启动 Tailscale
sudo tailscale up

# 登录
# 按照提示在浏览器中登录
```

### 配置 ACL

```json
// Tailscale ACL
{
  "Groups": {
    "group:admin": ["user@example.com"],
    "group:users": ["user1@example.com", "user2@example.com"]
  },
  "ACLs": [
    {
      "action": "accept",
      "src": ["group:admin"],
      "dst": ["*:*"]
    },
    {
      "action": "accept",
      "src": ["group:users"],
      "dst": ["tag:openclaw:*"]
    }
  ],
  "Tags": {
    "tag:openclaw": ["openclaw-server"]
  }
}
```

### 使用 Tailscale

```bash
# 服务器端
# 1. 安装并启动 Tailscale
sudo tailscale up

# 2. 分配标签
sudo tailscale tag apply openclaw-server

# 3. 记录 Tailscale IP
tailscale ip -4

# 客户端
# 1. 连接到同一 Tailscale 网络
sudo tailscale up

# 2. 访问 OpenClaw
curl http://100.x.x.x:18789/health
```

---

## VPN

### WireGuard VPN

#### 服务器配置

```ini
# /etc/wireguard/wg0.conf
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
# 客户端 1
PublicKey = <CLIENT1_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32

[Peer]
# 客户端 2
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.0.0.3/32
```

#### 客户端配置

```ini
# wg0.conf
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = your-server.com:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

#### 启动 WireGuard

```bash
# 服务器
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0

# 客户端
sudo wg-quick up wg0

# 验证连接
ping 10.0.0.1
curl http://10.0.0.1:18789/health
```

### OpenVPN

```bash
# 安装 OpenVPN
sudo apt install openvpn

# 生成证书
./easyrsa build-ca
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1 nopass

# 配置服务器
# /etc/openvpn/server.conf
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
keepalive 10 120

# 启动 OpenVPN
sudo systemctl start openvpn@server
```

---

## 反向代理

### Nginx 配置

```nginx
# /etc/nginx/sites-available/openclaw
server {
    listen 80;
    server_name your-domain.com;

    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL 证书
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # SSL 配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256...';

    # 安全头
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    # 反向代理
    location / {
        proxy_pass http://localhost:18789;
        proxy_http_version 1.1;

        # WebSocket 支持
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        # 其他头
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # 基本身份验证 (可选)
    # auth_basic "Restricted";
    # auth_basic_user_file /etc/nginx/.htpasswd;
}
```

### Caddy 配置

```caddyfile
# Caddyfile
your-domain.com {
    # 自动 HTTPS
    encode gzip

    # 反向代理
    reverse_proxy localhost:18789 {
        # WebSocket 支持
        header_up X-Forwarded-Host {host}
        header_up X-Forwarded-Proto {scheme}
    }

    # 访问控制 (可选)
    basicauth /admin {
        user $2p rounds 10000
    }
}
```

---

## 隧道服务

### ngrok

```bash
# 安装 ngrok
brew install ngrok  # macOS
# 或从 https://ngrok.com 下载

# 启动隧道
ngrok http 18789

# 输出:
# Forwarding https://abc123.ngrok.io -> http://localhost:18789

# 配置自定义域名 (需要付费)
ngrok http -hostname=custom.ngrok.io 18789
```

### frp (Fast Reverse Proxy)

#### 服务器端 (frps)

```ini
# frps.ini
[common]
bind_port = 7000
vhost_http_port = 80
vhost_https_port = 443
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = password
token = your_token
```

#### 客户端 (frpc)

```ini
# frpc.ini
[common]
server_addr = your-server.com
server_port = 7000
token = your_token

[web]
type = http
local_ip = 127.0.0.1
local_port = 18789
custom_domains = your-domain.com
```

---

## 安全配置

### 访问控制

```json
{
  "accessControl": {
    "enabled": true,
    "rules": [
      {
        "source": "0.0.0.0/0",
        "action": "block",
        "reason": "默认阻止所有"
      },
      {
        "source": "10.0.0.0/8",
        "action": "allow",
        "reason": "允许 VPN"
      },
      {
        "source": "100.64.0.0/10",
        "action": "allow",
        "reason": "允许 Tailscale"
      }
    ]
  }
}
```

### 速率限制

```nginx
# 在 Nginx 中配置
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

location /api {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://localhost:18789;
}
```

### IP 白名单

```nginx
# 只允许特定 IP
allow 1.2.3.4;
allow 5.6.7.8;
deny all;
```

---

## 方案对比

| 方案 | 难度 | 安全性 | 成本 | 推荐度 |
|-----|------|--------|------|-------|
| **Tailscale** | ⭐ 简单 | ⭐⭐⭐⭐⭐ | 免费/付费 | ⭐⭐⭐⭐⭐ |
| **WireGuard** | ⭐⭐ 中等 | ⭐⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ |
| **OpenVPN** | ⭐⭐⭐ 复杂 | ⭐⭐⭐⭐ | 免费 | ⭐⭐⭐ |
| **Nginx** | ⭐⭐ 中等 | ⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ |
| **Caddy** | ⭐ 简单 | ⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐⭐ |
| **ngrok** | ⭐ 简单 | ⭐⭐⭐ | 免费/付费 | ⭐⭐⭐ |
| **frp** | ⭐⭐ 中等 | ⭐⭐⭐⭐ | 免费 | ⭐⭐⭐⭐ |

---

## 小结

本节我们学习了：

1. ✅ 远程访问概述 - 方式和安全考虑
2. ✅ Tailscale - 零配置 VPN
3. ✅ VPN - WireGuard 和 OpenVPN
4. ✅ 反向代理 - Nginx 和 Caddy
5. ✅ 隧道服务 - ngrok 和 frp
6. ✅ 安全配置 - 访问控制和速率限制
7. ✅ 方案对比 - 各方案对比

## 选择建议

```
个人使用:
→ Tailscale (最简单)

小型团队:
→ Tailscale 或 WireGuard

生产环境:
→ Nginx/Caddy + TLS

快速测试:
→ ngrok 或 frp
```

## 下一步

[多 Gateway 配置](./25-multi-gateway.md) - 管理多个 Gateway 实例
