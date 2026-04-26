# 从入门到精通的 Hermes Agent 课程

<div align="center">

![Hermes Agent](https://img.shields.io/badge/Hermes%20Agent-Nous%20Research-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Language](https://img.shields.io/badge/Language-中文-red)

**一个能够自我改进的 AI Agent，终身学习，持续进化**

</div>

---

## 📖 课程简介

Hermes Agent 是由 [Nous Research](https://nousresearch.com) 打造的自改进 AI Agent。它是**唯一一个内置学习循环的 Agent**——能够从经验中创建 Skills、在使用中不断改进、主动保存知识，并在跨会话中建立对用户的深入了解。

### 核心特性

- 🧠 **持久记忆** - 跨会话的持久化记忆系统
- 📚 **Skills 系统** - Agent 自我创建的程序化记忆，可复用
- 🔌 **MCP 集成** - 连接 MCP 服务器，安全扩展功能
- 🎙️ **语音模式** - 支持 CLI、Telegram、Discord 实时语音交互
- 💬 **多平台消息网关** - Telegram、Discord、Slack、飞书、企业微信等
- ⚡ **47 个内置工具** - 开箱即用的强大工具集
- 🚀 **60 秒安装** - Linux、macOS、WSL2 一键安装

---

## 📚 课程目录

### 第一部分：入门基础

| 章节 | 内容 | 难度 |
|------|------|------|
| [第一章：认识 Hermes Agent](./chapters/ch01-intro/README.md) | 基本概念、核心特性、应用场景 | ⭐ |
| [第二章：安装与环境配置](./chapters/ch02-install/README.md) | 安装、配置、AI Provider | ⭐ |
| [第三章：快速上手](./chapters/ch03-quickstart/README.md) | 首次对话、CLI/TUI、核心功能尝鲜 | ⭐ |

### 第二部分：核心功能

| 章节 | 内容 | 难度 |
|------|------|------|
| [第四章：核心功能](./chapters/ch04-features/README.md) | 工具、记忆、Skills、MCP、语音 | ⭐⭐ |
| [4.1 内置工具详解](./chapters/ch04-features/tools.md) | 47 个工具的使用方法 | ⭐⭐ |
| [4.2 记忆系统](./chapters/ch04-features/memory.md) | 持久化记忆与跨会话学习 | ⭐⭐ |
| [4.3 Skills 系统](./chapters/ch04-features/skills.md) | 程序化记忆的创建与复用 | ⭐⭐ |
| [4.4 MCP 集成](./chapters/ch04-features/mcp.md) | MCP 服务器连接与工具过滤 | ⭐⭐ |
| [4.5 语音模式](./chapters/ch04-features/voice-mode.md) | 实时语音交互配置 | ⭐⭐ |

### 第三部分：平台与自动化

| 章节 | 内容 | 难度 |
|------|------|------|
| [第五章：消息平台](./chapters/ch05-messaging/README.md) | 消息网关配置 | ⭐⭐ |
| [5.1 Telegram](./chapters/ch05-messaging/telegram.md) | Telegram Bot 集成 | ⭐⭐ |
| [5.2 Discord](./chapters/ch05-messaging/discord.md) | Discord Bot 集成 | ⭐⭐ |
| [5.3 飞书/企业微信](./chapters/ch05-messaging/feishu.md) | 飞书/企业微信配置 | ⭐⭐ |
| [第六章：自动化](./chapters/ch06-automation/README.md) | 定时任务与自动化 | ⭐⭐⭐ |
| [6.1 CRON 定时任务](./chapters/ch06-automation/cron.md) | 自动化任务配置 | ⭐⭐⭐ |
| [6.2 每日简报 Bot](./chapters/ch06-automation/daily-briefing-bot.md) | 实战：每日简报 Bot | ⭐⭐⭐ |

### 第四部分：高级应用

| 章节 | 内容 | 难度 |
|------|------|------|
| [第七章：高级应用](./chapters/ch07-advanced/README.md) | 多 Agent、API Server | ⭐⭐⭐ |
| [7.1 多 Agent 配置](./chapters/ch07-advanced/profiles.md) | Profiles 与多 Agent 管理 | ⭐⭐⭐ |
| [7.2 API Server](./chapters/ch07-advanced/api-server.md) | API 服务部署 | ⭐⭐⭐ |

---

## 🎯 学习路径

```
初学者路径：
  安装 → 快速上手 → 核心功能 → 消息平台 → 自动化

进阶路径：
  核心功能 → MCP 集成 → 多 Agent → API Server → 自定义扩展
```

---

## ⚡ 快速开始

### 60 秒安装

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### 首次对话

```bash
# 经典 CLI
hermes

# 推荐 TUI 界面
hermes --tui
```

### 选择 AI Provider

```bash
hermes model
```

---

## 📦 环境要求

- **系统**: Linux、macOS、WSL2、Android (Termux)
- **上下文窗口**: 最少 64K tokens
- **Python**: 3.10+
- **推荐**: 4GB+ RAM

---

## 🔗 参考资源

- 🌐 [Hermes Agent 官方文档](https://hermes-agent.nousresearch.com/docs/)
- 🐙 [GitHub 仓库](https://github.com/NousResearch/hermes-agent)
- 💬 [Discord 社区](https://discord.gg/NousResearch)
- 🐦 [Twitter / X](https://x.com/NousResearch)

---

## 📝 参与贡献

本课程内容基于 Hermes Agent 官方文档编写，欢迎提交 Issue 和 Pull Request！

---

<div align="center">

**Made with ❤️ by Hermes Agent Course Team**

</div>