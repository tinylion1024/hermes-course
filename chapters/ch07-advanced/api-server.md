# 7.2 API Server

## 概述

Hermes Agent 支持以 **API Server** 模式部署，通过 HTTP API 提供 AI 对话能力。

---

## 应用场景

| 场景 | 说明 |
|------|------|
| Web 应用集成 | 为 Web 应用提供 AI 对话后端 |
| 第三方调用 | 允许其他服务调用 Hermes |
| 移动应用 | 为移动 App 提供 API |
| 微服务架构 | 作为独立的 AI 微服务 |

---

## 启动 API Server

### 基本启动

```bash
# 启动 API Server
hermes api

# 指定端口
hermes api --port 8080

# 指定绑定地址
hermes api --host 0.0.0.0 --port 8080
```

### 后台运行

```bash
# 作为后台服务运行
hermes api --background

# 或使用 systemd/Nohup
nohup hermes api --port 8080 > api.log 2>&1 &
```

---

## API 端点

### 核心端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/v1/chat` | POST | 发送对话请求 |
| `/v1/completions` | POST | 补全请求 |
| `/v1/models` | GET | 列出可用模型 |
| `/health` | GET | 健康检查 |
| `/v1/sessions` | GET | 列出会话 |
| `/v1/sessions/{id}` | GET | 获取会话详情 |

### 请求示例

```bash
# 发送对话请求
curl -X POST http://localhost:8080/v1/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "anthropic/claude-opus-4.6",
    "messages": [
      {"role": "user", "content": "你好！"}
    ]
  }'
```

### 响应示例

```json
{
  "id": "chat-123456",
  "model": "anthropic/claude-opus-4.6",
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "你好！有什么可以帮助你的吗？"
      }
    }
  ]
}
```

---

## OpenAI 兼容模式

Hermes API Server 支持 **OpenAI API 兼容**接口，方便迁移：

```yaml
api:
  # OpenAI 兼容模式
  openai_compatible: true

  # API 路径前缀
  base_path: /v1
```

### OpenAI SDK 调用示例

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="http://localhost:8080/v1"
)

response = client.chat.completions.create(
    model="anthropic/claude-opus-4.6",
    messages=[
        {"role": "user", "content": "你好"}
    ]
)
```

---

## 认证配置

### API Key 认证

```yaml
api:
  auth:
    enabled: true
    type: api_key

  # 生成的 API Keys
  keys:
    - name: "web-app"
      key: "sk-hermes-xxxxx"
    - name: "mobile-app"
      key: "sk-hermes-yyyyy"
```

### 生成 API Key

```bash
# 生成新的 API Key
hermes api key generate --name my-app

# 列出所有 Key
hermes api key list

# 撤销 Key
hermes api key revoke <key-id>
```

---

## WebSocket 支持

```yaml
api:
  websocket:
    enabled: true
    path: /ws
```

### WebSocket 连接

```javascript
const ws = new WebSocket("ws://localhost:8080/ws?key=YOUR_API_KEY");

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: "chat",
    messages: [{role: "user", content: "你好"}]
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(data);
};
```

---

## 配置参考

### 完整配置示例

```yaml
api:
  enabled: true

  # 服务器配置
  server:
    host: 0.0.0.0
    port: 8080
    workers: 4

  # CORS 配置
  cors:
    enabled: true
    allowed_origins:
      - https://your-app.com
      - http://localhost:3000

  # 认证
  auth:
    enabled: true
    type: api_key
    keys:
      - name: default
        key: sk-hermes-xxxxx

  # 限流
  rate_limit:
    enabled: true
    requests_per_minute: 60

  # 日志
  logging:
    level: info
    format: json
```

---

## 反向代理配置

### Nginx 配置

```nginx
server {
    listen 443 ssl;
    server_name api.your-domain.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### SSL 证书

```bash
# Let's Encrypt 证书
certbot --nginx -d api.your-domain.com
```

---

## Docker 部署

### Dockerfile

```dockerfile
FROM nousresearch/hermes-agent:latest

EXPOSE 8080

CMD ["hermes", "api", "--host", "0.0.0.0", "--port", "8080"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  hermes-api:
    image: nousresearch/hermes-agent:latest
    ports:
      - "8080:8080"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    volumes:
      - ./config.yaml:/root/.hermes/config.yaml
```

---

## 监控与运维

### 健康检查

```bash
curl http://localhost:8080/health
```

响应：
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime": 3600
}
```

### 日志

```bash
# 实时日志
hermes api logs --follow

# 错误日志
hermes api logs --level error
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 端口被占用 | 更换端口或停止占用进程 |
| CORS 错误 | 配置 `allowed_origins` |
| 认证失败 | 检查 API Key |
| 连接超时 | 增加 timeout 配置 |

---

## 最佳实践

1. **始终启用 HTTPS**（生产环境）
2. **使用 API Key** 保护接口
3. **配置限流**防止滥用
4. **监控日志**及时发现问题
5. **定期轮换** API Keys

---

## 下一步

恭喜完成本课程！继续探索：

- [官方文档](https://hermes-agent.nousresearch.com/docs/)
- [GitHub 仓库](https://github.com/NousResearch/hermes-agent)
- [Discord 社区](https://discord.gg/NousResearch)