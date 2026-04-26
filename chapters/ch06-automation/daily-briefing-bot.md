# 6.2 每日简报 Bot

## 实战项目概述

本节创建一个**每日简报 Bot**，自动生成并推送每日工作简报。

---

## 功能需求

1. 每天定时执行（如早上9点）
2. 分析用户近期工作内容
3. 生成简报，包含：
   - 昨日完成的任务
   - 今日待办
   - 遇到的问题（如果有）
4. 推送到指定平台（Telegram/Discord/Email）

---

## 实现步骤

### 第一步：创建简报 Skill

在 `~/.hermes/skills/daily-briefing/SKILL.md`：

```yaml
---
name: daily-briefing
description: 生成每日工作简报
triggers:
  - "生成简报"
  - "每日报告"
  - "daily briefing"
---

# Daily Briefing Skill

自动分析工作进度，生成简报。

## 输入

- 近期会话历史（自动获取）
- 项目进度（如果有记忆）

## 输出格式

\`\`\`
📅 每日简报 - {日期}

✅ 昨日完成
1. ...
2. ...

📋 今日待办
1. ...
2. ...

⚠️ 注意事项
- ...

💡 建议
- ...
\`\`\`

## 执行流程

1. 检索近期记忆中的任务信息
2. 分析会话历史中的工作内容
3. 整理并生成简报
4. 按目标格式输出
```

### 第二步：创建 CRON 任务

```bash
hermes cron create \
  --name "每日简报" \
  --schedule "0 9 * * *" \
  --prompt "使用 daily-briefing skill 生成今日简报，然后发送到我的 Telegram (@your_username)" \
  --skill daily-briefing \
  --target telegram:@your_username
```

### 第三步：测试

```bash
# 手动执行测试
hermes cron run <job-id>
```

---

## 进阶版本

### 多语言支持

```yaml
skill:
  daily-briefing:
    # 支持的语言
    languages:
      - zh-CN
      - en-US
    # 默认语言
    default: zh-CN
```

### 团队版本

```bash
# 为团队创建简报
hermes cron create \
  --name "团队日报" \
  --schedule "0 18 * * *" \
  --prompt "收集团队成员的工作更新，生成团队日报，发送到 #general 频道" \
  --target discord:#general
```

### 邮件版本

```bash
hermes cron create \
  --name "邮件日报" \
  --schedule "0 9 * * *" \
  --prompt "生成每日简报，发送邮件到 team@company.com" \
  --target email:team@company.com
```

---

## 完整配置示例

### config.yaml

```yaml
cron:
  enabled: true
  timezone: Asia/Shanghai

skills:
  daily-briefing:
    enabled: true
    auto_create: false

telegram:
  enabled: true
  bot_token: ${TELEGRAM_BOT_TOKEN}
```

### 预期输出示例

```
📅 每日简报 - 2024-01-15

✅ 昨日完成
1. 完成用户认证模块开发
2. 修复登录页面样式问题
3. Code review 3个 PR

📋 今日待办
1. 继续开发支付模块
2. 编写单元测试
3. 参加技术评审会议

⚠️ 注意事项
- 支付API文档待更新
- 需要联系第三方服务商

💡 建议
- 支付模块复杂度较高，建议预留更多测试时间
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 简报内容为空 | 检查记忆系统是否有内容 |
| 发送失败 | 检查目标渠道配置 |
| 格式混乱 | 检查 Skill 模板定义 |

---

## 下一步

- [第七章：高级应用](../ch07-advanced/README.md) - Profiles、API Server