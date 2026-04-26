# 6.1 CRON 定时任务

## 概述

Hermes Agent 内置 CRON 任务系统，支持定时执行自动化任务。

---

## 基本用法

### 创建定时任务

```bash
# 创建一次性任务（30分钟后执行）
hermes cron create --name "提醒" --schedule "30m" --prompt "提醒我开会"

# 创建循环任务（每天9点执行）
hermes cron create --name "日报" --schedule "0 9 * * *" --prompt "生成并发送每日报告"

# 每2小时执行一次
hermes cron create --name "健康检查" --schedule "every 2h" --prompt "检查服务状态"
```

### 调度格式

| 格式 | 说明 | 示例 |
|------|------|------|
| `30m` | 30分钟后 | 一次性任务 |
| `2h` | 2小时后 | 一次性任务 |
| `every 2h` | 每2小时 | 循环任务 |
| `0 9 * * *` | 每天9点 | cron 表达式 |
| `0 */6 * * *` | 每6小时 | cron 表达式 |
| `0 9,18 * * *` | 每天9点和18点 | cron 表达式 |

### 交互式创建

```bash
hermes cron create
# 交互式输入任务配置
```

---

## 管理任务

### 列出任务

```bash
# 列出所有任务
hermes cron list

# 查看任务详情
hermes cron show <job-id>

# 查看运行历史
hermes cron history <job-id>
```

### 控制任务

```bash
# 暂停任务
hermes cron pause <job-id>

# 恢复任务
hermes cron resume <job-id>

# 删除任务
hermes cron delete <job-id>

# 立即执行一次
hermes cron run <job-id>
```

---

## 配置

### config.yaml

```yaml
cron:
  # 启用 CRON
  enabled: true

  # 默认时区
  timezone: Asia/Shanghai

  # 最大并发任务数
  max_concurrent: 3

  # 任务超时（秒）
  default_timeout: 300

  # 失败重试
  retry:
    enabled: true
    max_attempts: 3
    delay: 60
```

### 环境变量

```bash
# 禁用 CRON
HERMES_CRON_ENABLED=false

# 设置时区
HERMES_CRON_TIMEZONE=Asia/Shanghai
```

---

## CRON 表达式详解

### 基本语法

```
┌───────────── 分钟 (0-59)
│ ┌───────────── 小时 (0-23)
│ │ ┌───────────── 日 (1-31)
│ │ │ ┌───────────── 月 (1-12)
│ │ │ │ ┌───────────── 星期 (0-7, 0和7都是周日)
│ │ │ │ │
* * * * *
```

### 常用示例

| 表达式 | 说明 |
|--------|------|
| `0 * * * *` | 每小时整点 |
| `0 9 * * *` | 每天9点 |
| `0 9,18 * * *` | 每天9点和18点 |
| `*/15 * * * *` | 每15分钟 |
| `0 */6 * * *` | 每6小时 |
| `0 9 * * 1-5` | 工作日9点 |
| `0 9 * * 0` | 每周日9点 |

---

## 实战示例

### 示例1：每日站会提醒

```bash
hermes cron create \
  --name "站会提醒" \
  --schedule "0 9 * * 1-5" \
  --prompt "发送消息提醒：站会将在5分钟后开始，请准备好昨日进展和今日计划" \
  --target telegram:@your_channel
```

### 示例2：定时健康检查

```bash
hermes cron create \
  --name "服务健康检查" \
  --schedule "*/30 * * * *" \
  --prompt "检查所有服务状态，如有异常立即发送告警"
```

### 示例3：每日报告生成

```bash
hermes cron create \
  --name "生成日报" \
  --schedule "0 18 * * *" \
  --prompt "分析今日工作，生成简短的每日报告，包含：1.完成的任务 2.遇到的问题 3.明日计划" \
  --target email:team@company.com
```

---

## 任务输出与通知

### 配置通知

```yaml
cron:
  notifications:
    # 任务完成时通知
    on_complete: true
    # 任务失败时通知
    on_failure: true
    # 通知渠道
    channels:
      - telegram:@your_channel
      - email:admin@example.com
```

### 输出处理

```bash
# 查看上次运行输出
hermes cron output <job-id>

# 查看日志
hermes cron logs <job-id> --recent 50
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 任务不执行 | 检查 CRON 是否启用 |
| 时区错误 | 设置正确的 timezone |
| 任务超时 | 增加 timeout 配置 |
| 通知失败 | 检查通知渠道配置 |

---

## 下一步

- [6.2 每日简报 Bot](./daily-briefing-bot.md) - 实战案例
- [第七章：高级应用](../ch07-advanced/README.md) - Profiles、API Server