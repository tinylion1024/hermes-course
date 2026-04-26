# 5.3 飞书/企业微信

## 飞书（Lark）配置

### 创建飞书 App

1. 访问 [飞书开放平台](https://open.feishu.cn/app)
2. 点击 **创建企业自建应用**
3. 填写应用信息
4. 获取 **App ID** 和 **App Secret**

### 配置权限

在应用后台添加以下权限：
- `im:message`
- `im:message.receive_v1`
- `im:chat`

### 获取 Webhook URL

1. 在应用后台创建 **Bot** 功能
2. 配置消息事件订阅
3. 获取 Webhook 地址

### 配置 Hermes

```yaml
feishu:
  enabled: true
  app_id: ${FEISHU_APP_ID}
  app_secret: ${FEISHU_APP_SECRET}
  bot_name: Hermes

  # Webhook 配置
  webhook:
    enabled: true
    url: https://your-domain.com/webhook/feishu
```

编辑 `~/.hermes/.env`：

```bash
FEISHU_APP_ID=cli_xxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxxxxx
```

---

## 企业微信配置

### 创建企业微信应用

1. 登录 [企业微信管理后台](https://work.weixin.qq.com)
2. 进入 **应用管理**
3. 创建应用，获取 **AgentId** 和 **Secret**

### 获取企业信息

- 企业 ID：在 **我的企业** 页面获取
- 应用 Secret：在应用详情页获取

### 配置 Hermes

```yaml
wecom:
  enabled: true
  corp_id: ${WECOM_CORP_ID}
  agent_id: ${WECOM_AGENT_ID}
  agent_secret: ${WECOM_AGENT_SECRET}

  # Webhook 配置（可选）
  webhook:
    enabled: true
    url: https://your-domain.com/webhook/wecom
```

编辑 `~/.hermes/.env`：

```bash
WECOM_CORP_ID=wwxxxxxxxxxxxxx
WECOM_AGENT_ID=1000001
WECOM_AGENT_SECRET=xxxxxxxxxxxxxxxxxx
```

---

## 飞书机器人使用

### 基本消息

```
用户 → 飞书消息 → Hermes Agent → 回复
```

### 支持的消息类型

| 类型 | 支持 |
|------|------|
| 文本消息 | ✅ |
| 图片消息 | ✅ |
| 命令 | ✅ |
| @Bot | ✅ |

### 命令

| 命令 | 说明 |
|------|------|
| `/help` | 显示帮助 |
| `/model` | 切换模型 |
| `/reset` | 重置会话 |

---

## 企业微信机器人使用

### 接收消息

企业微信应用消息发送到 Hermes：

1. 配置回调 URL
2. 启用应用消息接收
3. 设置 API 接收地址

### 发送消息

```yaml
wecom:
  # API 回调模式
  api:
    enabled: true
    callback_url: https://your-domain.com/wecom/callback
    aes_key: xxxxxxxxxx
    token: xxxxxxxxxx
```

---

## Webhook 配置（生产环境）

### Nginx 反向代理配置

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    location /webhook/feishu {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /webhook/wecom {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 飞书消息无响应 | 检查 Webhook URL 是否可达 |
| 企业微信无法接收 | 检查回调 URL 和 Token 验证 |
| 消息格式错误 | 检查 encoding 和加密配置 |
| 权限不足 | 添加必要的 API 权限 |

---

## 下一步

- [第六章：自动化](../ch06-automation/README.md) - 配置定时任务
- [第七章：高级应用](../ch07-advanced/README.md) - 多 Agent、API Server