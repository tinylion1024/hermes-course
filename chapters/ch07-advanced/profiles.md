# 7.1 多 Agent 配置（Profiles）

## 什么是 Profiles？

Profiles 允许你在同一台机器上运行**多个独立的 Hermes Agent 实例**，每个实例有独立的：
- 配置
- 会话历史
- 记忆
- Skills

---

## 使用场景

| 场景 | 说明 |
|------|------|
| 工作/个人分离 | 工作用 work profile，个人用 personal profile |
| 多项目并行 | 每个项目独立的 Agent |
| 多团队协作 | 不同团队用不同的 Agent 配置 |
| 开发/生产分离 | dev profile 和 prod profile |

---

## 基本操作

### 创建 Profile

```bash
# 创建新 Profile
hermes profile create work

# 创建带配置的 Profile
hermes profile create project-x --model anthropic/claude-opus-4.6
```

### 切换 Profile

```bash
# 使用指定 Profile 运行
hermes --profile work

# 切换到另一个 Profile
hermes profile switch personal

# 设置默认 Profile
hermes profile set-default work
```

### 列出 Profiles

```bash
# 列出所有 Profile
hermes profile list
```

### 删除 Profile

```bash
hermes profile delete <profile-name>
```

---

## Profile 结构

每个 Profile 有独立的目录结构：

```
~/.hermes/
├── default/              # 默认 Profile
│   ├── config.yaml
│   ├── .env
│   ├── sessions/
│   ├── memory/
│   └── skills/
├── work/                 # 工作 Profile
│   ├── config.yaml
│   ├── .env
│   ├── sessions/
│   ├── memory/
│   └── skills/
└── personal/             # 个人 Profile
    ├── config.yaml
    ├── .env
    ├── sessions/
    ├── memory/
    └── skills/
```

---

## Profile 配置

### 独立配置示例

`~/.hermes/work/config.yaml`：

```yaml
model: anthropic/claude-opus-4.6

memory:
  enabled: true
  # 工作相关记忆优先
  priority: work

skills:
  auto_create: true
  # 只加载工作相关的 Skills
  include_tags:
    - work
    - code
    - github

telegram:
  enabled: true
  bot_token: ${WORK_BOT_TOKEN}
  # 工作 Bot
```

### Profile 环境变量

`~/.hermes/work/.env`：

```bash
ANTHROPIC_API_KEY=sk-ant-work-xxxxx
TELEGRAM_BOT_TOKEN=123456:ABC-work
WORK_BOT_TOKEN=123456:ABC-work
```

---

## 高级用法

### Profile 模板

```bash
# 从模板创建 Profile
hermes profile create team-sre --template sre

# 查看可用模板
hermes profile templates
```

### Profile 同步

```bash
# 同步 Skills 到其他 Profile
hermes profile sync-skills work → personal

# 同步记忆（选择性）
hermes profile sync-memory work → personal --include work
```

### Profile 快捷方式

在 `~/.bashrc` 或 `~/.zshrc` 中添加别名：

```bash
alias hermes-work='hermes --profile work'
alias hermes-personal='hermes --profile personal'
alias hermes-projectx='hermes --profile project-x'
```

---

## 多 Agent 协作

### Agent 间通信

```bash
# 从 work Profile 发送消息到 personal Profile
hermes --profile work send --to personal "完成任务了！"
```

### 共享 Skills

创建共享 Skills 目录：

```yaml
# ~/.hermes/shared/skills/
skills:
  shared_skills_path: ~/.hermes/shared/skills
```

---

## 最佳实践

### 1. Profile 命名

使用有意义的命名：
```
✅ good: work, personal, project-alpha, sre-team
❌ bad: p1, test, abc, profile1
```

### 2. 隔离敏感信息

每个 Profile 的 `.env` 文件独立保管敏感信息。

### 3. 定期清理

```bash
# 清理旧会话
hermes --profile <name> sessions clean --older-than 30d

# 清理记忆
hermes --profile <name> memory clean --older-than 90d
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| Profile 切换无效 | 检查 profile 名称 |
| 找不到配置 | 确认配置文件在正确目录 |
| 权限错误 | 检查目录权限 |

---

## 下一步

- [7.2 API Server](./api-server.md) - 部署 HTTP API