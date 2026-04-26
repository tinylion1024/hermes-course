# 4.2 记忆系统

## 什么是记忆系统？

Hermes Agent 的记忆系统是其最强大的特性之一。它允许 Agent 在**跨会话**中保持状态、学习和积累知识，与传统的每次对话都从零开始的 AI 助手有本质区别。

---

## 记忆类型

### 1. 工作记忆（Working Memory）

当前会话的上下文，随会话结束而清除。

```
用户: 帮我写一个函数
Agent: [工作记忆中包含之前的对话]
用户: 改成 async 版本
Agent: [工作记忆中记得之前写的函数]
```

### 2. 持久记忆（Persistent Memory）

跨会话保存的重要信息。

```yaml
# ~/.hermes/memory/
├── user_preferences.yaml   # 用户偏好
├── project_context/         # 项目上下文
│   ├── repo_1/
│   └── repo_2/
└── knowledge/              # 知识库
```

### 3. 程序记忆（Skills）

见 [4.3 Skills 系统](./skills.md)。

---

## 记忆的工作原理

```
┌─────────────────────────────────────────────────────────────┐
│                    用户对话流程                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. 用户消息 → 检索相关记忆                                  │
│    └── 记忆系统搜索相关历史信息                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Agent 处理 → 更新记忆（如有必要）                         │
│    └── 如果当前对话包含新知识，存入记忆                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. 会话结束 → 持久化记忆                                     │
│    └── 重要信息写入磁盘                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 配置记忆系统

### config.yaml 配置

```yaml
memory:
  enabled: true
  persist: true

  # 记忆保留策略
  retention:
    # 短期记忆：7 天后降级
    short_term: 7d
    # 长期记忆：保留 90 天
    long_term: 90d

  # 记忆自动摘要（减少存储）
  auto_summarize:
    enabled: true
    threshold: 100  # 超过 100 条后自动摘要
```

### 环境变量

```bash
# 禁用记忆
HERMES_MEMORY_ENABLED=false

# 记忆存储路径
HERMES_MEMORY_PATH=~/.hermes/memory
```

---

## CLI 命令

### 查看记忆

```bash
# 列出所有记忆
hermes memory list

# 查看特定类型的记忆
hermes memory list --type user_preference
hermes memory list --type project_context

# 搜索记忆
hermes memory search "项目 X 的配置"
```

### 添加记忆

```bash
# 手动添加记忆
hermes memory add "用户喜欢在早上处理复杂的技术任务"

# 添加带标签的记忆
hermes memory add "用户使用 Python 3.11" --tag language --tag python
```

### 删除记忆

```bash
# 删除特定记忆
hermes memory delete <memory-id>

# 清除所有记忆（慎用！）
hermes memory clear --all
```

### 导入/导出

```bash
# 导出记忆
hermes memory export --output /path/to/backup.json

# 导入记忆
hermes memory import /path/to/backup.json
```

---

## 记忆类型详解

### 用户偏好（User Preferences）

存储用户的工作习惯和偏好：

```yaml
# ~/.hermes/memory/user_preferences.yaml
preferred_language: zh-CN
work_hours:
  start: 9:00
  end: 18:00
timezone: Asia/Shanghai
preferred_shell: zsh
editor: vscode
```

### 项目上下文（Project Context）

为每个项目维护独立的上下文：

```
~/.hermes/memory/project_context/
├── my-project/
│   ├── description: "电商后端服务"
│   ├── tech_stack: ["Python", "FastAPI", "PostgreSQL"]
│   ├── main_branch: main
│   └── key_files: ["src/main.py", "requirements.txt"]
└── another-project/
```

### 知识库（Knowledge）

存储学到的技术和概念：

```yaml
# ~/.hermes/memory/knowledge/
- concept: "CQRS 模式"
  summary: "命令查询职责分离，用于分离读写操作"
  source: "用户讨论架构设计时学习"
  date_added: "2024-01-15"
```

---

## 记忆的访问控制

### 隐私设置

```yaml
memory:
  # 哪些对话自动存入记忆
  auto_remember:
    explicit_requests: true  # 用户明确要求记住的
    inferred_preferences: true  # 推断的用户偏好
    project_context: true  # 项目上下文

  # 排除敏感内容
  exclude_patterns:
    - "password*"
    - "*secret*"
    - "*token*"
```

---

## 最佳实践

### 1. 敏感信息处理

> ⚠️ 不要让 Agent 记住敏感信息（密码、API Key、个人身份信息）。在配置中使用 `exclude_patterns`。

### 2. 定期清理

```bash
# 清理 90 天前的旧记忆
hermes memory clean --older-than 90d
```

### 3. 验证记忆准确性

定期检查记忆内容，确保信息准确：

```bash
hermes memory list --type all | less
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 记忆不保存 | 检查 `~/.hermes/memory` 权限 |
| 记忆不检索 | 使用 `hermes memory rebuild` 重建索引 |
| 记忆混乱 | 使用 `hermes memory clear --type <type>` 清除特定类型 |

---

## 下一步

- [4.3 Skills 系统](./skills.md) - 学习程序化记忆
- [4.4 MCP 集成](./mcp.md) - 扩展更多功能