# 监控与日志

> 学习目标：建立完善的监控体系

## 目录

- [监控体系概述](#监控体系概述)
- [Gateway 健康检查](#gateway-健康检查)
- [日志配置](#日志配置)
- [性能指标](#性能指标)
- [错误追踪](#错误追踪)
- [告警配置](#告警配置)
- [监控仪表板](#监控仪表板)
- [小结](#小结)

---

## 监控体系概述

### 监控层级

```
监控层级:
├── 基础设施
│   ├── CPU
│   ├── 内存
│   ├── 磁盘
│   └── 网络
├── 应用
│   ├── Gateway 状态
│   ├── Agent 性能
│   ├── 通道连接
│   └── API 调用
└── 业务
    ├── 用户活跃度
    ├── Token 使用
    ├── 会话统计
    └── 错误率
```

### 监控目标

```
目标:
├── 可用性: 99.9%
├── 响应时间: P95 < 5s
├── 错误率: < 1%
└── 资源使用: < 80%
```

---

## Gateway 健康检查

### 内置健康检查

```bash
# 检查 Gateway 状态
openclaw status

# 输出:
# Gateway: running
# PID: 12345
# Uptime: 2 days, 14 hours
# Sessions: 15 active
# Channels: 3 connected
```

### 健康检查端点

```javascript
// GET /health
{
  "status": "healthy",
  "timestamp": "2024-01-15T10:30:00Z",
  "components": {
    "gateway": "healthy",
    "channels": {
      "whatsapp": "connected",
      "telegram": "connected",
      "discord": "connected"
    },
    "agents": {
      "coding": "ready",
      "writing": "ready"
    },
    "database": "connected",
    "cache": "connected"
  }
}
```

### 自定义健康检查

```json
{
  "health": {
    "checks": [
      {
        "name": "database",
        "type": "tcp",
        "host": "localhost",
        "port": 5432,
        "interval": 30,
        "timeout": 5
      },
      {
        "name": "api",
        "type": "http",
        "url": "https://api.example.com/health",
        "interval": 60,
        "timeout": 10
      },
      {
        "name": "disk",
        "type": "disk",
        "path": "/",
        "threshold": 90,
        "interval": 300
      }
    ]
  }
}
```

---

## 日志配置

### 日志级别

```
DEBUG → INFO → WARN → ERROR → FATAL
  ↑       ↑      ↑      ↑       ↑
详细    信息   警告    错误    致命
```

### 日志配置

```json
{
  "logging": {
    "level": "info",
    "file": "~/.openclaw/gateway.log",
    "console": true,
    "format": "json",
    "rotation": {
      "enabled": true,
      "maxSize": "100M",
      "maxFiles": 10,
      "compress": true
    },
    "filters": {
      "channels": {
        "whatsapp": "debug",
        "telegram": "info"
      },
      "agents": {
        "coding": "debug"
      }
    }
  }
}
```

### 结构化日志

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "info",
  "context": {
    "component": "gateway",
    "sessionId": "session-001",
    "userId": "user123",
    "channel": "whatsapp"
  },
  "message": "Message received",
  "data": {
    "messageId": "msg-123",
    "length": 150
  }
}
```

### 日志查询

```bash
# 查看实时日志
openclaw logs -f

# 过滤日志级别
openclaw logs --level error

# 过滤通道
openclaw logs --channel whatsapp

# 过滤时间范围
openclaw logs --since 1h

# 搜索关键词
openclaw logs | grep "ERROR"

# 统计错误
openclaw logs --level error | wc -l
```

---

## 性能指标

### 关键指标

```json
{
  "metrics": {
    "enabled": true,
    "endpoint": "/metrics",
    "format": "prometheus",
    "metrics": [
      {
        "name": "response_time",
        "type": "histogram",
        "labels": ["agent", "channel", "user"],
        "buckets": [100, 500, 1000, 2000, 5000, 10000]
      },
      {
        "name": "token_usage",
        "type": "counter",
        "labels": ["agent", "model"]
      },
      {
        "name": "message_count",
        "type": "counter",
        "labels": ["channel", "direction"]
      },
      {
        "name": "active_sessions",
        "type": "gauge"
      },
      {
        "name": "error_rate",
        "type": "gauge",
        "labels": ["type"]
      }
    ]
  }
}
```

### Prometheus 配置

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'openclaw'
    scrape_interval: 15s
    static_configs:
      - targets: ['localhost:18789']
    metrics_path: /metrics
```

### Grafana 仪表板

```json
{
  "dashboard": {
    "title": "OpenClaw Dashboard",
    "panels": [
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, openclaw_response_time_bucket)"
          }
        ]
      },
      {
        "title": "Token Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(openclaw_token_usage_total[5m])"
          }
        ]
      },
      {
        "title": "Active Sessions",
        "type": "stat",
        "targets": [
          {
            "expr": "openclaw_active_sessions"
          }
        ]
      }
    ]
  }
}
```

---

## 错误追踪

### 错误收集

```json
{
  "errorTracking": {
    "enabled": true,
    "services": {
      "sentry": {
        "dsn": "${SENTRY_DSN}",
        "environment": "production",
        "sampleRate": 0.1
      }
    },
    "filter": {
      "exclude": [
        "timeout",
        "rate_limit"
      ]
    }
  }
}
```

### Sentry 集成

```javascript
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: 'production',
  tracesSampleRate: 0.1,
});

// 捕获错误
try {
  // ... 代码
} catch (error) {
  Sentry.captureException(error);
}
```

### 错误分析

```bash
# 查看错误统计
openclaw errors stats

# 查看最近错误
openclaw errors recent

# 错误趋势
openclaw errors trend --period 7d

# 错误详情
openclaw errors show ERROR_ID
```

---

## 告警配置

### 告警规则

```json
{
  "alerts": {
    "enabled": true,
    "rules": [
      {
        "name": "high_error_rate",
        "condition": "error_rate > 0.05",
        "duration": "5m",
        "severity": "critical",
        "notification": ["email", "slack"]
      },
      {
        "name": "slow_response",
        "condition": "response_time_p95 > 5000",
        "duration": "10m",
        "severity": "warning",
        "notification": ["slack"]
      },
      {
        "name": "high_memory",
        "condition": "memory_usage > 0.8",
        "duration": "5m",
        "severity": "warning",
        "notification": ["email"]
      },
      {
        "name": "gateway_down",
        "condition": "gateway_status != 'running'",
        "duration": "1m",
        "severity": "critical",
        "notification": ["email", "slack", "sms"]
      }
    ]
  }
}
```

### 告警渠道

```json
{
  "notifications": {
    "email": {
      "enabled": true,
      "to": ["admin@example.com"],
      "from": "alerts@openclaw.example.com",
      "smtp": {
        "host": "smtp.example.com",
        "port": 587,
        "secure": true,
        "auth": {
          "user": "${SMTP_USER}",
          "pass": "${SMTP_PASS}"
        }
      }
    },
    "slack": {
      "enabled": true,
      "webhook": "${SLACK_WEBHOOK_URL}",
      "channel": "#alerts",
      "username": "OpenClaw Bot",
      "icon": ":warning:"
    },
    "webhook": {
      "enabled": true,
      "url": "${WEBHOOK_URL}",
      "method": "POST",
      "headers": {
        "Authorization": "Bearer ${WEBHOOK_TOKEN}"
      }
    }
  }
}
```

### Prometheus AlertManager

```yaml
# alertmanager.yml
route:
  receiver: 'default'
  group_by: ['alertname', 'cluster', 'service']

receivers:
  - name: 'default'
    email_configs:
      - to: 'admin@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.example.com:587'

  - name: 'slack'
    slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#alerts'
```

---

## 监控仪表板

### Control UI 仪表板

```bash
# 打开监控仪表板
openclaw monitor

# 或通过 Web UI
openclaw dashboard
# 导航到 Monitoring 页面
```

### 自定义仪表板

```json
{
  "dashboard": {
    "name": "Custom Dashboard",
    "layout": "grid",
    "panels": [
      {
        "type": "stat",
        "title": "总消息数",
        "metric": "message_count_total",
        "valueFormat": "number"
      },
      {
        "type": "graph",
        "title": "响应时间",
        "metric": "response_time_p95",
        "timeRange": "1h"
      },
      {
        "type": "table",
        "title": "活跃会话",
        "metric": "active_sessions",
        "groupBy": ["agent", "channel"]
      }
    ]
  }
}
```

### Grafana 集成

```bash
# 使用 Docker 运行 Grafana
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -v grafana-data:/var/lib/grafana \
  grafana/grafana:latest

# 导入 OpenClaw 仪表板
# Settings → Data Sources → Add Prometheus
# URL: http://prometheus:9090

# 导入仪表板 JSON
# Dashboards → Import → Upload JSON file
```

---

## 小结

本节我们学习了：

1. ✅ 监控体系概述 - 层级和目标
2. ✅ Gateway 健康检查 - 内置检查和自定义检查
3. ✅ 日志配置 - 级别、结构化、查询
4. ✅ 性能指标 - 关键指标和 Prometheus 配置
5. ✅ 错误追踪 - Sentry 集成和错误分析
6. ✅ 告警配置 - 规则和通知渠道
7. ✅ 监控仪表板 - Control UI 和 Grafana

## 监控检查清单

- [ ] 配置日志级别和输出
- [ ] 设置日志轮转
- [ ] 启用性能指标
- [ ] 配置健康检查
- [ ] 设置错误追踪
- [ ] 配置告警规则
- [ ] 设置通知渠道
- [ ] 创建监控仪表板
- [ ] 测试告警
- [ ] 定期审查告警规则

## 下一步

[备份与恢复](./23-backup.md) - 建立数据保护策略
