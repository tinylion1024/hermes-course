# 5.1 Telegram

## 概述

Telegram 是 Hermes Agent 最常用的消息平台之一，提供稳定的 Bot API 支持。

---

## 创建 Telegram Bot

### 1. 通过 BotFather 创建

1. 在 Telegram 中搜索 **@BotFather**
2. 发送 `/newbot`
3. 输入 Bot 名称（如 "My Hermes Bot"）
4. 输入用户名（必须以 `bot` 结尾，如 `myhermesbot`）
5. 复制获得的 **Bot Token**

### 2. 获取 Chat ID

1. 搜索并启动你的 Bot
2. 发送任意消息给 Bot
3. 访问：`https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
4. 找到 `"chat":{"id":...}` 中的数字，这就是你的 Chat ID

---

## 配置 Hermes

### 交互式配置

```bash
hermes gateway setup telegram
```

### 手动配置

编辑 `~/.hermes/config.yaml`：

```yaml
telegram:
  enabled: true
  bot_token: ${TELEGRAM_BOT_TOKEN}
  # 可选：限制可访问的 Chat ID
  allowed_chats:
    - 123456789  # 你的 Chat ID
    - -1001234567890  # 群组 ID
```

编辑 `~/.hermes/.env`：

```bash
TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```

---

## 使用方式

### 基本使用

```
用户 → Telegram 消息 → Hermes Agent → 回复
```

### 命令

| 命令 | 说明 |
|------|------|
| `/start` | 开始对话 |
| `/help` | 显示帮助 |
| `/model` | 切换模型 |
| `/voice` | 启用语音模式 |
| `/reset` | 重置对话 |

---

## 高级配置

### 群组支持

```yaml
telegram:
  # 群组模式
  group:
    enabled: true
    # 是否需要 @Bot
    require_at: false
    # 允许的群组
    allowed_groups:
      - -1001234567890
```

### 命令菜单

BotFather 支持自定义命令菜单：

```
/model - 切换 AI 模型
/voice - 语音模式
/skill - 管理技能
/memory - 查看记忆
```

### Webhook 模式（生产环境推荐）

```yaml
telegram:
  # Webhook 配置（生产环境推荐）
  webhook:
    enabled: true
    url: https://your-domain.com/webhook/telegram
    secret: your-webhook-secret
```

---

## Telegram Bot + Voice

结合语音模式：

```yaml
voice:
  enabled: true
  telegram:
    enabled: true
    bot_token: ${TELEGRAM_BOT_TOKEN}

tts:
  provider: minimax
  model: speech-02-hd
```

发送语音消息给 Bot → 收到语音回复

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| Bot 无响应 | 检查 bot_token 是否正确 |
| 无法接收消息 | 检查 Webhook/Polling 配置 |
| 权限错误 | Bot 需要 `send_messages` 权限 |
| 群组消息不处理 | 检查 `allowed_groups` 配置 |

---

## 下一步

- [5.2 Discord](./discord.md) - 配置 Discord Bot
- [5.3 飞书/企业微信](./feishu.md) - 配置飞书