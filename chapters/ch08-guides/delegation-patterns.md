# Delegation & Parallel Work｜并行委托模式

> 原文：[Delegation & Parallel Work](https://hermes-agent.nousresearch.com/docs/guides/delegation-patterns)

---

## 目录

- [When to Delegate｜何时使用委托](#when-to-delegate何时使用委托)
- [Pattern: Parallel Research｜并行研究](#pattern-parallel-research并行研究)
- [Pattern: Code Review｜代码审查](#pattern-code-review代码审查)
- [The Context Problem｜上下文问题](#the-context-problem上下文问题)
- [Pattern: Compare Alternatives｜方案对比](#pattern-compare-alternatives方案对比)
- [Pattern: Multi-File Refactoring｜多文件重构](#pattern-multi-file-refactoring多文件重构)

---

## When to Delegate｜何时使用委托

### ✅ 适合委托的场景

| 场景 | 说明 |
|------|------|
| 推理密集型子任务 | 调试、代码审查、研究综合 |
| 会导致上下文泛滥的任务 | 大量中间数据 |
| 平行独立工作流 | 同时研究 A 和 B |
| 需要全新上下文的任务 | 希望 Agent 无偏见地处理 |

### ❌ 不适合委托的场景

| 场景 | 替代方案 |
|------|----------|
| 单个工具调用 | 直接使用工具 |
| 逻辑简单的机械性多步骤工作 | `execute_code` |
| 需要用户交互的任务 | Subagent 无法使用 `clarify` |
| 简单文件编辑 | 直接处理 |

---

## Pattern: Parallel Research｜并行研究

同时研究三个主题，获取结构化摘要：

```markdown
Research these three topics in parallel:
1. Current state of WebAssembly outside the browser
2. RISC-V server chip adoption in 2025
3. Practical quantum computing applications

Focus on recent developments and key players.
```

**幕后逻辑**：

```python
delegate_task(
    tasks=[
        {
            "goal": "Research WebAssembly outside the browser in 2025",
            "context": "Focus on: runtimes (Wasmtime, Wasmer), cloud/edge use cases, WASI progress",
            "toolsets": ["web"]
        },
        {
            "goal": "Research RISC-V server chip adoption",
            "context": "Focus on: server chips shipping, cloud providers adopting, software ecosystem",
            "toolsets": ["web"]
        },
        {
            "goal": "Research practical quantum computing applications",
            "context": "Focus on: error correction breakthroughs, real-world use cases, key companies",
            "toolsets": ["web"]
        }
    ]
)
```

所有三个任务**同时运行**。每个 subagent 独立搜索网络并返回摘要。父 Agent 然后将它们综合成一份连贯的简报。

---

## Pattern: Code Review｜代码审查

将安全审查委托给一个全新上下文的 subagent，避免先入为主：

```markdown
Review the authentication module at src/auth/ for security issues.
Check for SQL injection, JWT validation problems, password handling,
and session management. Fix anything you find and run the tests.
```

### Context 字段是关键

```python
delegate_task(
    goal="Review src/auth/ for security issues and fix any found",
    context="""Project at /home/user/webapp. Python 3.11, Flask, PyJWT, bcrypt.
    Auth files: src/auth/login.py, src/auth/jwt.py, src/auth/middleware.py
    Test command: pytest tests/auth/ -v
    Focus on: SQL injection, JWT validation, password hashing, session management.
    Fix issues found and verify tests pass.""",
    toolsets=["terminal", "file"]
)
```

---

## The Context Problem｜上下文问题

> ⚠️ **Subagent 对你的对话一无所知。** 它们完全全新开始。

如果委托"修复我们在讨论的 bug"，subagent 完全不知道你说的是哪个 bug。

**始终显式传递所有需要的信息**：
- 文件路径
- 错误信息
- 项目结构
- 约束条件

---

## Pattern: Compare Alternatives｜方案对比

并行评估同一问题的多种方案，然后选择最佳：

```markdown
I need to add full-text search to our Django app. Evaluate three approaches
in parallel:
1. PostgreSQL tsvector (built-in)
2. Elasticsearch via django-elasticsearch-dsl
3. Meilisearch via meilisearch-python

For each: setup complexity, query capabilities, resource requirements,
and maintenance overhead. Compare them and recommend one.
```

因为它们是隔离的，不会有交叉污染——每个评估都立足于自身优点。父 Agent 收到三个摘要后进行对比。

---

## Pattern: Multi-File Refactoring｜多文件重构

将大型重构任务分散到并行 subagent，每个处理代码库的不同部分：

```python
delegate_task(
    tasks=[
        {
            "goal": "Refactor all API endpoint handlers to use the new response format",
            "context": """Project at /home/user/api-server.
        Files: src/handlers/users.py, src/handlers/auth.py, src/handlers/billing.py
        Old format: return {"data": result, "status": "ok"}
        New format: return APIResponse(data=result, status=200).to_dict()
        Import: from src.responses import APIResponse
        Run tests after: pytest tests/handlers/ -v""",
            "toolsets": ["terminal", "file"]
        },
        {
            "goal": "Update all client SDK methods to handle the new response format",
            "context": """Project at /home/user/api-server.
        Files: sdk/python/client.py, sdk/python/models.py
        Old parsing: result = response.json()["data"]
        New parsing: result = response.json()["data"] (same key, but add status code checking)
        Also update sdk/python/tests/test_client.py""",
            "toolsets": ["terminal", "file"]
        },
        {
            "goal": "Update API documentation to reflect the new response format",
            "context": """Project at /home/user/api-server.
        Docs at: docs/api/. Format: Markdown with code examples.
        Update all response examples from old format to new format.
        Add a 'Response Format' section to docs/api/overview.md""",
            "toolsets": ["terminal", "file"]
        }
    ]
)
```

---

## 快速参考

| 模式 | 适用场景 | 关键 |
|------|----------|------|
| Parallel Research | 同时研究多个主题 | 每个 subagent 独立搜索 |
| Code Review | 安全/代码质量审查 | 传递所有上下文 |
| Compare Alternatives | 方案选型 | 隔离评估，无交叉污染 |
| Multi-File Refactoring | 大型重构 | 分散到不同文件/模块 |

**核心原则**：
- Subagent **完全无记忆**——传递一切
- 每个 subagent **独立运行**——互不干扰
- 只有**最终摘要返回**——减少 token 使用
