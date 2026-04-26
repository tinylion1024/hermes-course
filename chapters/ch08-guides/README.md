# 第八章：实战指南与最佳实践

> 本章精炼自 [Hermes Agent 官方 Guides & Tutorials](https://hermes-agent.nousresearch.com/docs/guides/tips)，将 20+ 实战指南炼化为可直接落地的精华。

---

## 章节结构

```
ch08-guides/
├── README.md                        # 本章导航
├── tips-best-practices.md           # Tips & Best Practices（官方原文精华）
├── work-with-skills.md              # Skills 深度使用指南
├── delegation-patterns.md           # 并行委托与多任务模式
├── automation-patterns.md            # 自动化模式（CRON + Templates）
├── integrations.md                  # 集成大全（MCP / SOUL.md / Voice / Plugin / Python）
├── github-integrations.md           # GitHub 集成（PR Review / Webhook）
├── cloud-deployments.md              # 云端部署（AWS Bedrock / Azure / 本地模型）
└── migration-guide.md                # 迁移指南（OpenClaw → Hermes）
```

---

## 快速导航

### 新手必读
| 指南 | 内容 | 难度 |
|------|------|------|
| [Tips & Best Practices](./tips-best-practices.md) | 提升效率的 50+ 实战技巧 | ⭐ |
| [Work with Skills](./work-with-skills.md) | 善用 Skills 系统 | ⭐ |

### 进阶实战
| 指南 | 内容 | 难度 |
|------|------|------|
| [Delegation Patterns](./delegation-patterns.md) | 并行委托与多 Agent 协作 | ⭐⭐ |
| [Automation Patterns](./automation-patterns.md) | Cron 定时任务与自动化模板 | ⭐⭐ |
| [Integrations](./integrations.md) | MCP / SOUL / Voice / Plugin / Python | ⭐⭐ |

### 专项集成
| 指南 | 内容 | 难度 |
|------|------|------|
| [GitHub Integrations](./github-integrations.md) | PR Review + Webhook 自动审查 | ⭐⭐ |
| [Cloud Deployments](./cloud-deployments.md) | AWS Bedrock / Azure / 本地模型 | ⭐⭐ |
| [Migration Guide](./migration-guide.md) | 从 OpenClaw 迁移 | ⭐ |

---

## 核心学习路径

```
入门路径：
  1. tips-best-practices.md        → 掌握日常使用技巧
  2. work-with-skills.md           → 学会使用和创建 Skills

实战路径：
  3. automation-patterns.md        → 定时自动化
  4. delegation-patterns.md         → 并行任务处理
  5. github-integrations.md         → GitHub 自动化

深度路径：
  6. integrations.md                → MCP / SOUL / Voice / Plugin
  7. cloud-deployments.md           → 各类 AI Provider
  8. migration-guide.md             → 从其他工具迁移
```

---

## 本章特色

- **官方原文精华**：所有内容提炼自 Hermes Agent 官方文档
- **可直接落地**：每个指南都有具体命令和示例
- **场景驱动**：按实际使用场景组织，而非简单翻译
- **中文本地化**：保留关键命令的英文原文，便于实操

---

> 💡 **提示**：本章内容较丰富，建议按上述路径循序渐进学习。每篇文档都可独立阅读。
