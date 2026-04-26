# 第二章：安装与环境配置

## 2.1 快速安装（60 秒）

### 支持的系统

- ✅ Linux（Ubuntu、Debian、CentOS、Fedora 等）
- ✅ macOS
- ✅ WSL2（Windows Subsystem for Linux）
- ✅ Android / Termux

### 一键安装命令

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### 安装后配置 Shell

```bash
# 重新加载 shell 配置
source ~/.bashrc  # Bash 用户
source ~/.zshrc   # Zsh 用户
```

### Windows 用户注意事项

> ⚠️ Windows 原生不支持，请先安装 WSL2

1. 安装 WSL2：
   ```powershell
   wsl --install
   ```

2. 在 WSL2 终端中运行安装命令

3. 重启后运行：
   ```bash
   source ~/.bashrc
   ```

---

## 2.2 Android / Termux 安装

如果在手机上使用，请参阅专门的 [Termux 安装指南](../ch02-install/termux.md)。

### 特殊要求

- Termux 环境
- 可能需要手动安装部分依赖
- Android 特定功能限制请参考官方文档

---

## 2.3 Nix / NixOS 安装

对于 NixOS 用户，提供专门的 [Nix 安装指南](../ch02-install/nix-setup.md)。

---

## 2.4 Docker 安装

详细 Docker 配置请参阅 [Docker 安装指南](../ch02-install/docker.md)。

### 基本用法

```bash
# 拉取镜像
docker pull nousresearch/hermes-agent

# 运行容器
docker run -it nousresearch/hermes-agent
```

---

## 2.5 配置 AI Provider

### 选择 Provider

运行交互式配置：

```bash
hermes model
```

### Provider 推荐

| 场景 | 推荐方案 | 说明 |
|------|----------|------|
| 最省力 | Nous Portal 或 OpenRouter | 开箱即用 |
| 已有 Claude/Copycode 账号 | Anthropic 或 OpenAI | 直接使用 |
| 本地/私有推理 | Ollama 或自定义端点 | 隐私优先 |
| 多 Provider 路由 | OpenRouter | 灵活切换 |
| 自定义 GPU 服务器 | vLLM、SGLang、LiteLLM | 高性能 |

### 环境变量配置

```bash
# 设置 API Key
hermes config set OPENROUTER_API_KEY sk-or-v1-xxxxx

# 或直接编辑配置文件
nano ~/.hermes/.env
```

### 上下文窗口要求

> ⚠️ **最低要求：64K tokens**
>
> Hermes Agent 要求模型至少具有 **64,000 tokens** 的上下文窗口。较小的模型无法维持多步骤工具调用工作流，会在启动时被拒绝。
>
> 大多数托管模型（Claude、GPT、Gemini、Qwen、DeepSeek）都满足此要求。
>
> 本地模型示例：
> - llama.cpp: `--ctx-size 65536`
> - Ollama: `-c 65536`

---

## 2.6 配置文件结构

### 文件位置

```
~/.hermes/
├── .env              # 密钥和 Token
├── config.yaml       # 非敏感配置
├── sessions/         # 会话历史
├── memory/           # 记忆数据
├── skills/          # 技能模块
└── logs/             # 日志文件
```

### .env 文件示例

```bash
# API Keys
ANTHROPIC_API_KEY=sk-ant-xxxxx
OPENAI_API_KEY=sk-xxxxx
OPENROUTER_API_KEY=sk-or-v1-xxxxx

# Provider 配置
ANTHROPIC_BASE_URL=https://api.anthropic.com

# 自定义模型
OLLAMA_BASE_URL=http://localhost:11434
```

### config.yaml 示例

```yaml
model: anthropic/claude-opus-4.6
terminal.backend: docker

# 记忆配置
memory:
  enabled: true
  persist: true

# 技能配置
skills:
  auto_create: true
  persist: true
```

### 使用 CLI 配置

```bash
# 设置模型
hermes config set model anthropic/claude-opus-4.6

# 设置终端后端
hermes config set terminal.backend docker

# 查看当前配置
hermes config list
```

---

## 2.7 验证安装

### 检查版本

```bash
hermes --version
```

### 运行测试对话

```bash
hermes
```

输入测试消息：

```
总结这个代码库，用 5 个要点，并告诉我主入口文件是什么。
```

### 验证成功标准

✅ 显示欢迎横幅，包含选定的模型/Provider  
✅ Hermes 无错误回复  
✅ 可以使用工具（如需要）  
✅ 多轮对话正常  

---

## 2.8 故障排除

| 问题 | 解决方案 |
|------|----------|
| 安装失败 | 检查 curl 和 bash 是否可用 |
| 模型无法连接 | 检查 API Key 和网络 |
| 上下文窗口不足 | 更换支持 64K+ 的模型 |
| 权限错误 | 检查 ~/.hermes 目录权限 |

---

## 2.9 更新与卸载

详细说明请参阅 [更新与卸载指南](../ch02-install/updating.md)。

---

## 2.10 下一步

- [第三章：快速上手](../ch03-quickstart/README.md) - 开始你的第一次对话
- [第四章：核心功能](../ch04-features/README.md) - 探索 47 个内置工具