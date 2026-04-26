# 第三章：快速上手

## 3.1 首次对话

### 启动 Hermes

```bash
# 经典 CLI 界面
hermes

# 推荐：TUI 界面（支持鼠标、模态框）
hermes --tui
```

### 首次运行预期

启动后会看到欢迎横幅，显示：
- 当前选定的模型/Provider
- 可用的工具列表
- 已加载的 Skills

### 测试提示词示例

```
总结这个代码库，用 5 个要点，并告诉我主入口文件是什么。
检查当前目录，告诉我看起来像主项目文件的是什么。
帮我为这个代码库设置一个干净的 GitHub PR 工作流。
```

### 验证成功标准

| 检查项 | 预期结果 |
|--------|----------|
| 横幅显示 | ✅ 显示模型和 Provider 信息 |
| 正常回复 | ✅ 无错误响应 |
| 工具调用 | ✅ 可以执行工具（如 terminal、file read） |
| 多轮对话 | ✅ 对话可以持续多轮 |

---

## 3.2 CLI vs TUI 界面

Hermes 提供两种终端界面：

### 经典 CLI（prompt_toolkit）

```bash
hermes
```

- 传统命令行界面
- 支持自动补全
- 轻量级

### TUI 界面（推荐）

```bash
hermes --tui
```

| 特性 | CLI | TUI |
|------|-----|-----|
| 模态叠加 | ❌ | ✅ |
| 鼠标选择 | ❌ | ✅ |
| 非阻塞输入 | ❌ | ✅ |
| 现代化体验 | ❌ | ✅ |

**推荐使用 TUI**——界面更现代，功能更丰富。

---

## 3.3 会话管理

### 继续上次会话

```bash
# 继续最近一次会话
hermes --continue

# 短格式
hermes -c
```

### 会话切换

```bash
# 列出所有会话
hermes sessions list

# 切换到指定会话
hermes sessions switch <session-id>
```

### 重要提示

> 💡 在移动到其他设置之前，确保 `hermes --continue` 能正常工作。如果不工作，检查是否在同一 Profile 下，以及会话是否实际保存了。

---

## 3.4 核心功能尝鲜

### 使用终端工具

```
❯ 我的磁盘使用情况如何？显示最大的 5 个目录。
```

Agent 会在你的机器上运行终端命令并展示结果。

### 斜杠命令（Slash Commands）

输入 `/` 查看所有可用命令：

| 命令 | 功能 |
|------|------|
| `/help` | 显示所有可用命令 |
| `/tools` | 列出可用工具 |
| `/model` | 交互式切换模型 |
| `/personality pirate` | 尝试有趣的个性 |
| `/save` | 保存当前对话 |

### 多行输入

- **Alt + Enter** 或 **Ctrl + J**：添加新行
- 适合粘贴代码或编写详细提示

### 中断 Agent

如果 Agent 运行时间过长：
- 直接输入新消息并回车——会中断当前任务
- **Ctrl + C** 也可以

---

## 3.5 配置详解

### hermes setup（快速设置）

```bash
hermes setup
```

交互式引导完成基本配置。

### hermes model（模型选择）

```bash
hermes model
```

交互式选择和配置 AI Provider。

### hermes config（配置管理）

```bash
# 查看所有配置
hermes config list

# 设置配置项
hermes config set model anthropic/claude-opus-4.6
hermes config set terminal.backend docker

# 查看特定配置
hermes config get model
```

---

## 3.6 Profiles（多 Agent 配置）

运行多个独立的 Agent 实例：

```bash
# 创建新 Profile
hermes profile create work

# 在指定 Profile 下运行
hermes --profile work

# 列出所有 Profile
hermes profile list
```

详细说明请参阅 [第七章：高级应用 - Profiles](../ch07-advanced/profiles.md)。

---

## 3.7 快速入门路径

根据你的目标选择：

| 目标 | 第一步 | 接下来 |
|------|--------|--------|
| 只想在本地跑起来 | `hermes setup` | 运行真实对话并验证 |
| 已有 Provider 账号 | `hermes model` | 保存配置，开始聊天 |
| 想做 Bot/常驻设置 | CLI 跑通后 `hermes gateway setup` | 连接 Telegram、Discord 等 |
| 想用本地/私有模型 | `hermes model` → custom endpoint | 验证端点、模型名、上下文长度 |
| 想要多 Provider 兜底 | 先跑通 `hermes model` | 基础聊天跑通后再加路由和兜底 |

> 📌 **经验法则**：如果 Hermes 连正常聊天都做不到，先不要添加更多功能。先让一个干净的对话跑起来，然后逐步叠加 gateway、cron、skills、voice、routing。

---

## 3.8 下一步

- [第四章：核心功能](../ch04-features/README.md) - 深入了解工具、记忆、Skills
- [第五章：消息平台](../ch05-messaging/README.md) - 连接 Telegram、Discord 等