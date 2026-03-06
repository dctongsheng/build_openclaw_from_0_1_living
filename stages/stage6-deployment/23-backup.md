# 备份与恢复

> 学习目标：建立数据保护策略

## 目录

- [备份策略概述](#备份策略概述)
- [Workspace 备份](#workspace-备份)
- [Session 备份](#session-备份)
- [配置备份](#配置备份)
- [灾难恢复计划](#灾难恢复计划)
- [自动化备份](#自动化备份)
- [恢复测试](#恢复测试)
- [小结](#小结)

---

## 备份策略概述

### 3-2-1 原则

```
3 - 3 份副本
2 - 2 种不同介质
1 - 1 份异地备份

示例:
├── 本地 (主) - 运行数据
├── 本地 (备) - 定期备份
└── 异地 (云) - 远程备份
```

### 备份类型

```
完整备份:
├── 所有数据
├── 最安全
└── 恢复最快

增量备份:
├── 只备份变化
├── 节省空间
└── 需要完整备份

差异备份:
├── 自完整备份后的变化
├── 平衡空间和时间
└── 中等恢复时间
```

---

## Workspace 备份

### 备份内容

```
~/.openclaw/workspace/
├── AGENTS.md
├── SOUL.md
├── TOOLS.md
├── IDENTITY.md
└── USER.md
```

### 手动备份

```bash
# 创建备份目录
mkdir -p ~/backups/openclaw

# 备份 workspace
cp -r ~/.openclaw/workspace ~/backups/openclaw/workspace-$(date +%Y%m%d)

# 创建压缩备份
tar czf ~/backups/openclaw/workspace-$(date +%Y%m%d).tar.gz -C ~/.openclaw workspace

# 使用 rsync 同步
rsync -av ~/.openclaw/workspace/ ~/backups/openclaw/workspace/
```

### 自动化备份

```bash
#!/bin/bash
# backup-workspace.sh

BACKUP_DIR="$HOME/backups/openclaw"
DATE=$(date +%Y%m%d_%H%M%S)

# 创建备份
tar czf "$BACKUP_DIR/workspace-$DATE.tar.gz" -C ~/.openclaw workspace

# 删除旧备份 (保留最近 7 天)
find "$BACKUP_DIR" -name "workspace-*.tar.gz" -mtime +7 -delete

echo "Backup completed: workspace-$DATE.tar.gz"
```

### 配置 Cron 任务

```bash
# 编辑 crontab
crontab -e

# 添加每天凌晨 2 点备份
0 2 * * * /path/to/backup-workspace.sh >> /var/log/openclaw-backup.log 2>&1

# 每周日完整备份
0 2 * * 0 /path/to/full-backup.sh >> /var/log/openclaw-backup.log 2>&1
```

---

## Session 备份

### Session 数据

```
~/.openclaw/sessions/
├── session-001.jsonl
├── session-002.jsonl
└── ...
```

### 备份策略

```json
{
  "backup": {
    "sessions": {
      "enabled": true,
      "strategy": "incremental",
      "schedule": "0 */4 * * *",  // 每 4 小时
      "retention": {
        "days": 30,
        "minSessions": 100
      },
      "compression": true
    }
  }
}
```

### 导出会话

```bash
# 导出所有会话
openclaw sessions export --all --format json --output sessions-backup.json

# 导出特定会话
openclaw sessions export session-001 --output session-001.json

# 导出为 CSV
openclaw sessions export --format csv --output sessions.csv
```

---

## 配置备份

### 配置文件

```
~/.openclaw/
├── openclaw.json       # 主配置
├── channels/           # 通道配置
├── agents/             # Agent 配置
└── skills/             # Skills
```

### 完整配置备份

```bash
#!/bin/bash
# backup-config.sh

BACKUP_DIR="$HOME/backups/openclaw"
DATE=$(date +%Y%m%d_%H%M%S)
CONFIG_DIR="$HOME/.openclaw"

# 创建备份目录
mkdir -p "$BACKUP_DIR/config-$DATE"

# 备份配置文件
cp "$CONFIG_DIR/openclaw.json" "$BACKUP_DIR/config-$DATE/"
cp -r "$CONFIG_DIR/channels" "$BACKUP_DIR/config-$DATE/"
cp -r "$CONFIG_DIR/agents" "$BACKUP_DIR/config-$DATE/"
cp -r "$CONFIG_DIR/skills" "$BACKUP_DIR/config-$DATE/"

# 创建压缩包
tar czf "$BACKUP_DIR/config-$DATE.tar.gz" -C "$BACKUP_DIR" "config-$DATE"

# 清理
rm -rf "$BACKUP_DIR/config-$DATE"

echo "Config backup completed: config-$DATE.tar.gz"
```

### 版本控制

```bash
# 使用 Git 管理配置
cd ~/.openclaw
git init
git add openclaw.json channels/ agents/
git commit -m "Initial config backup"

# 设置远程仓库
git remote add origin https://github.com/user/openclaw-config.git
git push -u origin main

# 自动备份脚本
#!/bin/bash
cd ~/.openclaw
git add -A
git commit -m "Auto backup: $(date)"
git push
```

---

## 灾难恢复计划

### 恢复优先级

```
优先级 1 - 配置文件
优先级 2 - Workspace (自定义)
优先级 3 - Sessions (可重新生成)
优先级 4 - 日志 (可选)
```

### 恢复步骤

#### 1. 配置恢复

```bash
# 解压配置备份
tar xzf ~/backups/openclaw/config-20240115.tar.gz -C ~/.openclaw/

# 或从 Git 恢复
cd ~/.openclaw
git pull origin main
```

#### 2. Workspace 恢复

```bash
# 解压 workspace
tar xzf ~/backups/openclaw/workspace-20240115.tar.gz -C ~/.openclaw/

# 验证
openclaw config validate
```

#### 3. Sessions 恢复

```bash
# 导入会话
openclaw sessions import --file sessions-backup.json

# 或复制会话文件
cp -r ~/backups/openclaw/sessions/* ~/.openclaw/sessions/
```

### 完整恢复脚本

```bash
#!/bin/bash
# restore.sh

BACKUP_DATE=${1:-$(date +%Y%m%d)}
BACKUP_DIR="$HOME/backups/openclaw"

echo "Starting restore from $BACKUP_DATE..."

# 停止 Gateway
openclaw stop

# 恢复配置
echo "Restoring configuration..."
tar xzf "$BACKUP_DIR/config-$BACKUP_DATE.tar.gz" -C /tmp/
cp -r /tmp/config-$BACKUP_DATE/* ~/.openclaw/

# 恢复 workspace
echo "Restoring workspace..."
tar xzf "$BACKUP_DIR/workspace-$BACKUP_DATE.tar.gz" -C ~/.openclaw/

# 恢复 sessions
echo "Restoring sessions..."
tar xzf "$BACKUP_DIR/sessions-$BACKUP_DATE.tar.gz" -C ~/.openclaw/

# 重启 Gateway
echo "Restarting Gateway..."
openclaw start

# 验证
echo "Verifying restore..."
openclaw doctor
openclaw status

echo "Restore completed!"
```

---

## 自动化备份

### 完整备份脚本

```bash
#!/bin/bash
# full-backup.sh

set -e

BACKUP_DIR="$HOME/backups/openclaw"
DATE=$(date +%Y%m%d_%H%M%S)
OPENCLAW_DIR="$HOME/.openclaw"

# 创建备份目录
mkdir -p "$BACKUP_DIR/$DATE"

echo "Starting full backup at $(date)"

# 备份配置
echo "Backing up configuration..."
tar czf "$BACKUP_DIR/$DATE/config.tar.gz" \
  -C "$OPENCLAW_DIR" \
  openclaw.json channels agents skills

# 备份 workspace
echo "Backing up workspace..."
tar czf "$BACKUP_DIR/$DATE/workspace.tar.gz" \
  -C "$OPENCLAW_DIR" \
  workspace

# 备份 sessions
echo "Backing up sessions..."
tar czf "$BACKUP_DIR/$DATE/sessions.tar.gz" \
  -C "$OPENCLAW_DIR" \
  sessions

# 创建完整备份包
echo "Creating full backup archive..."
cd "$BACKUP_DIR"
tar czf "full-backup-$DATE.tar.gz" "$DATE"
rm -rf "$DATE"

# 清理旧备份
echo "Cleaning old backups..."
find "$BACKUP_DIR" -name "full-backup-*.tar.gz" -mtime +30 -delete

# 上传到云存储 (可选)
if [ -n "$CLOUD_BACKUP_ENABLED" ]; then
  echo "Uploading to cloud..."
  aws s3 cp "$BACKUP_DIR/full-backup-$DATE.tar.gz" \
    s3://my-backups/openclaw/ \
    --storage-class GLACIER
fi

echo "Backup completed: full-backup-$DATE.tar.gz"
```

### Cron 配置

```bash
# 每日增量备份
0 */6 * * * /path/to/incremental-backup.sh

# 每周完整备份
0 2 * * 0 /path/to/full-backup.sh

# 每月验证备份
0 3 1 * * /path/to/verify-backups.sh
```

---

## 恢复测试

### 测试脚本

```bash
#!/bin/bash
# test-restore.sh

BACKUP_FILE=$1
TEST_DIR="/tmp/openclaw-restore-test"

echo "Testing restore: $BACKUP_FILE"

# 创建测试目录
mkdir -p "$TEST_DIR"

# 解压备份
tar xzf "$BACKUP_FILE" -C "$TEST_DIR"

# 验证配置
echo "Validating configuration..."
openclaw config validate --config "$TEST_DIR/openclaw.json"

# 验证 workspace
if [ -f "$TEST_DIR/workspace/AGENTS.md" ]; then
  echo "✓ Workspace OK"
else
  echo "✗ Workspace missing"
  exit 1
fi

# 清理
rm -rf "$TEST_DIR"

echo "Restore test passed!"
```

### 定期测试

```bash
# 每月测试最新备份
0 4 1 * * /path/to/test-restore.sh $(ls -t ~/backups/openclaw/full-backup-*.tar.gz | head -1)
```

---

## 小结

本节我们学习了：

1. ✅ 备份策略概述 - 3-2-1 原则和备份类型
2. ✅ Workspace 备份 - 手动和自动化
3. ✅ Session 备份 - 策略和导出
4. ✅ 配置备份 - 完整配置备份和版本控制
5. ✅ 灾难恢复计划 - 恢复优先级和步骤
6. ✅ 自动化备份 - 完整备份脚本和 Cron
7. ✅ 恢复测试 - 测试脚本和定期测试

## 备份检查清单

- [ ] 配置自动备份
- [ ] 设置 Cron 任务
- [ ] 验证备份完整性
- [ ] 测试恢复流程
- [ ] 配置异地备份
- [ ] 设置备份告警
- [ ] 定期审查备份策略
- [ ] 文档化恢复步骤

## 下一步

[远程访问](./24-remote-access.md) - 安全地从外部访问
