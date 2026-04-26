# GitHub Integrations｜GitHub 集成

> 原文：[GitHub PR Review Agent](https://hermes-agent.nousresearch.com/docs/guides/github-pr-review-agent) + [GitHub PR Reviews via Webhook](https://hermes-agent.nousresearch.com/docs/guides/webhook-github-pr-review)

---

## 目录

- [PR Review Agent｜PR 审查 Agent](#pr-review-agentpr-审查-agent)
  - [架构图](#架构图)
  - [前置要求](#前置要求)
  - [Step 1: 验证设置](#step-1-验证设置)
  - [Step 2: 手动审查测试](#step-2-手动审查测试)
  - [Step 3: 创建 Review Skill](#step-3-创建-review-skill)
  - [Step 4: 教会它你的规范](#step-4-教会它你的规范)
  - [Step 5: 创建自动化 Cron 任务](#step-5-创建自动化-cron-任务)
  - [常用调度方案](#常用调度方案)
- [Webhook 自动审查｜实时 PR 评论](#webhook-自动审查实时-pr-评论)
  - [工作原理](#工作原理)
  - [前置要求](#前置要求-1)
  - [Step 1: 启用 Webhook 平台](#step-1-启用-webhook-平台)
  - [Step 2: 启动 Gateway](#step-2-启动-gateway)
  - [Step 3: 在 GitHub 注册 Webhook](#step-3-在-github-注册-webhook)
  - [Step 4: 测试 PR](#step-4-测试-pr)
  - [本地测试 with ngrok](#本地测试-with-ngrok)
  - [安全：提示注入风险](#安全提示注入风险)

---

## PR Review Agent｜PR 审查 Agent

**问题**：团队 PR 堆积、无人审查、bug 被合并。

**方案**：一个 AI Agent 7x24 小时监控仓库，审查每个新 PR 的 bug、安全问题和代码质量，并发送摘要——让你只需关注真正需要人工判断的 PR。

### 架构图

```
┌───────────────────────────────────────────────────────────────────┐
│                                                                   │
│   Cron Timer  ──▶  Hermes Agent  ──▶  GitHub API  ──▶  Review     │
│   (every 2h)       + gh CLI           (PR diffs)       delivery   │
│                    + skill                             (Telegram,│
│                    + memory                            Discord,  │
│                                                        local)    │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

本指南使用 **Cron 任务**轮询 PR——无需公网服务器，可跑在 NAT 防火墙后。

### 前置要求

| 要求 | 说明 |
|------|------|
| Hermes Agent 已安装 | 见安装指南 |
| Gateway 运行中 | `hermes gateway install` 或前台运行 |
| `gh` CLI 已安装并认证 | `gh auth login` |
| 消息配置（可选）| Telegram 或 Discord |
| 本地模式（无消息时）| `deliver: "local"` 保存到 `~/.hermes/cron/output/` |

### Step 1: 验证设置

```bash
hermes
```

在聊天中测试：

```
Run: gh pr list --repo NousResearch/hermes-agent --state open --limit 3
```

看到开放 PR 列表？准备就绪。

### Step 2: 手动审查测试

在聊天中让 Hermes 审查一个真实 PR：

```
Review this pull request. Read the diff, check for bugs, security issues,
and code quality. Be specific about line numbers and quote problematic code.
Run: gh pr diff 3888 --repo NousResearch/hermes-agent
```

Hermes 会：
1. 执行 `gh pr diff` 获取代码变更
2. 阅读整个 diff
3. 生成结构化审查

如果质量满意，自动化它。

### Step 3: 创建 Review Skill

Skill 让 Hermes 有**一致的审查指南**，跨会话和 Cron 运行都保持一致。没有它，审查质量参差不齐。

```bash
mkdir -p ~/.hermes/skills/code-review
```

创建 `~/.hermes/skills/code-review/SKILL.md`：

```markdown
---
name: code-review
description: Review pull requests for bugs, security issues, and code quality
---

# Code Review Guidelines

When reviewing a pull request:

## What to Check

1. **Bugs** — Logic errors, off-by-one, null/undefined handling
2. **Security** — Injection, auth bypass, secrets in code, SSRF
3. **Performance** — N+1 queries, unbounded loops, memory leaks
4. **Style** — Naming conventions, dead code, missing error handling
5. **Tests** — Are changes tested? Do tests cover edge cases?

## Output Format

For each finding:
- **File:Line** — exact location
- **Severity** — Critical / Warning / Suggestion
- **What's wrong** — one sentence
- **Fix** — how to fix it

## Rules

- Be specific. Quote the problematic code.
- Don't flag style nitpicks unless they affect readability.
- If the PR looks good, say so. Don't invent problems.
- End with: APPROVE / REQUEST_CHANGES / COMMENT
```

验证加载——启动 `hermes`，应该在启动时的 skills 列表中看到 `code-review`。

### Step 4: 教会它你的规范

这才是让审查者真正有用的部分。开启会话教会 Hermes 你团队的标准：

```
Remember: In our backend repo, we use Python with FastAPI.
All endpoints must have type annotations and Pydantic models.
We don't allow raw SQL — only SQLAlchemy ORM.
Test files go in tests/ and must use pytest fixtures.

Remember: In our frontend repo, we use TypeScript with React.
No `any` types allowed. All components must have props interfaces.
We use React Query for data fetching, never useEffect for API calls.
```

这些记忆**永久保存**——审查者无需每次告知你的规范。

### Step 5: 创建自动化 Cron 任务

现在把它们串联起来。创建每 2 小时运行的 Cron 任务：

```bash
hermes cron create \
  "0 */2 * * *" \
  "Check for new open PRs and review them.
Repos to monitor:
- myorg/backend-api
- myorg/frontend-app

Steps:
1. Run: gh pr list --repo REPO --state open --limit 5 --json number,title,author,createdAt
2. For each PR created or updated in the last 4 hours:
   - Run: gh pr diff NUMBER --repo REPO
   - Review the diff using the code-review guidelines
3. Format output as:
## PR Reviews — today
### [repo] #[number]: [title]
**Author:** [name] | **Verdict:** APPROVE/REQUEST_CHANGES/COMMENT
[findings]

If no new PRs found, say: No new PRs to review." \
  --name "pr-review" \
  --deliver telegram \
  --skill code-review
```

### 常用调度方案

| Cron 表达式 | 含义 |
|-------------|------|
| `0 */2 * * *` | 每 2 小时 |
| `0 9,13,17 * * 1-5` | 工作日 9:00、13:00、17:00 |
| `0 9 * * 1` | 每周一 9:00 |

---

## Webhook 自动审查｜实时 PR 评论

想实时审查而非轮询？Webhook 方案在 PR 打开或更新时**即时**触发——无需轮询。

### 工作原理

```
PR opened/updated → GitHub sends webhook → Hermes receives → 
Agent fetches diff via gh → Posts review comment to PR
```

### 前置要求

| 要求 | 说明 |
|------|------|
| Hermes Agent 运行中 | `hermes gateway` |
| `gh` CLI 已安装认证 | `gh auth login` |
| 公网可达 URL | 本地测试用 ngrok |
| GitHub 仓库管理员权限 | 管理 webhooks |

### Step 1: 启用 Webhook 平台

在 `~/.hermes/config.yaml` 中添加：

```yaml
platforms:
  webhook:
    enabled: true
    extra:
      port: 8644                    # 默认，可改
      rate_limit: 30                # 每路由每分钟最大请求数
    routes:
      github-pr-review:
        secret: "your-webhook-secret-here"  # 必须与 GitHub webhook secret 匹配
        events:
          - pull_request            # GitHub 推送事件类型
        prompt: |
          A pull request event was received (action: {action}).
          PR #{number}: {pull_request.title}
          Author: {pull_request.user.login}
          Branch: {pull_request.head.ref} → {pull_request.base.ref}
          Description: {pull_request.body}
          URL: {pull_request.html_url}

          If the action is "closed" or "labeled", stop here and do not post a comment.
          Otherwise:
          1. Run: gh pr diff {number} --repo {repository.full_name}
          2. Review the code changes for correctness, security issues, and clarity.
          3. Write a concise, actionable review comment and post it.
        deliver: github_comment
        deliver_extra:
          repo: "{repository.full_name}"
          pr_number: "{number}"
```

**关键字段说明**：

| 字段 | 说明 |
|------|------|
| `secret` | 此路由的 HMAC secret。不填则回退到 `extra.secret` |
| `events` | 接受的 `X-GitHub-Event` 头值列表 |
| `prompt` | 模板；`{field}` 和 `{nested.field}` 从 GitHub payload 解析 |
| `deliver: github_comment` | 通过 `gh pr comment` 发帖 |
| `deliver_extra` | 从 payload 解析 repo 和 PR 号 |

> ℹ️ **Payload 不包含 diff**：GitHub webhook payload 包含 PR 元数据（标题、描述、分支名、URL）**但不包含 diff**。提示让 Agent 运行 `gh pr diff` 获取实际变更。`terminal` 工具在默认 `hermes-webhook` toolset 中，无需额外配置。

### Step 2: 启动 Gateway

```bash
hermes gateway
```

应该看到：

```
[webhook] Listening on 0.0.0.0:8644 — routes: github-pr-review
```

验证：

```bash
curl http://localhost:8644/health
# {"status": "ok", "platform": "webhook"}
```

### Step 3: 在 GitHub 注册 Webhook

仓库 → **Settings** → **Webhooks** → **Add webhook**

| 设置 | 值 |
|------|---|
| Payload URL | `https://your-public-url.example.com/webhooks/github-pr-review` |
| Content type | `application/json` |
| Secret | 与 config 中 `secret` 相同的值 |
| Which events? | → Select individual events → check **Pull requests** |

点击 **Add webhook**。GitHub 会立即发送一个 `ping` 事件确认连接。它被安全忽略——`ping` 不在 events 列表中，返回 `{"status": "ignored", "event": "ping"}`。

### Step 4: 测试 PR

创建分支、提交变更、打开 PR。30-90 秒内（取决于 PR 大小和模型），Hermes 应该发布审查评论。

实时跟踪 Agent 进度：

```bash
tail -f "${HERMES_HOME:-/$HOME/.hermes}/logs/gateway.log"
```

### 本地测试 with ngrok

本地运行 Hermes？用 ngrok 暴露它：

```bash
ngrok http 8644
```

复制 `https://...ngrok-free.app` URL 作为 GitHub Payload URL。免费版每次重启 URL 会变。

**本地测试技巧**——用 `deliver: log` 而非 `deliver: github_comment`：

```yaml
# 测试时
deliver: log

# 正式使用
deliver: github_comment
```

### 安全：提示注入风险

> ⚠️ **警告**：Webhook payload 包含攻击者可控数据——PR 标题、commit 消息、描述可能包含恶意指令。当 webhook 端点暴露到互联网时，在沙箱环境中运行 gateway（Docker、SSH 后端）。

---

## 两种方案对比

| 方案 | 触发方式 | 实时性 | 需要公网？ | 适用场景 |
|------|----------|--------|------------|----------|
| **Cron 轮询** | 定时 | 轮询间隔 | 否 | 团队 PR 不多、预算有限 |
| **Webhook** | 事件驱动 | 即时 | 是 | 大量 PR、需要快速反馈 |

---

## 快速参考

| 任务 | 命令 |
|------|------|
| 验证 GitHub 访问 | `gh pr list --repo ORG/REPO --state open --limit 3` |
| 创建 Review Skill | `mkdir -p ~/.hermes/skills/code-review` |
| 创建 Cron 审查任务 | `hermes cron create "0 */2 * * *" "..." --name pr-review --deliver telegram --skill code-review` |
| 启用 Webhook | `platforms.webhook.enabled: true` in `config.yaml` |
| 注册 GitHub Webhook | Settings → Webhooks → Add webhook |

**核心原则**：
- 有 Skill → 质量一致
- 有记忆 → 无需重复告知规范
- 自包含提示 → Cron 任务可靠
