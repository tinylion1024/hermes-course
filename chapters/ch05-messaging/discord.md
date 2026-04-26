# 5.2 Discord

## 概述

Discord 是功能最丰富的消息平台，支持文字频道、语音频道（VC）以及实时语音对话。

---

## 创建 Discord Application

### 1. 创建 Application

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 点击 **New Application**
3. 输入名称，点击 **Create**

### 2. 添加 Bot

1. 在左侧菜单选择 **Bot**
2. 点击 **Add Bot**
3. 记录 **Bot Token**（点击 Reset Token 生成）

### 3. 配置 Bot 权限

在 **OAuth2 > URL Generator** 中：

1. 选择 scopes：
   - ✅ `bot`
   - ✅ `applications.commands`

2. 选择 Bot Permissions：
   - ✅ Send Messages
   - ✅ Read Message History
   - ✅ Use Slash Commands
   - ✅ Connect（语音）
   - ✅ Speak（语音）

3. 复制生成的 URL 并访问授权

---

## 配置 Hermes

### 交互式配置

```bash
hermes gateway setup discord
```

### 手动配置

编辑 `~/.hermes/config.yaml`：

```yaml
discord:
  enabled: true
  bot_token: ${DISCORD_BOT_TOKEN}

  # 可选配置
  settings:
    # 命令前缀（默认 /）
    command_prefix: "/"
    # 是否回复自己
    self_reply: false
```

编辑 `~/.hermes/.env`：

```bash
DISCORD_BOT_TOKEN=ODY4xxxxxx.xxxxxx.xxxxxx
```

---

## 邀请 Bot

生成邀请链接：

```
https://discord.com/api/oauth2/authorize?client_id=YOUR_CLIENT_ID&permissions=PERMISSIONS&scope=bot
```

或使用 Discord Developer Portal 的 OAuth2 URL Generator。

---

## 使用方式

### 文字频道

| 命令 | 说明 |
|------|------|
| `@Bot` | 唤醒 Bot |
| `/help` | 显示帮助 |
| `/model` | 切换模型 |
| `/voice` | 语音模式 |

### 语音频道（VC）

1. 加入语音频道
2. Bot 会自动连接
3. 语音消息自动转文字并回复

---

## 高级配置

### 语音频道配置

```yaml
discord:
  voice:
    enabled: true
    # 加入后自动响应
    auto_respond: true
    # 说话人识别
    speaker_recognition: false
```

### 多服务器支持

```yaml
discord:
  # 允许的服务器 ID
  allowed_guilds:
    - 123456789012345678
  # 每服务器独立会话
  per_guild_sessions: true
```

---

## Discord + Voice Mode

语音模式完整配置：

```yaml
voice:
  enabled: true

discord:
  enabled: true
  bot_token: ${DISCORD_BOT_TOKEN}
  voice:
    enabled: true

tts:
  provider: minimax
  model: speech-02-hd
  voice: male-qn-qingse
```

### Discord 语音频道使用

1. 将 Bot 邀请到服务器
2. 加入语音频道
3. Bot 加入同一频道
4. 说话 → Bot 识别并语音回复

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| Bot 不在线 | 检查 Token 是否正确 |
| 无权响应 | 检查 App Permissions |
| 语音无响应 | 检查 Connect/Speak 权限 |
| 命令不工作 | 检查 `applications.commands` scope |

---

## 下一步

- [5.3 飞书/企业微信](./feishu.md) - 配置飞书
- [第六章：自动化](../ch06-automation/README.md) - 配置定时任务