# 4.5 语音模式

## 概述

Hermes Agent 支持**实时语音交互**，允许用户通过语音与 Agent 对话，无需打字。

---

## 支持平台

| 平台 | 支持状态 | 说明 |
|------|----------|------|
| CLI 终端 | ✅ 完全支持 | 在终端直接语音输入 |
| Telegram | ✅ 完全支持 | 语音消息交互 |
| Discord | ✅ 完全支持 | 文字频道语音 |
| Discord VC | ✅ 完全支持 | 语音频道实时对话 |

---

## 配置语音 Provider

### 支持的 TTS/STS 提供商

| 提供商 | 说明 | 需要 API Key |
|--------|------|-------------|
| OpenAI | GPT-4o Realtime API | ✅ OpenAI API Key |
| MiniMax | MiniMax TTS/STS | ✅ MiniMax API Key |
| ElevenLabs | 高质量语音 | ✅ ElevenLabs API Key |
| Azure TTS | Azure 语音服务 | ✅ Azure 账号 |

### 配置示例

```yaml
voice:
  enabled: true

  # TTS（文字转语音）
  tts:
    provider: minimax  # openai / minimax / elevenlabs / azure
    model: speech-02-hd  # 模型名称
    voice: male-qn-qingse  # 语音选择

  # STS（语音转语音）- 可选
  sts:
    provider: minimax
    model: speech-02-hd
```

### OpenAI 配置

```yaml
voice:
  tts:
    provider: openai
    model: gpt-4o-mini-realtime
    voice: alloy
```

### MiniMax 配置

```yaml
voice:
  tts:
    provider: minimax
    model: speech-02-hd
    voice: male-qn-qingse
    api_key: ${MINIMAX_API_KEY}
```

---

## CLI 语音使用

### 启用语音模式

```bash
# 启用语音
hermes voice enable

# 使用 TUI 界面
hermes --tui
```

### 在 TUI 中使用

1. 按下**麦克风按钮**激活语音输入
2. 对着麦克风说话
3. Agent 语音回复

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+V` | 开始/停止语音录制 |
| `Esc` | 取消语音输入 |
| `Space` | 在录制时暂停 |

---

## Telegram 语音配置

### 1. 设置 Telegram Bot

```bash
hermes gateway setup telegram
```

### 2. 配置 voice

```yaml
voice:
  enabled: true

  telegram:
    # Telegram Bot Token
    bot_token: ${TELEGRAM_BOT_TOKEN}

  tts:
    provider: minimax
    model: speech-02-hd
```

### 3. 使用

在 Telegram 中：
- 发送语音消息 → Agent 语音回复
- 发送文字消息 → Agent 文字回复

---

## Discord 语音配置

### 1. 创建 Discord Application

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 创建新 Application
3. 添加 Bot
4. 获取 Bot Token

### 2. 配置

```yaml
voice:
  enabled: true

  discord:
    bot_token: ${DISCORD_BOT_TOKEN}
    # 语音频道配置
    voice:
      enabled: true
      # 语音频道语言
      language: zh-CN
```

### 3. 邀请 Bot 到服务器

生成邀请链接：
```
https://discord.com/api/oauth2/authorize?client_id=YOUR_CLIENT_ID&permissions=PERMISSIONS&scope=bot
```

### 4. 使用

```
# 在文字频道
- 发送语音消息 → 语音回复
- @mention Bot → 语音回复

# 在语音频道（VC）
- 加入 VC → 自动接收语音输入
- 实时语音对话
```

---

## 语音模式架构

```
┌─────────────────────────────────────────────────────────────┐
│                     用户语音输入                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   语音识别（STT）                            │
│  ├── OpenAI Realtime API                                   │
│  ├── MiniMax STS                                           │
│  └── 第三方 ASR 服务                                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Hermes Agent 核心                         │
│  └── 处理语音输入，生成回复                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   语音合成（TTS）                            │
│  ├── MiniMax TTS                                           │
│  ├── OpenAI TTS                                            │
│  └── ElevenLabs                                            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     用户语音输出                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 高级配置

### 语音活性检测（VAD）

```yaml
voice:
  vad:
    # 语音活动检测阈值
    threshold: 0.5
    # 静音超时（秒）
    silence_timeout: 3
    # 最大录制时长（秒）
    max_duration: 60
```

### 音频格式

```yaml
voice:
  audio:
    # 采样率
    sample_rate: 24000
    # 声道数
    channels: 1
    # 编码格式
    codec: opus
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 语音无响应 | 检查麦克风权限 |
| TTS 不播放 | 检查音频输出设备 |
| Discord VC 无声 | 检查 Bot 权限和语音连接 |
| 语音延迟高 | 使用更近的服务器/本地 TTS |

---

## 最佳实践

### 1. 网络条件

语音模式对网络要求较高：
- 推荐延迟 < 200ms
- 丢包率 < 5%

### 2. 隐私考虑

> ⚠️ 语音消息会被录制和处理，请确保：
> - 在安静环境中使用
> - 避免分享敏感信息
> - 了解数据保留政策

### 3. 模型选择

不同 TTS 模型适合不同场景：

| 场景 | 推荐模型 |
|------|----------|
| 中文对话 | MiniMax speech-02-hd |
| 英文对话 | OpenAI alloy |
| 情感对话 | ElevenLabs |

---

## 下一步

- [第五章：消息平台](../ch05-messaging/README.md) - 配置更多消息平台
- [第六章：自动化](../ch06-automation/README.md) - 设置定时任务