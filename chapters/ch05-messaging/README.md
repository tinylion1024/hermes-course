# 第五章：消息平台

## 5.0 章节概述

Hermes Agent 提供了强大的**消息网关**功能，支持连接 20+ 消息平台。本章介绍主要平台的配置和使用。

| 平台 | 状态 | 说明 |
|------|------|------|
| [Telegram](./telegram.md) | ✅ 完全支持 | Bot 模式 |
| [Discord](./discord.md) | ✅ 完全支持 | Bot + 语音频道 |
| [飞书/Lark](./feishu.md) | ✅ 完全支持 | 机器人 + Webhook |
| [Slack](./slack.md) | ✅ 完全支持 | Slack App |
| [企业微信](./wecom.md) | ✅ 完全支持 | 企业自建应用 |
| [WhatsApp](./whatsapp.md) | ✅ 完全支持 | WhatsApp Business |
| [Email](./email.md) | ✅ 完全支持 | IMAP/SMTP |
| [SMS](./sms.md) | ✅ 支持 | Twilio |

---

## 消息网关架构

```
┌─────────────────────────────────────────────────────────────┐
│                   Hermes Agent                               │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Message Gateway                             ││
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                ││
│  │  │ Telegram │ │ Discord  │ │  Feishu  │  ...           ││
│  │  │  Adapter │ │  Adapter │ │  Adapter │                ││
│  │  └──────────┘ └──────────┘ └──────────┘                ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

## 统一配置

### config.yaml

```yaml
gateway:
  # 启用消息网关
  enabled: true

  # 默认回复设置
  default:
    # 回复超时（秒）
    reply_timeout: 30
    # 最大重试次数
    max_retries: 3

  # 过滤设置
  filters:
    # 忽略的消息类型
    ignore:
      - read_receipt
      - typing
```

---

## 下一步

- [5.1 Telegram](./telegram.md) - Telegram Bot 配置
- [5.2 Discord](./discord.md) - Discord Bot 配置
- [5.3 飞书/企业微信](./feishu.md) - 飞书配置