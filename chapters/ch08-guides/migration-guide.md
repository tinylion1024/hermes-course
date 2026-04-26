# Migration Guide｜迁移指南

> 原文：[Migrate from OpenClaw](https://hermes-agent.nousresearch.com/docs/guides/migrate-from-openclaw)

---

## 目录

- [快速开始](#快速开始)
- [选项说明](#选项说明)
- [迁移内容](#迁移内容)
  - [Persona、Memory 和 Instructions](#personamemory-和-instructions)
  - [Skills](#skills)
  - [Model 和 Provider 配置](#model-和-provider-配置)
  - [Agent Behavior](#agent-behavior)
  - [Session Reset Policies](#session-reset-policies)
  - [MCP Servers](#mcp-servers)
  - [TTS (Text-to-Speech)](#tts-text-to-speech)

---

## 快速开始

```bash
# 预览然后迁移（始终先显示预览，然后询问确认）
hermes claw migrate

# 仅预览，不做任何更改
hermes claw migrate --dry-run

# 完整迁移包括 API keys，跳过确认
hermes claw migrate --preset full --yes
```

默认从 `~/.openclaw/` 读取。自动检测旧版 `~/.clawdbot/` 或 `~/.moltbot/` 目录。同样检测旧配置文件名（`clawdbot.json`、`moltbot.json`）。

---

## 选项说明

| 选项 | 说明 |
|------|------|
| `--dry-run` | 仅预览——显示将要迁移的内容后停止 |
| `--preset <name>` | `full`（默认，包含 secrets）或 `user-data`（排除 API keys）|
| `--overwrite` | 冲突时覆盖现有 Hermes 文件（默认：跳过）|
| `--migrate-secrets` | 包含 API keys（`--preset full` 时默认启用）|
| `--source <path>` | 指定 OpenClaw 目录 |
| `--workspace-target <path>` | 指定 AGENTS.md 放置位置 |
| `--skill-conflict <mode>` | `skip`（默认）、`overwrite`、`rename` |
| `--yes` | 跳过确认提示 |

---

## 迁移内容

### Persona、Memory 和 Instructions

| 内容 | OpenClaw 源 | Hermes 目标 | 说明 |
|------|-------------|-------------|------|
| Persona | `workspace/SOUL.md` | `~/.hermes/SOUL.md` | 直接复制 |
| Workspace instructions | `workspace/AGENTS.md` | `AGENTS.md` 在 `--workspace-target` | 需要 `--workspace-target` 参数 |
| Long-term memory | `workspace/MEMORY.md` | `~/.hermes/memories/MEMORY.md` | 解析为条目，合并现有，去重 |
| User profile | `workspace/USER.md` | `~/.hermes/memories/USER.md` | 同上 |
| Daily memory files | `workspace/memory/*.md` | `~/.hermes/memories/MEMORY.md` | 所有日文件合并到主 memory |

也检查 `workspace.default/` 和 `workspace-main/` 作为备用路径（OpenClaw 在近几个版本重命名了 `workspace/` 为 `workspace-main/`，并使用 `workspace-{agentId}` 做多 Agent 设置）。

---

### Skills

| 源 | OpenClaw 位置 | Hermes 目标 |
|------|---------------|-------------|
| Workspace skills | `workspace/skills/` | `~/.hermes/skills/openclaw-imports/` |
| Managed/shared skills | `~/.openclaw/skills/` | `~/.hermes/skills/openclaw-imports/` |
| Personal cross-project | `~/.agents/skills/` | `~/.hermes/skills/openclaw-imports/` |
| Project-level shared | `workspace/.agents/skills/` | `~/.hermes/skills/openclaw-imports/` |

Skill 冲突由 `--skill-conflict` 处理：
- `skip`：保留现有 Hermes skill
- `overwrite`：替换它
- `rename`：创建 `-imported` 副本

---

### Model 和 Provider 配置

| 内容 | OpenClaw 配置路径 | Hermes 目标 | 说明 |
|------|-------------------|-------------|------|
| Default model | `agents.defaults.model` | `config.yaml` → `model` | 可以是字符串或 `{primary, fallbacks}` 对象 |
| Custom providers | `models.providers.*` | `config.yaml` → `custom_providers` | 映射 `baseUrl`、`apiType`/`api`；处理短名称（"openai"、"anthropic"）和带连字符的名称（"openai-completions"、"anthropic-messages"、"google-generative-ai"）|
| Provider API keys | `models.providers.*.apiKey` | `~/.hermes/.env` | 需要 `--migrate-secrets` |

---

### Agent Behavior

| 内容 | OpenClaw 配置路径 | Hermes 配置路径 | 映射 |
|------|-------------------|-----------------|------|
| Max turns | `agents.defaults.timeoutSeconds` | `agent.max_turns` | `timeoutSeconds / 10`，最大 200 |
| Verbose mode | `agents.defaults.verboseDefault` | `agent.verbose` | "off" / "on" / "full" |
| Reasoning effort | `agents.defaults.thinkingDefault` | `agent.reasoning_effort` | "always"/"high"/"xhigh" → "high", "auto"/"medium"/"adaptive" → "medium", "off"/"low"/"none"/"minimal" → "low" |
| Compression | `agents.defaults.compaction.mode` | `compression.enabled` | "off" → false，其他 → true |
| Compression model | `agents.defaults.compaction.model` | `compression.summary_model` | 直接字符串复制 |
| Human delay | `agents.defaults.humanDelay.mode` | `human_delay.mode` | "natural" / "custom" / "off" |
| Human delay timing | `agents.defaults.humanDelay.minMs` / `.maxMs` | `human_delay.min_ms` / `.max_ms` | 直接复制 |
| Timezone | `agents.defaults.userTimezone` | `timezone` | 直接字符串复制 |
| Exec timeout | `tools.exec.timeoutSec` | `terminal.timeout` | 直接复制（字段是 `timeoutSec`，不是 `timeout`）|
| Docker sandbox | `agents.defaults.sandbox.backend` | `terminal.backend` | "docker" → "docker" |
| Docker image | `agents.defaults.sandbox.docker.image` | `terminal.docker_image` | 直接复制 |

---

### Session Reset Policies

| OpenClaw 配置路径 | Hermes 配置路径 | 说明 |
|-------------------|-----------------|------|
| `session.reset.mode` | `session_reset.mode` | "daily"、"idle" 或两者 |
| `session.reset.atHour` | `session_reset.at_hour` | 日重置小时（0–23）|
| `session.reset.idleMinutes` | `session_reset.idle_minutes` | 空闲分钟数 |

注意：OpenClaw 也有 `session.resetTriggers`（简单字符串数组如 `["daily", "idle"]`）。如果结构化 `session.reset` 不存在，迁移会回退到从 `resetTriggers` 推断。

---

### MCP Servers

| OpenClaw 字段 | Hermes 字段 | 说明 |
|---------------|-------------|------|
| `mcp.servers.*.command` | `mcp_servers.*.command` | stdio 传输 |
| `mcp.servers.*.args` | `mcp_servers.*.args` | — |
| `mcp.servers.*.env` | `mcp_servers.*.env` | — |
| `mcp.servers.*.cwd` | `mcp_servers.*.cwd` | — |
| `mcp.servers.*.url` | `mcp_servers.*.url` | HTTP/SSE 传输 |
| `mcp.servers.*.tools.include` | `mcp_servers.*.tools.include` | 工具过滤 |
| `mcp.servers.*.tools.exclude` | `mcp_servers.*.tools.exclude` | 工具过滤 |

---

### TTS (Text-to-Speech)

TTS 设置从**两个** OpenClaw 配置位置读取，优先级：

1. `messages.tts.provider` → `~/.hermes/.env` → `VOICE_PROVIDER`
2. `agents.defaults.tts.*` → 各字段映射

---

## 迁移后检查清单

- [ ] `hermes --version` 正常运行
- [ ] 模型正确响应
- [ ] Skills 已导入（`hermes skills list`）
- [ ] Memory 已迁移（`~/.hermes/memories/`）
- [ ] SOUL.md 正确（`~/.hermes/SOUL.md`）
- [ ] 消息平台（如果有）连接正常
- [ ] Cron 任务（如果有）正常工作
