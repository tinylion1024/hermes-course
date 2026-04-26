# Tips & Best Practices｜效率提升实战指南

> 原文：[Tips & Best Practices](https://hermes-agent.nousresearch.com/docs/guides/tips) — 快速胜利集合，让 Hermes Agent 使用效率立即提升。

---

## 目录

- [Getting the Best Results｜获得最佳结果](#getting-the-best-results获得最佳结果)
  - [Be Specific About What You Want｜明确表达需求](#be-specific-about-what-you-want明确表达需求)
  - [Provide Context Up Front｜前置所有上下文](#provide-context-up-front前置所有上下文)
  - [Use Context Files for Recurring Instructions｜用上下文文件管理重复指令](#use-context-files-管理重复指令)
  - [Let the Agent Use Its Tools｜让 Agent 自主使用工具](#let-the-agent-use-its-tools让-agent-自主使用工具)
  - [Use Skills for Complex Workflows｜用 Skills 处理复杂工作流](#use-skills-处理复杂工作流)
- [CLI Power User Tips｜命令行高效技巧](#cli-power-user-tips命令行高效技巧)
  - [Multi-Line Input｜多行输入](#multi-line-input多行输入)
  - [Paste Detection｜粘贴检测](#paste-detection粘贴检测)
  - [Interrupt and Redirect｜中断与重定向](#interrupt-and-redirect中断与重定向)
  - [Resume Sessions with -c｜恢复会话](#resume-sessions-with--c恢复会话)
  - [Clipboard Image Paste｜剪贴板图片粘贴](#clipboard-image-paste剪贴板图片粘贴)
  - [Slash Command Autocomplete｜斜杠命令自动补全](#slash-command-autocomplete斜杠命令自动补全)
  - [Verbose Tool Output｜工具输出模式](#verbose-tool-output工具输出模式)
- [Context Files｜上下文文件](#context-files上下文文件)
  - [AGENTS.md｜项目大脑](#agentsmd项目大脑)
  - [SOUL.md｜人格定制](#sould个性化配置)
  - [.cursorrules 兼容性](#cursorrules-兼容性)
  - [Discovery 机制](#discovery-机制)
- [Memory & Skills｜记忆与技能](#memory--skills记忆与技能)
  - [Memory vs. Skills: What Goes Where｜职责划分](#memory-vs-skills-what-goes-where职责划分)
  - [When to Create Skills｜何时创建技能](#when-to-create-skills何时创建技能)
  - [Managing Memory Capacity｜管理记忆容量](#managing-memory-capacity管理记忆容量)
  - [Let the Agent Remember｜让 Agent 记住](#let-the-agent-remember让-agent-记住)
- [Performance & Cost｜性能与成本](#performance--cost性能与成本)
  - [Don't Break the Prompt Cache｜保持提示缓存](#dont-break-the-prompt-cache保持提示缓存)
  - [Use /compress Before Hitting Limits｜限额前压缩](#use-compress-before-hitting-limits限额前压缩)
  - [Delegate for Parallel Work｜并行委托](#delegate-for-parallel-work并行委托)
  - [Use execute_code for Batch Operations｜批处理脚本](#use-execute_code-for-batch-operations批处理脚本)
  - [Choose the Right Model｜选择合适的模型](#choose-the-right-model选择合适的模型)
- [Messaging Tips｜消息平台技巧](#messaging-tips消息平台技巧)
  - [Set a Home Channel｜设置主频道](#set-a-home-channel设置主频道)
  - [Use /title to Organize Sessions｜用标题组织会话](#use-title-to-organize-sessions用标题组织会话)
  - [DM Pairing for Team Access｜DM 配对授权](#dm-pairing-for-team-accessdm-配对授权)
  - [Tool Progress Display Modes｜工具进度显示模式](#tool-progress-display-modes工具进度显示模式)
- [Security｜安全建议](#security安全建议)
  - [Use Docker for Untrusted Code｜沙箱运行不受信代码](#use-docker-for-untrusted-code沙箱运行不受信代码)
  - [Avoid Windows Encoding Pitfalls｜避免 Windows 编码陷阱](#avoid-windows-encoding-pitfalls避免-windows-编码陷阱)
  - [Review Before Choosing "Always"｜慎选"始终允许"](#review-before-choosing-always慎选始终允许)
  - [Command Approval Is Your Safety Net｜命令审批是安全网](#command-approval-is-your-safety-net命令审批是安全网)
  - [Use Allowlists for Messaging Bots｜消息 Bot 用白名单](#use-allowlists-for-messaging-bots消息-bot-用白名单)

---

## Getting the Best Results｜获得最佳结果

### Be Specific About What You Want｜明确表达需求

模糊的提示产生模糊的结果。

```diff
- "fix the code"
+ "fix the TypeError in api/handlers.py on line 47 — the
+  process_request() function receives None from parse_body()."
```

**越多的上下文意味着越少的迭代次数。**

---

### Provide Context Up Front｜前置所有上下文

一次性提供所有相关信息：**文件路径、错误信息、预期行为**。

> 一条精心设计的消息胜过三轮澄清。把错误追踪直接粘贴进来——Agent 可以直接解析。

---

### Use Context Files｜管理重复指令 {#use-context-files-管理重复指令}

如果发现自己总在重复同样的指令，将它们写入 `AGENTS.md` 文件：

```bash
# 项目根目录下的 AGENTS.md 示例
# Project Context
- This is a FastAPI backend with SQLAlchemy ORM
- Always use async/await for database operations
- Tests go in tests/ and use pytest-asyncio
- Never commit .env files
```

Agent **自动在每个会话中读取**，零额外操作。

---

### Let the Agent Use Its Tools｜让 Agent 自主使用工具

不要手把手指导每一个步骤：

```diff
- "open tests/test_foo.py, look at line 42, then fix the bug..."
+ "find and fix the failing test"
```

Agent 拥有文件搜索、终端访问和代码执行能力——让它自行探索和迭代。

---

### Use Skills｜处理复杂工作流 {#use-skills-处理复杂工作流}

在写一个很长的提示之前，检查是否已有现成的 Skill：

```bash
/skills                    # 浏览所有可用技能
/axolotl                   # 直接调用
/github-pr-workflow        # 直接调用
```

---

## CLI Power User Tips｜命令行高效技巧

### Multi-Line Input｜多行输入

| 系统 | 快捷键 |
|------|--------|
| macOS / Linux | `Alt+Enter` 或 `Ctrl+J` |
| 通用 | `Ctrl+J` |

插入换行而不发送，编写多行提示、粘贴代码块或构建复杂请求。

---

### Paste Detection｜粘贴检测

CLI 自动检测多行粘贴。直接粘贴代码块或错误追踪——不会逐行发送。粘贴被缓冲后作为一条消息发送。

---

### Interrupt and Redirect｜中断与重定向

| 操作 | 效果 |
|------|------|
| `Ctrl+C` 一次 | 中断 Agent 当前响应，可输入新消息重定向 |
| 2 秒内 `Ctrl+C` 两次 | 强制退出 |

---

### Resume Sessions｜恢复会话 {#resume-sessions-with--c恢复会话}

```bash
hermes -c                    # 恢复到上次停止的地方
hermes -r "my project"      # 按标题恢复
```

---

### Clipboard Image Paste｜剪贴板图片粘贴

`Ctrl+V` 直接将图片从剪贴板粘贴到聊天。Agent 用视觉分析截图、图表、错误弹窗或 UI 模型——无需先保存到文件。

---

### Slash Command Autocomplete｜斜杠命令自动补全

输入 `/` 然后按 `Tab` 查看所有可用命令——包括内置命令和每个已安装的 Skill。无需记忆——Tab 自动补全。

---

### Verbose Tool Output｜工具输出模式 {#verbose-tool-output工具输出模式}

```bash
/verbose    # 循环切换：off → new → all → verbose
```

| 模式 | 适用场景 |
|------|----------|
| `off` | 简单问答，最简洁 |
| `new` | 消息平台，只看新工具调用 |
| `all` | CLI，观看 Agent 所做的一切 |
| `verbose` | 详细调试 |

---

## Context Files｜上下文文件

### AGENTS.md｜项目大脑

在项目根目录创建 `AGENTS.md`，包含架构决策、编码约定和项目特定指令。它会**自动注入每个会话**。

```markdown
# Project Context
- This is a FastAPI backend with SQLAlchemy ORM
- Always use async/await for database operations
- Tests go in tests/ and use pytest-asyncio
- Never commit .env files
```

---

### SOUL.md｜个性化配置 {#sould个性化配置}

编辑 `~/.hermes/SOUL.md`（或 `$HERMES_HOME/SOUL.md`）来设置 Agent 的稳定默认语气。

```markdown
# Soul
You are a senior backend engineer. Be terse and direct.
Skip explanations unless asked. Prefer one-liners over verbose solutions.
Always consider error handling and edge cases.
```

| 文件 | 用途 |
|------|------|
| `SOUL.md` | 持久化人格/语气 |
| `AGENTS.md` | 项目特定指令 |

完整指南见 [Use SOUL.md with Hermes](./integrations.md#use-soul-with-hermes)。

---

### .cursorrules 兼容性

已有 `.cursorrules` 或 `.cursor/rules/*.mdc` 文件？Hermes 也会读取它们。无需重复——从工作目录自动加载。

---

### Discovery 机制

- **顶层 `AGENTS.md`**：在会话开始时从当前工作目录加载
- **子目录 `AGENTS.md`**：通过工具调用懒加载（`subdirectory_hints.py`），**不**预先加载到系统提示

> ⚠️ **提示**：保持上下文文件专注简洁。每个字符都计入 token 预算，因为它们被注入到**每一条**消息中。

---

## Memory & Skills｜记忆与技能

### Memory vs. Skills｜职责划分 {#memory-vs-skills-what-goes-where}

| 类型 | 用途 | 例子 |
|------|------|------|
| **Memory** | 事实 | 环境、偏好、项目位置、已学到的信息 |
| **Skills** | 流程 | 多步骤工作流、工具特定指令、可复用配方 |

> 用 Memory 存储"是什么"，用 Skills 存储"怎么做"。

---

### When to Create Skills｜何时创建技能

如果某个任务需要 5+ 步骤且你会重复做：

> "save what you just did as a skill called **deploy-staging**"

下次只需输入 `/deploy-staging`，Agent 加载完整流程。

---

### Managing Memory Capacity｜管理记忆容量

Memory 有意设置了上限：
- `MEMORY.md`：约 2,200 字符
- `USER.md`：约 1,375 字符

满了之后 Agent 会合并条目。也可以主动说：
- "clean up your memory"
- "replace the old Python 3.9 note — we're on 3.12 now"

> ⚠️ **警告**：Memory 是冻结快照——会话期间所做的更改不会出现在系统提示中，直到下一个会话开始。Agent 会立即写入磁盘，但提示缓存不会在会话中失效。

---

### Let the Agent Remember｜让 Agent 记住 {#let-the-agent-remember让-agent-记住}

在一个高效会话之后，说：

> "remember this for next time"

Agent 会保存关键要点。也可以具体指定：

> "save to memory that our CI uses GitHub Actions with the deploy.yml workflow"

---

## Performance & Cost｜性能与成本

### Don't Break the Prompt Cache｜保持提示缓存 {#dont-break-the-prompt-cache保持提示缓存}

大多数 LLM Provider 会缓存系统提示前缀。如果保持系统提示稳定（相同的上下文文件、相同的记忆），会话中的后续消息会获得缓存命中，大幅降低成本。

> 避免在会话中更改模型或系统提示。

---

### Use /compress｜限额前压缩 {#use-compress-before-hitting-limits压缩}

长会话会累积 token。当注意到响应变慢或被截断时，运行 `/compress`。这会总结会话历史，保留关键上下文，同时大幅减少 token 数量。

```bash
/usage    # 查看当前 token 消耗
/insights # 查看过去 30 天的使用模式
```

---

### Delegate for Parallel Work｜并行委托 {#delegate-for-parallel-work并行委托}

需要同时研究三个主题？让 Agent 使用 `delegate_task` 并行委托任务。每个子 Agent 独立运行，只有最终摘要返回——大幅减少主会话的 token 使用。

---

### execute_code｜批处理脚本 {#use-execute_code-for-batch-operations批处理脚本}

不要一个一个运行终端命令，让 Agent 写一个一次性完成的脚本：

> "Write a Python script to rename all .jpeg files to .jpg and run it"

更便宜、更快速。

---

### Choose the Right Model｜选择合适的模型 {#choose-the-right-model选择合适的模型}

用 `/model` 在会话中切换模型：

| 场景 | 推荐模型 |
|------|----------|
| 复杂推理和架构决策 | Claude Sonnet/Opus, GPT-4o |
| 简单任务（格式化、重命名、样板生成） | 更快的小模型 |

> 💡 定期运行 `/usage` 查看 token 消耗。运行 `/insights` 查看过去 30 天的使用模式。

---

## Messaging Tips｜消息平台技巧

### Set a Home Channel｜设置主频道 {#set-a-home-channel设置主频道}

用 `/sethome` 在你首选的 Telegram 或 Discord 聊天中将其设为主频道。Cron 任务结果和计划任务输出会发送到这里。没有它，Agent 没有地方发送主动消息。

---

### Use /title｜用标题组织会话 {#use-title-to-organize-sessions用标题组织会话}

用 `/title auth-refactor` 或 `/title research-llm-quantization` 命名会话。

命名会话易于通过 `hermes sessions list` 查找，通过 `hermes -r "auth-refactor"` 恢复。未命名会话堆积后无法区分。

---

### DM Pairing｜DM 配对授权 {#dm-pairing-for-team-accessdm-配对授权}

不想手动收集用户 ID？启用 DM 配对。当队友 DM Bot 时，他们会收到一次性配对码。你用 `hermes pairing approve telegram XKGH5N7P` 审批——简单安全。

---

### Tool Progress Display Modes｜工具进度显示模式 {#tool-progress-display-modes工具进度显示模式}

用 `/verbose` 控制工具活动显示多少内容。

在消息平台上，少即是多——保持"new"模式只显示新工具调用。在 CLI 中，"all"给你满足感十足的全览。

> 💡 在消息平台上，会话在空闲时间后自动重置（默认：24 小时）或每天凌晨 4 点。如需更长会话，在 `~/.hermes/config.yaml` 中调整。

---

## Security｜安全建议

### Docker for Untrusted Code｜沙箱运行不受信代码 {#use-docker-for-untrusted-code沙箱运行不受信代码}

处理不受信任的仓库或运行不熟悉的代码时，用 Docker 或 Daytona 作为终端后端：

```bash
# 在 .env 中：
TERMINAL_BACKEND=docker
TERMINAL_DOCKER_IMAGE=hermes-sandbox:latest
```

容器内的破坏性命令无法伤害你的主机系统。

---

### Avoid Windows Encoding Pitfalls｜避免 Windows 编码陷阱 {#avoid-windows-encoding-pitfalls避免-windows-编码陷阱}

Windows 某些默认编码（如 cp125x）无法表示所有 Unicode 字符，可能导致写入文件时出现 `UnicodeEncodeError`。

始终使用显式 UTF-8 编码打开文件：

```python
with open("results.txt", "w", encoding="utf-8") as f:
    f.write("✓ All good\n")
```

在 PowerShell 中切换到 UTF-8：

```powershell
$OutputEncoding = [Console]::OutputEncoding = [Text.UTF8Encoding]::new($false)
```

---

### Review Before Choosing "Always"｜慎选"始终允许" {#review-before-choosing-always慎选始终允许}

当 Agent 触发危险命令审批（`rm -rf`、`DROP TABLE` 等）时，你有四个选项：**once、session、always、deny**。

在确认前谨慎考虑"always"——这会永久允许该模式。从"session"开始，直到你感到舒适。

---

### Command Approval Is Your Safety Net｜命令审批是安全网 {#command-approval-is-your-safety-net命令审批是安全网}

Hermes 在执行前检查每条命令是否匹配危险模式列表。包括递归删除、SQL drops、pipe curl 到 shell 等。

> ⚠️ **不要在生产环境禁用它**——它存在有充分的理由。

> ⚠️ **警告**：在容器后端（Docker、Singularity、Modal、Daytona）运行时会跳过危险命令检查，因为容器是安全边界。确保容器镜像正确锁定。

---

### Use Allowlists for Messaging Bots｜消息 Bot 用白名单 {#use-allowlists-for-messaging-bots消息-bot-用白名单}

永远不要在有终端访问的 Bot 上设置 `GATEWAY_ALLOW_ALL_USERS=true`。

```bash
# 推荐：每个平台显式白名单
TELEGRAM_ALLOWED_USERS=123456789,987654321
DISCORD_ALLOWED_USERS=123456789012345678

# 或使用跨平台白名单
GATEWAY_ALLOWED_USERS=123456789,987654321
```

---

## 快速参考卡

| 类别 | 技巧 |
|------|------|
| **效果提升** | 明确需求、前置上下文、使用 AGENTS.md |
| **CLI 技巧** | `Alt+Enter` 多行、`Ctrl+C` 中断、`-c` 恢复会话 |
| **上下文文件** | AGENTS.md（项目）、SOUL.md（人格）|
| **记忆管理** | Memory 存事实、Skills 存流程、5+ 步骤创建 Skill |
| **性能优化** | 保持提示缓存稳定、`/compress` 压缩、`/model` 选对模型 |
| **消息平台** | `/sethome`、`/title` 命名、DM 配对授权 |
| **安全** | Docker 沙箱、Windows UTF-8、白名单授权 |
