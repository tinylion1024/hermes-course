# Integrations｜集成大全

> 原文：[Use MCP with Hermes](https://hermes-agent.nousresearch.com/docs/guides/use-mcp-with-hermes) + [Use SOUL.md with Hermes](https://hermes-agent.nousresearch.com/docs/guides/use-soul-with-hermes) + [Use Voice Mode with Hermes](https://hermes-agent.nousresearch.com/docs/guides/use-voice-mode-with-hermes) + [Build a Plugin](https://hermes-agent.nousresearch.com/docs/guides/build-a-hermes-plugin) + [Using Hermes as a Python Library](https://hermes-agent.nousresearch.com/docs/guides/python-library)

---

## 目录

- [MCP｜Model Context Protocol](#mcpmodel-context-protocol)
  - [启用 MCP 服务器](#启用-mcp-服务器)
  - [stdio vs HTTP/SSE 传输](#stdio-vs-httpsse-传输)
  - [工具过滤](#工具过滤)
  - [官方 MCP 服务器](#官方-mcp-服务器)
- [SOUL.md｜人格定制](#sould人格定制)
  - [创建 SOUL.md](#创建-souldmd)
  - [SOUL 与 AGENTS.md 的区别](#soul-与-agentsmd-的区别)
  - [工作原理](#工作原理)
- [Voice Mode｜语音模式](#voice-mode语音模式)
  - [配置语音](#配置语音)
  - [使用语音模式](#使用语音模式)
  - [语音设置](#语音设置)
- [Build a Plugin｜构建插件](#build-a-plugin构建插件)
  - [插件结构](#插件结构)
  - [定义工具](#定义工具)
  - [定义 Skills](#定义-skills)
  - [配置加载](#配置加载)
  - [安装插件](#安装插件)
- [Python Library｜Python 库](#python-librarypython-库)
  - [基础用法](#基础用法)
  - [流式响应](#流式响应)
  - [工具调用处理](#工具调用处理)

---

## MCP｜Model Context Protocol

MCP 让你将外部工具和数据源集成到 Hermes。通过在 `config.yaml` 中定义服务器来连接。

### 启用 MCP 服务器

```yaml
mcp_servers:
  # 本地 Python 服务器（stdio 传输）
  local-python:
    command: python
    args:
      - /path/to/server.py

  # 远程 HTTP 服务器
  remote-api:
    url: https://api.example.com/mcp
    headers:
      Authorization: "Bearer ${REMOTE_API_KEY}"

  # 带环境变量的服务器
  filesystem:
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-filesystem"
      - /tmp
    env:
      NODE_ENV: production
```

### stdio vs HTTP/SSE 传输

| 传输 | 配置方式 | 适用场景 |
|------|----------|----------|
| `stdio` | `command` + `args` | 本地进程 |
| `HTTP/SSE` | `url` + `headers` | 远程服务器 |

### 工具过滤

```yaml
mcp_servers:
  my-server:
    url: https://api.example.com/mcp
    tools:
      include:          # 只暴露这些工具
        - search
        - fetch
      exclude:          # 排除这些工具
        - admin
```

### 官方 MCP 服务器

```bash
# 文件系统
npx -y @modelcontextprotocol/server-filesystem /tmp

# Git
npx -y @modelcontextprotocol/server-github

# Slack
npx -y @modelcontextprotocol/server-slack
```

---

## SOUL.md｜人格定制

SOUL.md 为 Hermes 定义稳定的默认声音和性格。

### 创建 SOUL.md

```bash
# 编辑全局 SOUL
nano ~/.hermes/SOUL.md

# 或使用自定义 Hermes home
nano $HERMES_HOME/SOUL.md
```

**示例 SOUL.md**：

```markdown
# Soul

You are a senior backend engineer. Be terse and direct.
Skip explanations unless asked. Prefer one-liners over verbose solutions.
Always consider error handling and edge cases.
```

### SOUL 与 AGENTS.md 的区别

| 文件 | 用途 | 内容类型 |
|------|------|----------|
| `SOUL.md` | 持久化人格/语气 | 角色定义、交流风格 |
| `AGENTS.md` | 项目特定指令 | 架构、约定、项目规则 |

### 工作原理

- Hermes 自动使用 `~/.hermes/SOUL.md`（或 `$HERMES_HOME/SOUL.md`）
- 作为实例范围的默认人格来源
- 会话开始时加载到系统提示

---

## Voice Mode｜语音模式

Hermes 支持语音交互——说话而不是打字。

### 配置语音

```yaml
# ~/.hermes/.env
VOICE_PROVIDER=openai          # openai / elevenlabs / azure
VOICE_MODEL=tts-1              # TTS 模型
VOICE_VOICE=alloy              # 语音名称
VOICE_API_KEY=sk-...           # API key
```

### 使用语音模式

```bash
# 启动语音模式
hermes voice

# 在 TUI 中
# 按 V 键切换语音模式
```

### 语音设置

| 设置 | 说明 | 选项 |
|------|------|------|
| `VOICE_PROVIDER` | TTS 提供商 | `openai`, `elevenlabs`, `azure` |
| `VOICE_MODEL` | 模型 | `tts-1`, `tts-1-hd` (OpenAI) |
| `VOICE_VOICE` | 语音 | `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer` |

---

## Build a Plugin｜构建插件

插件扩展 Hermes 功能——添加新工具、Skills、平台适配器等。

### 插件结构

```
my-plugin/
├── plugin.json              # 插件元数据
├── src/
│   ├── __init__.py
│   └── tools.py             # 工具定义
├── skills/
│   └── my-skill/
│       └── SKILL.md
├── references/
│   └── api.md
└── scripts/
    └── setup.sh
```

### plugin.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My custom plugin",
  "tools": ["my_tool", "another_tool"],
  "skills": ["my-skill"],
  "entry": "src:setup"
}
```

### 定义工具

```python
# src/tools.py
from hermes import tool

@tool
def my_tool(query: str) -> str:
    """Search something in my API.
    
    Args:
        query: The search query
    """
    # 实现
    return result
```

### 定义 Skills

```markdown
# skills/my-skill/SKILL.md
---
name: my-skill
description: Does something useful
---

# My Skill

## When to Use
...

## Procedure
...
```

### 配置加载

```python
# src/__init__.py
def setup(config: dict):
    # 访问插件配置
    api_key = config.get("api_key")
    # 初始化资源
```

### 安装插件

```bash
hermes plugin install ./my-plugin

# 或从 URL
hermes plugin install https://github.com/user/hermes-plugin
```

---

## Python Library｜Python 库

将 Hermes 作为 Python 库集成到你的应用中。

### 基础用法

```python
from hermes import Hermes

client = Hermes()

response = client.chat("Explain quantum computing in simple terms")
print(response)
```

### 流式响应

```python
for chunk in client.stream("Write a story about AI"):
    print(chunk, end="", flush=True)
```

### 工具调用处理

```python
def calculator(expr: str) -> str:
    """Evaluate a math expression."""
    import eval as old_eval
    return str(old_eval(expr))

client = Hermes(tools=[calculator])

response = client.chat("What is 2 + 2? Use the calculator tool.")
print(response)
```

---

## 快速参考

| 集成 | 配置文件 | 说明 |
|------|----------|------|
| MCP | `config.yaml` → `mcp_servers` | 连接外部工具和数据源 |
| SOUL.md | `~/.hermes/SOUL.md` | 定义 Agent 人格 |
| Voice | `.env` → `VOICE_*` | 语音交互 |
| Plugin | `~/.hermes/plugins/` | 扩展功能 |
| Python | `from hermes import Hermes` | 应用集成 |
