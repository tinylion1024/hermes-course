# 4.3 Skills 系统

## 什么是 Skills？

Skills 是 Hermes Agent 的**程序化记忆**系统。不同于记忆系统存储原始信息，Skills 存储的是**可复用的工作流程和最佳实践**。

```
记忆（Memory）    → 记住"用户用 Python"
Skills          → 记住"如何用 Python 最佳实践写 API"
```

---

## Skills vs 记忆

| 特性 | 记忆 (Memory) | Skills |
|------|--------------|--------|
| 存储内容 | 原始信息、偏好、事实 | 工作流程、模板、最佳实践 |
| 创建方式 | 自动+手动 | 通常由 Agent 自动创建 |
| 使用方式 | 检索后使用 | 被 Agent 调用执行 |
| 粒度 | 细粒度（条目） | 粗粒度（工作流） |
| 格式 | YAML/JSON | Markdown + 代码 |

---

## Skills 的结构

```
~/.hermes/skills/
├── skill-name/
│   ├── SKILL.md           # Skill 定义（必须）
│   ├── README.md          # 使用文档
│   ├── scripts/           # 辅助脚本
│   │   ├── setup.sh
│   │   └── validate.py
│   └── references/        # 参考资料
│       ├── api.md
│       └── examples/
```

### SKILL.md 格式

```yaml
---
name: python-api-best-practices
description: 快速创建 Python REST API 的最佳实践
author: hermes-agent
version: 1.2.0
tags: [python, api, fastapi, backend]
triggers:
  - "写 python api"
  - "创建 fastapi"
  - "python rest"
---

# Python API Best Practices

本 Skill 提供创建高质量 Python REST API 的模板和检查清单。

## 1. 项目结构

推荐结构：

\`\`\`
project/
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── routers/
│   ├── models/
│   └── schemas/
├── tests/
├── requirements.txt
└── README.md
\`\`\`

## 2. FastAPI 最小模板

\`\`\`python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}
\`\`\`

## 3. 检查清单

- [ ] 使用 Pydantic 做请求/响应验证
- [ ] 添加 OpenAPI 文档
- [ ] 错误处理
- [ ] 日志记录
```

---

## 创建 Skills

### 1. Agent 自动创建

当 Agent 发现某个工作流值得复用时，会自动创建 Skill：

```
用户: 帮我写一个 GitHub PR 审核的工作流
Agent: [完成工作流后]
Agent: 这个工作流很通用，我来保存为 Skill...
[Skill "github-pr-review" 已创建]
```

### 2. 手动创建

```bash
# 创建新 Skill
hermes skills create --name my-custom-skill --description "我的自定义技能"

# 从模板创建
hermes skills create --name api-skill --template rest-api
```

### 3. 导入他人分享的 Skill

```bash
# 从 URL 导入
hermes skills import https://example.com/skill.tar.gz

# 从文件导入
hermes skills import /path/to/skill.tar.gz
```

---

## 管理 Skills

### 列出 Skills

```bash
# 列出所有 Skills
hermes skills list

# 查看详情
hermes skills show <skill-name>

# 按标签筛选
hermes skills list --tag python
```

### 更新 Skill

```bash
# 编辑 Skill
hermes skills edit <skill-name>

# 更新版本
hermes skills bump <skill-name> --version 1.3.0
```

### 删除 Skill

```bash
hermes skills delete <skill-name>
```

### 导出/分享

```bash
# 导出 Skill
hermes skills export <skill-name> --output skill.tar.gz

# 分享到社区（需要配置）
hermes skills publish <skill-name>
```

---

## 使用 Skills

### Agent 自动调用

当 Agent 判断某个 Skill 适用于当前任务时，会自动调用：

```
用户: 帮我写一个 Python API
Agent: [检测到相关 Skill "python-api-best-practices"]
Agent: 使用 Skill: python-api-best-practices
[Skill 被加载并应用]
```

### 手动调用

```bash
# 在对话中触发
/hermes: 使用 skill python-api-best-practices

# 强制使用特定 Skill
/hermes: apply-skill <skill-name>
```

---

## Skill 触发器（Triggers）

Triggers 定义何时自动使用该 Skill：

```yaml
---
name: docker-deploy
triggers:
  - "部署 docker"
  - "docker compose"
  - "containerize"
  - "k8s"
---
```

当用户消息匹配这些关键词时，Skill 会被自动加载。

---

## Skill 模板系统

### 常用模板

| 模板名 | 说明 |
|--------|------|
| `rest-api` | REST API 项目结构 |
| `cli-tool` | CLI 工具模板 |
| `web-scraper` | 网页爬虫模板 |
| `data-pipeline` | 数据处理流水线 |
| `bot` | 聊天机器人模板 |

### 创建自定义模板

```bash
# 将现有 Skill 转为基础模板
hermes skills templateize <skill-name>
```

---

## Skills 持久化

### 同步机制

```
新会话开始
    │
    ▼
加载所有持久化的 Skills
    │
    ▼
根据触发器/上下文自动挂载相关 Skills
    │
    ▼
对话中使用
    │
    ▼
如 Skill 被改进 → 更新磁盘副本
```

### 配置

```yaml
skills:
  # 自动创建 Skill
  auto_create: true

  # 自动持久化改进
  auto_persist: true

  # Skill 加载策略
  load_strategy: eager  # eager（预加载）或 lazy（按需）
```

---

## 最佳实践

### 1. 保持 Skill 专注

一个 Skill 做一件事：

```
✅ good-skill: "GitHub PR 审核"
❌ bad-skill: "做所有事情"
```

### 2. 清晰的 Trigger 关键词

```yaml
triggers:
  - "部署"
  - "deploy"
  - "docker"
  - "容器化"
```

### 3. 包含验证步骤

```markdown
## 验证

1. 运行测试：`pytest tests/`
2. 检查语法：`python -m py_compile`
3. 验证部署：`docker ps`
```

### 4. 记录适用场景和限制

```markdown
## 适用场景

✅ REST API
✅ 微服务
❌ 异步任务队列（用 celery-skill）
❌ GraphQL（用 graphql-skill）
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| Skill 不触发 | 检查 triggers 关键词 |
| Skill 内容过期 | 使用 `hermes skills refresh <skill-name>` |
| Skill 冲突 | 使用 `hermes skills list --verbose` 查看冲突 |

---

## 下一步

- [4.4 MCP 集成](./mcp.md) - 连接外部 MCP 服务器
- [4.5 语音模式](./voice-mode.md) - 配置语音交互