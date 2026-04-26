# 第四章：核心功能

## 4.0 章节概述

本章详细介绍 Hermes Agent 的核心功能模块：

| 功能 | 说明 |
|------|------|
| [4.1 内置工具](./tools.md) | 47 个内置工具的详细使用指南 |
| [4.2 记忆系统](./memory.md) | 持久化记忆与跨会话学习 |
| [4.3 Skills 系统](./skills.md) | 程序化记忆的创建与复用 |
| [4.4 MCP 集成](./mcp.md) | 连接 MCP 服务器，安全扩展功能 |
| [4.5 语音模式](./voice-mode.md) | 实时语音交互配置与使用 |

---

## 4.1 内置工具详解

### 工具分类

Hermes Agent 内置 47 个工具，分为以下类别：

#### 核心工具（Core）

| 工具 | 功能 |
|------|------|
| `terminal` | 在本地/远程执行 shell 命令 |
| `read_file` | 读取文件内容 |
| `write_file` | 写入或创建文件 |
| `search_files` | 搜索文件内容和文件名 |
| `patch` | 智能查找替换 |
| `execute_code` | 执行 Python/JS 代码 |

#### 自动化工具（Automation）

| 工具 | 功能 |
|------|------|
| `cronjob` | 创建和管理定时任务 |
| `process` | 管理后台进程 |
| `webhook_subscriptions` | 创建 webhook 订阅 |

#### 媒体工具（Media）

| 工具 | 功能 |
|------|------|
| `browser_navigate` | 网页导航 |
| `browser_click` | 点击页面元素 |
| `browser_snapshot` | 获取页面快照 |
| `browser_vision` | 视觉分析截图 |
| `vision_analyze` | 分析图片内容 |
| `text_to_speech` | 文字转语音 |
| `image_gen` | 图片生成 |

#### 消息工具（Messaging）

| 工具 | 功能 |
|------|------|
| `send_message` | 发送消息到各平台 |
| `telegram_send` | 发送 Telegram 消息 |
| `discord_send` | 发送 Discord 消息 |
| `email_send` | 发送邮件 |

#### 数据工具（Data）

| 工具 | 功能 |
|------|------|
| `json_parse` | JSON 解析 |
| `csv_read` | CSV 文件读取 |
| `api_request` | HTTP API 请求 |

---

## 4.2 记忆系统

### 什么是记忆系统？

记忆系统是 Hermes Agent 的持久化存储层，允许 Agent：
- 跨会话记住用户偏好
- 积累项目上下文
- 建立用户画像

### 记忆类型

| 类型 | 说明 | 持久化 |
|------|------|--------|
| 工作记忆 | 当前会话的上下文 | ❌ 会话结束清除 |
| 持久记忆 | 跨会话的重要信息 | ✅ 持久化存储 |
| 程序记忆 | Skills 和工作流程 | ✅ 持久化存储 |

### 配置记忆

```yaml
memory:
  enabled: true
  persist: true
  # 记忆保留策略
  retention:
    short_term: 7d
    long_term: 90d
```

### 查看记忆

```bash
# 查看当前记忆内容
hermes memory list

# 查看特定类型的记忆
hermes memory list --type user_preference
```

### 管理记忆

```bash
# 添加记忆
hermes memory add "用户喜欢在早上处理复杂任务"

# 删除记忆
hermes memory delete <memory-id>

# 搜索记忆
hermes memory search "项目偏好"
```

---

## 4.3 Skills 系统

### 什么是 Skills？

Skills 是 Hermes Agent 自我创建的程序化记忆模块。它们：
- 从经验中自动生成
- 在使用中持续改进
- 可被新会话复用
- 用户可查看和编辑

### 创建 Skill

Skill 可以通过以下方式创建：

1. **Agent 自动创建**：Agent 认为某个工作流值得保存时自动创建
2. **手动创建**：用户通过 CLI 创建
3. **从模板创建**：基于现有模板创建

### Skill 结构

```
~/.hermes/skills/
├── skill-name/
│   ├── SKILL.md      # Skill 定义
│   ├── README.md     # 使用说明
│   ├── scripts/      # 辅助脚本
│   └── references/   # 参考资料
```

### SKILL.md 格式

```yaml
---
name: example-skill
description: 这个 Skill 做什么
---

# Example Skill

描述内容...
```

### 管理 Skills

```bash
# 列出所有 Skills
hermes skills list

# 查看 Skill 详情
hermes skills show <skill-name>

# 删除 Skill
hermes skills delete <skill-name>
```

---

## 4.4 MCP 集成

### 什么是 MCP？

MCP（Model Context Protocol）允许 Hermes Agent 连接外部服务器来扩展功能。

### 配置 MCP

编辑 `~/.hermes/config.yaml`：

```yaml
mcp:
  servers:
    - name: filesystem
      command: npx mcp-server-filesystem
      args:
        - /path/to/allowed/directory
    - name: github
      command: npx mcp-server-github
      env:
        GITHUB_TOKEN: your-token-here
```

### MCP 工具过滤

```yaml
mcp:
  servers:
    - name: github
      command: npx mcp-server-github
      # 只允许特定工具
      allowed_tools:
        - github_repos
        - github_issues
```

### 安全考虑

> ⚠️ MCP 可以执行本地命令，请：
> - 只允许可信的 MCP 服务器
> - 使用 `allowed_tools` 限制功能
> - 定期审查已安装的 MCP

---

## 4.5 语音模式

### 支持平台

- ✅ CLI 终端
- ✅ Telegram
- ✅ Discord
- ✅ Discord 语音频道（VC）

### 启用语音模式

```bash
hermes voice enable
```

### 配置语音 Provider

```yaml
voice:
  provider: openai  # 或 minimax、elevenlabs 等
  model: tts-1
  voice: alloy
```

### 使用语音

```
# 在 TUI 中按麦克风按钮
# 或发送语音消息
```

---

## 4.6 下一步

- [第五章：消息平台](../ch05-messaging/README.md) - 连接 Telegram、Discord
- [第六章：自动化](../ch06-automation/README.md) - 配置定时任务