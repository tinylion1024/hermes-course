# Work with Skills｜Skills 深度使用指南

> 原文：[Working with Skills](https://hermes-agent.nousresearch.com/docs/guides/work-with-skills)

---

## 目录

- [Finding Skills｜查找技能](#finding-skills查找技能)
- [Searching for a Skill｜搜索技能](#searching-for-a-skill搜索技能)
- [The Skills Hub｜Skills Hub](#the-skills-hub-skills-hub)
- [Using a Skill｜使用技能](#using-a-skill使用技能)
- [Progressive Disclosure｜渐进式加载](#progressive-disclosure渐进式加载)
- [Installing from the Hub｜从 Hub 安装](#installing-from-the-hub从-hub-安装)
- [Verifying Installation｜验证安装](#verifying-installation验证安装)
- [Plugin-Provided Skills｜插件提供的技能](#plugin-provided-skills插件提供的技能)
- [Configuring Skill Settings｜配置技能设置](#configuring-skill-settings配置技能设置)
- [Creating Your Own Skill｜创建自己的技能](#creating-your-own-skill创建自己的技能)

---

## Finding Skills｜查找技能

每个 Hermes 安装都附带了捆绑的 Skills。查看可用内容：

```bash
# 在任何聊天会话中：
/skills

# 或从 CLI：
hermes skills list
```

输出示例：

```
ascii-art         Generate ASCII art using pyfiglet, cowsay, boxes...
arxiv             Search and retrieve academic papers from arXiv...
github-pr-workflow Full PR lifecycle — create branches, commit...
plan              Plan mode — inspect context, write a markdown...
excalidraw        Create hand-drawn style diagrams using Excalidraw...
```

---

## Searching for a Skill｜搜索技能

```bash
# 按关键词搜索
/skills search docker
/skills search music
```

---

## The Skills Hub｜Skills Hub {#the-skills-hub-skills-hub}

官方可选 Skills（较重或小众的 Skills，默认不激活）可通过 Hub 获取：

```bash
# 浏览官方可选 Skills
/skills browse

# 在 Hub 中搜索
/skills search blockchain
```

---

## Using a Skill｜使用技能

每个已安装的 Skill 自动成为一个斜杠命令。直接输入：

```bash
# 加载技能并给予任务
/ascii-art Make a banner that says "HELLO WORLD"
/plan Design a REST API for a todo app
/github-pr-workflow Create a PR for the auth refactor

# 只需技能名（无任务）会加载它，让你描述需求
/excalidraw
```

你也可以通过自然对话触发 Skills——让 Hermes 使用特定技能，它会通过 `skill_view` 工具加载。

---

## Progressive Disclosure｜渐进式加载 {#progressive-disclosure渐进式加载}

Skills 使用**token 高效的加载模式**。Agent 不会一次性加载所有内容：

| 层级 | 函数 | Token 成本 | 加载时机 |
|------|------|------------|----------|
| 列表 | `skills_list()` | ~3k tokens | 会话开始时 |
| 详情 | `skill_view(name)` | 视技能大小 | Agent 决定需要时 |
| 文件 | `skill_view(name, file_path)` | 更少 | 仅在需要时 |

这意味着 Skills **在真正使用前不消耗 token**。

---

## Installing from the Hub｜从 Hub 安装 {#installing-from-the-hub从-hub-安装}

官方可选 Skills 随 Hermes 附送，但默认不激活。显式安装：

```bash
# 安装官方可选技能
hermes skills install official/research/arxiv

# 在聊天会话中安装
/skills install official/creative/songwriting-and-ai-music
```

**安装后**：
- Skill 目录被复制到 `~/.hermes/skills/`
- 出现在 `skills_list()` 输出中
- 作为斜杠命令可用

> 💡 已安装的 Skills 在**新会话**生效。如果想在当前会话可用，用 `/reset` 开始新会话，或添加 `--now` 立即使 prompt 缓存失效（下次 turn 花费更多 tokens）。

---

## Verifying Installation｜验证安装

```bash
# 检查是否存在
hermes skills list | grep arxiv

# 或在聊天中
/skills search arxiv
```

---

## Plugin-Provided Skills｜插件提供的技能

插件可以使用命名空间名称（`plugin:skill`）捆绑自己的 Skills。这防止与内置 Skills 名称冲突。

```python
# 用限定名称加载插件技能
skill_view("superpowers:writing-plans")

# 不受影响的同名内置技能
skill_view("writing-plans")
```

插件 Skills **不会**列在系统提示中，也**不会**出现在 `skills_list()` 中。它们是可选的——在知道插件提供时显式加载。加载后，Agent 看到同一插件兄弟技能的横幅列表。

---

## Configuring Skill Settings｜配置技能设置

某些 Skills 在 frontmatter 中声明它们需要的配置：

```yaml
metadata:
  hermes:
    config:
      - key: tenor.api_key
        description: "Tenor API key for GIF search"
        prompt: "Enter your Tenor API key"
        url: "https://developers.google.com/tenor/guides/quickstart"
```

首次加载需要配置的 Skill 时，Hermes 会提示你输入。值存储在 `config.yaml` 的 `skills.config.*` 下。

**从 CLI 管理 Skill 配置**：

```bash
# 交互式配置特定技能
hermes skills config gif-search

# 查看所有技能配置
hermes config get skills.config
```

---

## Creating Your Own Skill｜创建自己的技能

Skills 就是带 YAML frontmatter 的 markdown 文件。创建一个不到五分钟。

### 1. 创建目录

```bash
mkdir -p ~/.hermes/skills/my-category/my-skill
```

### 2. 编写 SKILL.md

```markdown
---
name: my-skill
description: Brief description of what this skill does
version: 1.0.0
metadata:
  hermes:
    tags: ["my-tag", "automation"]
    category: my-category
---

# My Skill

## When to Use

Use this skill when the user asks about [specific topic] or needs to [specific task].

## Procedure

1. First, check if [prerequisite] is available
2. Run `command --with-flags`
3. Parse the output and present results

## Pitfalls

- Common failure: [description]. Fix: [solution]
- Watch out for [edge case]

## Verification

Run `check-command` to confirm the result is correct.
```

### 3. 添加参考文件（可选）

```
my-skill/
├── SKILL.md                    # 主技能文档
├── references/
│   ├── api-docs.md             # Agent 可咨询的 API 参考
│   └── examples.md             # 示例输入/输出
├── templates/
│   └── config.yaml             # Agent 可使用的模板文件
└── scripts/
    └── setup.sh                # Agent 可执行的脚本
```

在 SKILL.md 中引用这些：

```markdown
For API details, load the reference: `skill_view("my-skill", "references/api-docs.md")`
```

### 4. 测试它

开始新会话并尝试你的技能：

```
/my-skill
```

---

## 快速参考

| 命令 | 说明 |
|------|------|
| `/skills` | 列出所有技能 |
| `/skills search <keyword>` | 搜索技能 |
| `/skills browse` | 浏览 Hub |
| `/skills install <name>` | 从 Hub 安装 |
| `/my-skill` | 使用技能 |

**创建流程**：
1. `mkdir -p ~/.hermes/skills/<category>/<skill>`
2. 编写 `SKILL.md`（frontmatter + markdown）
3. （可选）添加 `references/`、`templates/`、`scripts/`
4. 新会话中测试
