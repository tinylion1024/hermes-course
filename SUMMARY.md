# 课程目录

## 快速导航

### [第一章：认识 Hermes Agent](./chapters/ch01-intro/README.md)
- [1.1 什么是 Hermes Agent](./chapters/ch01-intro/README.md#11-什么是-hermes-agent)
- [1.2 核心特性](./chapters/ch01-intro/README.md#12-核心特性)
- [1.3 工作原理](./chapters/ch01-intro/README.md#13-工作原理)
- [1.4 应用场景](./chapters/ch01-intro/README.md#14-应用场景)
- [1.5 为什么选择 Hermes Agent](./chapters/ch01-intro/README.md#15-为什么选择-hermes-agent)

### [第二章：安装与环境配置](./chapters/ch02-install/README.md)
- [2.1 快速安装](./chapters/ch02-install/README.md#21-快速安装60-秒)
- [2.2 Android / Termux 安装](./chapters/ch02-install/README.md#22-android--termux-安装)
- [2.3 Nix / NixOS 安装](./chapters/ch02-install/README.md#23-nix--nixos-安装)
- [2.4 Docker 安装](./chapters/ch02-install/README.md#24-docker-安装)
- [2.5 配置 AI Provider](./chapters/ch02-install/README.md#25-配置-ai-provider)
- [2.6 配置文件结构](./chapters/ch02-install/README.md#26-配置文件结构)
- [2.7 验证安装](./chapters/ch02-install/README.md#27-验证安装)
- [2.8 故障排除](./chapters/ch02-install/README.md#28-故障排除)

### [第三章：快速上手](./chapters/ch03-quickstart/README.md)
- [3.1 首次对话](./chapters/ch03-quickstart/README.md#31-首次对话)
- [3.2 CLI vs TUI 界面](./chapters/ch03-quickstart/README.md#32-cli-vs-tui-界面)
- [3.3 会话管理](./chapters/ch03-quickstart/README.md#33-会话管理)
- [3.4 核心功能尝鲜](./chapters/ch03-quickstart/README.md#34-核心功能尝鲜)
- [3.5 配置详解](./chapters/ch03-quickstart/README.md#35-配置详解)
- [3.6 Profiles](./chapters/ch03-quickstart/README.md#36-profiles多-agent-配置)
- [3.7 快速入门路径](./chapters/ch03-quickstart/README.md#37-快速入门路径)

### [第四章：核心功能](./chapters/ch04-features/README.md)

#### [4.1 内置工具详解](./chapters/ch04-features/tools.md)
- [核心工具](./chapters/ch04-features/tools.md#核心工具core-tools)
- [自动化工具](./chapters/ch04-features/tools.md#自动化工具automation)
- [媒体工具](./chapters/ch04-features/tools.md#媒体工具media)
- [消息工具](./chapters/ch04-features/tools.md#消息工具messaging)
- [数据工具](./chapters/ch04-features/tools.md#数据工具data)

#### [4.2 记忆系统](./chapters/ch04-features/memory.md)
- [记忆类型](./chapters/ch04-features/memory.md#记忆类型)
- [配置记忆系统](./chapters/ch04-features/memory.md#配置记忆系统)
- [CLI 命令](./chapters/ch04-features/memory.md#cli-命令)
- [最佳实践](./chapters/ch04-features/memory.md#最佳实践)

#### [4.3 Skills 系统](./chapters/ch04-features/skills.md)
- [Skills vs 记忆](./chapters/ch04-features/skills.md#skills-vs-记忆)
- [Skills 的结构](./chapters/ch04-features/skills.md#skills-的结构)
- [创建 Skills](./chapters/ch04-features/skills.md#创建-skills)
- [管理 Skills](./chapters/ch04-features/skills.md#管理-skills)
- [Skill 触发器](./chapters/ch04-features/skills.md#skill-触发器triggers)

#### [4.4 MCP 集成](./chapters/ch04-features/mcp.md)
- [MCP 工作原理](./chapters/ch04-features/mcp.md#mcp-工作原理)
- [MCP 服务器](./chapters/ch04-features/mcp.md#mcp-服务器)
- [配置 MCP](./chapters/ch04-features/mcp.md#配置-mcp)
- [安全最佳实践](./chapters/ch04-features/mcp.md#mcp-安全最佳实践)

#### [4.5 语音模式](./chapters/ch04-features/voice-mode.md)
- [支持平台](./chapters/ch04-features/voice-mode.md#支持平台)
- [配置语音 Provider](./chapters/ch04-features/voice-mode.md#配置语音-provider)
- [CLI 语音使用](./chapters/ch04-features/voice-mode.md#cli-语音使用)
- [Telegram 语音配置](./chapters/ch04-features/voice-mode.md#telegram-语音配置)
- [Discord 语音配置](./chapters/ch04-features/voice-mode.md#discord-语音配置)

### [第五章：消息平台](./chapters/ch05-messaging/README.md)
- [5.0 章节概述](./chapters/ch05-messaging/README.md#50-章节概述)
- [5.1 Telegram](./chapters/ch05-messaging/telegram.md)
- [5.2 Discord](./chapters/ch05-messaging/discord.md)
- [5.3 飞书/企业微信](./chapters/ch05-messaging/feishu.md)

### [第六章：自动化](./chapters/ch06-automation/README.md)
- [6.0 章节概述](./chapters/ch06-automation/README.md#60-章节概述)
- [6.1 CRON 定时任务](./chapters/ch06-automation/cron.md)
- [6.2 每日简报 Bot](./chapters/ch06-automation/daily-briefing-bot.md)

### [第七章：高级应用](./chapters/ch07-advanced/README.md)
- [7.0 章节概述](./chapters/ch07-advanced/README.md#70-章节概述)
- [7.1 多 Agent 配置（Profiles）](./chapters/ch07-advanced/profiles.md)
- [7.2 API Server](./chapters/ch07-advanced/api-server.md)

---

## 学习路径推荐

```
初学者路径：
第一章 → 第二章 → 第三章 → 第四章（工具+记忆）→ 第五章（Telegram）

进阶路径：
第四章（完整）→ 第五章（多平台）→ 第六章（自动化）→ 第七章（高级）

开发者路径：
第四章（MCP）→ 第七章（API Server）→ 自定义扩展开发
```