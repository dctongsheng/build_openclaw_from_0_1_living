# 多 Gateway 配置

> 学习目标：管理多个 Gateway 实例

## 目录

- [多 Gateway 架构](#多-gateway-架构)
- [Gateway 发现](#gateway-发现)
- [负载均衡](#负载均衡)
- [故障转移](#故障转移)
- [配置同步](#配置同步)
- [集群管理](#集群管理)
- [小结](#小结)

---

## 多 Gateway 架构

### 架构模式

```
模式 1: 主备 (Primary-Backup)
├── 主 Gateway: 处理所有请求
└── 备 Gateway: 故障时接管

模式 2: 负载均衡 (Load Balancing)
├── 多个 Gateway: 分担负载
└── 负载均衡器: 分发请求

模式 3: 地理分布 (Geo-Distributed)
├── 区域 Gateway: 服务本地用户
└── 全局负载均衡: 智能路由
```

### 架构选择

```
选择标准:
├── 可靠性要求 → 主备模式
├── 性能要求 → 负载均衡
├── 全球用户 → 地理分布
└── 复杂场景 → 混合模式
```

---

## Gateway 发现

### 服务发现配置

```json
{
  "discovery": {
    "enabled": true,
    "method": "consul",  // consul, etcd, redis
    "config": {
      "consul": {
        "address": "localhost:8500",
        "token": "${CONSUL_TOKEN}",
        "service": "openclaw",
        "tags": ["gateway", "api"],
        "check": {
          "interval": "10s",
          "timeout": "5s",
          "http": "http://localhost:18789/health"
        }
      }
    }
  }
}
```

### 注册 Gateway

```bash
# 注册到 Consul
curl -X PUT http://localhost:8500/v1/agent/service/register \
  -d '{
    "ID": "openclaw-1",
    "Name": "openclaw",
    "Tags": ["gateway", "api"],
    "Address": "192.168.1.10",
    "Port": 18789,
    "Check": {
      "HTTP": "http://192.168.1.10:18789/health",
      "Interval": "10s"
    }
  }'

# 查看服务
curl http://localhost:8500/v1/health/service/openclaw
```

### 健康检查

```json
{
  "healthCheck": {
    "enabled": true,
    "endpoint": "/health",
    "interval": 10,
    "timeout": 5,
    "unhealthyThreshold": 3,
    "healthyThreshold": 2
  }
}
```

---

## 负载均衡

### HAProxy 配置

```
# haproxy.cfg
defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend openclaw_front
    bind *:80
    default_backend openclaw_back

backend openclaw_back
    balance roundrobin
    option httpchk GET /health
    server gateway1 192.168.1.10:18789 check
    server gateway2 192.168.1.11:18789 check
    server gateway3 192.168.1.12:18789 check

    # 会话保持
    cookie SRVID insert indirect nocache

    # 故障转移
    option redispatch
```

### Nginx 负载均衡

```nginx
upstream openclaw_backend {
    # 负载均衡算法
    least_conn;

    # Gateway 服务器
    server gateway1:18789 weight=1 max_fails=3 fail_timeout=30s;
    server gateway2:18789 weight=1 max_fails=3 fail_timeout=30s;
    server gateway3:18789 weight=1 max_fails=3 fail_timeout=30s;

    # 会话保持
    ip_hash;

    # 健康检查
    check interval=30s rise=2 fall=3;
}

server {
    listen 80;

    location / {
        proxy_pass http://openclaw_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Traefik 配置

```yaml
# traefik.yml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

providers:
  consul:
    endpoints:
      - "consul:8500"
    exposedByDefault: false

  file:
    filename: /etc/traefik/dynamic.yml

# dynamic.yml
http:
  routers:
    openclaw:
      rule: "Host(`openclaw.example.com`)"
      service: "openclaw"
      entryPoints:
        - "websecure"
      tls:
        certResolver: "letsencrypt"

  services:
    openclaw:
      loadBalancer:
        servers:
          - url: "http://gateway1:18789"
          - url: "http://gateway2:18789"
          - url: "http://gateway3:18789"
        healthCheck:
          path: "/health"
          interval: "10s"
```

---

## 故障转移

### 主备模式

```json
{
  "ha": {
    "mode": "primary-backup",
    "primary": {
      "id": "gateway-1",
      "address": "192.168.1.10",
      "port": 18789
    },
    "backup": {
      "id": "gateway-2",
      "address": "192.168.1.11",
      "port": 18789,
      "activation": "automatic",
      "healthCheck": {
        "interval": 5,
        "failures": 3
      }
    }
  }
}
```

### 自动故障转移脚本

```bash
#!/bin/bash
# failover.sh

PRIMARY="gateway-1"
BACKUP="gateway-2"
HEALTH_URL="http://localhost:18789/health"

check_health() {
    local gateway=$1
    curl -sf "$HEALTH_URL" > /dev/null
    return $?
}

while true; do
    if ! check_health "$PRIMARY"; then
        echo "Primary failed, activating backup..."
        # 启动备份
        ssh "$BACKUP" "openclaw start"

        # 等待恢复
        while ! check_health "$PRIMARY"; do
            sleep 10
        done

        echo "Primary recovered, switching back..."
        ssh "$BACKUP" "openclaw stop"
    fi

    sleep 30
done
```

### Keepalived 配置

```bash
# /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
    state MASTER  # MASTER 或 BACKUP
    interface eth0
    virtual_router_id 51
    priority 100  # MASTER 优先级更高
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        192.168.1.100  # 虚拟 IP
    }

    track_script {
        chk_openclaw
    }
}

vrrp_script chk_openclaw {
    script "/usr/bin/curl -f http://localhost:18789/health"
    interval 2
    weight 2
}
```

---

## 配置同步

### 配置中心

```json
{
  "configSync": {
    "enabled": true,
    "backend": "consul",
    "prefix": "openclaw/config",
    "watch": true,
    "syncOnStart": true,
    "syncInterval": 60
  }
}
```

### 配置同步脚本

```bash
#!/bin/bash
# sync-config.sh

CONFIG_DIR="/etc/openclaw"
REMOTES=(
    "user@192.168.1.11"
    "user@192.168.1.12"
)

# 推送配置到所有 Gateway
for remote in "${REMOTES[@]}"; do
    echo "Syncing to $remote..."
    rsync -avz "$CONFIG_DIR/" "$remote:$CONFIG_DIR/"

    # 重启 Gateway
    ssh "$remote" "openclaw restart"
done

echo "Configuration synced to all gateways"
```

### Git 配置同步

```bash
# 使用 Git 管理配置
cd /etc/openclaw
git pull

# 定期同步的 Cron 任务
*/5 * * * * cd /etc/openclaw && git pull && openclaw reload
```

---

## 集群管理

### Kubernetes StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: openclaw
spec:
  serviceName: openclaw
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
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        volumeMounts:
        - name: data
          mountPath: /root/.openclaw
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### Docker Swarm

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    image: openclaw/gateway:latest
    deploy:
      mode: replicated
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    ports:
      - "18789:18789"
    volumes:
      - openclaw-data:/root/.openclaw
    networks:
      - openclaw-net

volumes:
  openclaw-data:

networks:
  openclaw-net:
    driver: overlay
```

---

## 小结

本节我们学习了：

1. ✅ 多 Gateway 架构 - 主备、负载均衡、地理分布
2. ✅ Gateway 发现 - 服务发现和健康检查
3. ✅ 负载均衡 - HAProxy、Nginx、Traefik
4. ✅ 故障转移 - 主备模式和自动故障转移
5. ✅ 配置同步 - 配置中心和同步脚本
6. ✅ 集群管理 - Kubernetes 和 Docker Swarm

## 第六阶段完成！

恭喜！你已经完成了生产部署学习：

- ✅ 部署选项
- ✅ 监控与日志
- ✅ 备份与恢复
- ✅ 远程访问
- ✅ 多 Gateway 配置

## 下一步

[第七阶段：项目实战](../stage7-projects/26-project-coding-assistant.md) - 实战项目 1：个人 AI 编程助手
