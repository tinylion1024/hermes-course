# Automation Patterns｜自动化模式

> 原文：[Automate Anything with Cron](https://hermes-agent.nousresearch.com/docs/guides/automate-with-cron) + [Automation Templates](https://hermes-agent.nousresearch.com/docs/guides/automation-templates) + [Cron Troubleshooting](https://hermes-agent.nousresearch.com/docs/guides/cron-troubleshooting)

---

## 目录

- [Key Concept｜核心概念](#key-concept核心概念)
- [Pattern 1: Website Change Monitor｜网站变更监控](#pattern-1-website-change-monitor网站变更监控)
- [Pattern 2: Daily Briefing Bot｜每日简报 Bot](#pattern-2-daily-briefing-bot每日简报-bot)
- [Pattern 3: Health Check & Alerting｜健康检查与告警](#pattern-3-health-check--alerting健康检查与告警)
- [Pattern 4: Automated Reporting｜自动化报告](#pattern-4-automated-reporting自动化报告)
- [Pattern 5: Slack/Teams Integration｜Slack/Teams 集成](#pattern-5-slackteams-integration-slackteams-集成)
- [The Golden Rule: Self-Contained Prompts｜黄金法则：自包含提示](#the-golden-rule-self-contained-prompts黄金法则自包含提示)
- [Multi-Topic Briefings｜多主题简报](#multi-topic-briefings多主题简报)
- [Using Delegation for Parallel Research｜用委托进行并行研究](#using-delegation-for-parallel-research用委托进行并行研究)
- [Cron Troubleshooting｜故障排除](#cron-troubleshooting故障排除)

---

## Key Concept｜核心概念

> ⚠️ **Cron 任务在全新的 Agent 会话中运行**，没有当前聊天的记忆。提示必须**完全自包含**——包含 Agent 需要知道的一切。

---

## Pattern 1: Website Change Monitor｜网站变更监控

监控 URL 变化，仅在有变化时通知。

`script` 参数是秘密武器。Python 脚本在每次执行前运行，其 stdout 成为 Agent 的上下文。脚本处理机械性工作（获取、对比）；Agent 处理推理（这个变化有趣吗？）。

### 创建监控脚本

```bash
mkdir -p ~/.hermes/scripts
```

`~/.hermes/scripts/watch-site.py`：

```python
import hashlib
import json
import os
import urllib.request
from datetime import datetime, timedelta

WATCH_FILE = os.path.expanduser("~/.hermes/scripts/.watch-state.json")
TARGET_URL = "https://news.ycombinator.com"
MAX_AGE_HOURS = 24

def fetch():
    with urllib.request.urlopen(TARGET_URL, timeout=10) as r:
        return r.read().decode("utf-8", errors="replace")

def get_stored():
    if os.path.exists(WATCH_FILE):
        with open(WATCH_FILE) as f:
            return json.load(f)
    return {}

def save(data):
    with open(WATCH_FILE, "w") as f:
        json.dump(data, f)

def main():
    new_content = fetch()
    new_hash = hashlib.md5(new_content.encode()).hexdigest()
    stored = get_stored()

    changed = stored.get("hash") != new_hash
    stale = (
        stored.get("timestamp")
        and datetime.fromisoformat(stored["timestamp"]) < datetime.now() - timedelta(hours=MAX_AGE_HOURS)
    )

    if changed or stale:
        save({"hash": new_hash, "timestamp": datetime.now().isoformat()})
        if changed:
            print(f"CHANGE_DETECTED: Content at {TARGET_URL} has changed.")
        else:
            print(f"STALE_DATA: No change detected in {MAX_AGE_HOURS}h, but check is overdue.")
        print("---DIFF_CONTENT---")
        print(new_content[:5000])  # First 5000 chars for context
    else:
        print("NO_CHANGE: No significant change detected.")

if __name__ == "__main__":
    main()
```

### 创建 Cron 任务

```bash
hermes cron create \
  --name "hacker-news-monitor" \
  --schedule "0 */4 * * *" \
  --script ~/.hermes/scripts/watch-site.py \
  --deliver telegram \
  "Analyze the fetched content for interesting articles or changes.
   Summarize the top 3 most interesting findings in a concise report.
   If NO_CHANGE or STALE_DATA, just acknowledge briefly."
```

---

## Pattern 2: Daily Briefing Bot｜每日简报 Bot

### 快速创建

```bash
hermes cron create \
  --schedule "0 8 * * *" \
  --deliver telegram \
  "Create a morning briefing. Search the web for the latest news about AI agents
   and open source LLMs. Find at least 5 recent articles from the past 24 hours.
   Summarize the top 3 most important stories in a concise daily briefing format.
   For each story include: a clear headline, a 2-sentence summary, and the source URL.
   Use a friendly, professional tone. Format with emoji bullet points."
```

---

## Pattern 3: Health Check & Alerting｜健康检查与告警

### 基本健康检查

```bash
hermes cron create \
  --name "daily-health-check" \
  --schedule "0 9 * * *" \
  --deliver telegram \
  "Run a health check on our services:
   1. Check if the web server is responding: curl -s -o /dev/null -w '%{http_code}' https://oursite.com
   2. Check database connectivity
   3. Check disk usage: df -h
   4. Check memory: free -h

   Report status as: ✅ All healthy / ⚠️ Issues found / 🔴 Critical

   If issues found, list each one with a brief description."
```

---

## Pattern 4: Automated Reporting｜自动化报告

### 每周代码统计报告

```bash
hermes cron create \
  --name "weekly-code-stats" \
  --schedule "0 10 * * 1" \
  --deliver telegram \
  "Generate a weekly code activity report:
   1. Run: git log --since='7 days ago' --oneline --stat
   2. Count commits per author: git shortlog -sn --since='7 days ago'
   3. List new files: git log --name-status --since='7 days ago' | grep -E '^[A-Z]' | sort -u
   4. Check for any failing tests

   Format as:
   ## Weekly Code Report — [DATE]
   ### Summary
   - Total commits: X
   - Authors: X
   - New files: X

   ### Top Contributors
   [list]

   ### Major Changes
   [list with descriptions]

   If tests failed, add a ⚠️ Warning section."
```

---

## Pattern 5: Slack/Teams Integration｜Slack/Teams 集成 {#pattern-5-slackteams-integration-slackteams-集成}

通过 Webhook 发送通知到 Slack 或 Microsoft Teams。

### 创建 Webhook 脚本

`~/.hermes/scripts/slack-notify.py`：

```python
#!/usr/bin/env python3
import json
import sys
import urllib.request

WEBHOOK_URL = "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

def send(message):
    payload = {"text": message}
    data = json.dumps(payload).encode("utf-8")
    req = urllib.request.Request(
        WEBHOOK_URL,
        data=data,
        headers={"Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req, timeout=10) as r:
        return r.read()

if __name__ == "__main__":
    message = sys.stdin.read()
    send(message)
```

### 在 Cron 中使用

```bash
hermes cron create \
  --name "slack-daily-briefing" \
  --schedule "0 8 * * *" \
  --deliver local \
  "Your briefing content here"
```

然后配置 Agent 通过本地脚本发送到 Slack。

---

## The Golden Rule: Self-Contained Prompts｜黄金法则：自包含提示 {#the-golden-rule-self-contained-prompts黄金法则自包含提示}

> ⚠️ **Cron 任务在完全新的会话中运行**——没有对之前对话的记忆，没有对"之前设置的内容"的上下文。提示必须包含 Agent 完成工作所需的一切。

```diff
- Bad prompt:
- "Do my usual morning briefing."

+ Good prompt:
+ "Search the web for the latest news about AI agents and open source LLMs.
+  Find at least 5 recent articles from the past 24 hours. Summarize the
+  top 3 most important stories in a concise daily briefing format. For each
+  story include: a clear headline, a 2-sentence summary, and the source URL.
+  Use a friendly, professional tone. Format with emoji bullet points."
```

好提示具体说明了**搜索什么**、**多少文章**、**什么格式**、**什么语气**。它是 Agent 在一次完成工作所需的一切。

---

## Multi-Topic Briefings｜多主题简报

一份简报覆盖多个领域：

```bash
/cron add "0 8 * * *" "Create a morning briefing covering three topics. For each topic, search the web for recent news from the past 24 hours and summarize the top 2 stories with links.

Topics:
1. AI and machine learning — focus on open source models and agent frameworks
2. Cryptocurrency — focus on Bitcoin, Ethereum, and regulatory news
3. Space exploration — focus on SpaceX, NASA, and commercial space

Format as a clean briefing with section headers and emoji. End with today's date and a motivational quote."
```

---

## Using Delegation for Parallel Research｜用委托进行并行研究

为了更快的简报，让 Hermes 将每个主题委托给子 Agent：

```bash
hermes cron create \
  --name "fast-multi-topic-briefing" \
  --schedule "0 8 * * *" \
  --deliver telegram \
  "Create a morning briefing by delegating each topic to a sub-agent:
   1. AI/ML news
   2. Crypto news
   3. Space news

   Each sub-agent should search for recent news (past 24h) and return 2 stories.
   Synthesize into one clean briefing with section headers."
```

---

## Cron Troubleshooting｜故障排除

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Cron 不运行 | Gateway 未启动 | `hermes gateway` 确保运行 |
| 消息未送达 | 未设置 home channel | `/sethome` 或检查 deliver 参数 |
| Token 耗尽 | 提示太长 | 简化提示，减少搜索 |
| 错误执行 | 脚本路径错误 | 检查 `--script` 路径 |

### 诊断步骤

```bash
# 1. 检查 cron 任务列表
hermes cron list

# 2. 检查 Gateway 状态
hermes gateway status

# 3. 查看日志
tail -f ~/.hermes/logs/gateway.log

# 4. 手动运行任务测试
hermes cron run <job_id>
```

### 检查项清单

- [ ] Gateway 正在运行
- [ ] Cron 任务已创建并启用
- [ ] Home channel 已设置（对于主动消息）
- [ ] 提示是自包含的
- [ ] 脚本路径正确（如果使用 script 参数）

---

## 快速参考

| Cron 表达式 | 含义 |
|-------------|------|
| `0 8 * * *` | 每天 8:00 |
| `0 */2 * * *` | 每 2 小时 |
| `0 9,13,17 * * 1-5` | 工作日 9:00、13:00、17:00 |
| `0 9 * * 1` | 每周一 9:00 |

**黄金法则**：提示必须**完全自包含**，因为 Cron 在全新会话中运行。
