# Cloud Deployments｜云端部署

> 原文：[AWS Bedrock](https://hermes-agent.nousresearch.com/docs/guides/aws-bedrock) + [Azure AI Foundry](https://hermes-agent.nousresearch.com/docs/guides/azure-foundry) + [Run Local LLMs on Mac](https://hermes-agent.nousresearch.com/docs/guides/local-llm-on-mac)

---

## 目录

- [AWS Bedrock｜Amazon AI 云](#aws-bedrockamazon-ai-云)
  - [配置 AWS Bedrock](#配置-aws-bedrock)
  - [可用模型](#可用模型)
  - [认证方式](#认证方式)
- [Azure AI Foundry｜Microsoft AI 云](#azure-ai-foundrymicrosoft-ai-云)
  - [配置 Azure AI Foundry](#配置-azure-ai-foundry)
  - [可用模型](#可用模型-1)
- [Run Local LLMs on Mac｜Mac 本地模型](#run-local-llms-on-macmac-本地模型)
  - [为什么要本地运行？](#为什么要本地运行)
  - [安装 Ollama](#安装-ollama)
  - [配置 Hermes 使用 Ollama](#配置-hermes-使用-ollama)
  - [推荐模型](#推荐模型)
  - [性能提示](#性能提示)

---

## AWS Bedrock｜Amazon AI 云

AWS Bedrock 提供多种 AI 模型，通过 AWS IAM 认证访问。

### 配置 AWS Bedrock

```bash
# 在 ~/.hermes/.env 中设置
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1
```

或在 `~/.hermes/config.yaml` 中：

```yaml
model:
  provider: aws-bedrock
  name: anthropic.claude-3-5-sonnet-20241022-v2-0

providers:
  aws-bedrock:
    region: us-east-1
    # 或使用 AWS 默认凭证链（环境变量、~/.aws/credentials 等）
```

### 可用模型

| 模型 | 说明 |
|------|------|
| `anthropic.claude-3-5-sonnet-20241022-v2-0` | Claude 3.5 Sonnet |
| `anthropic.claude-3-opus-20240229-v1-0` | Claude 3 Opus |
| `anthropic.claude-3-haiku-20240307-v1-0` | Claude 3 Haiku |
| `amazon.titan-text-express-v1` | Amazon Titan |
| `meta.llama3-2-90b-instruct-v1:0` | Llama 3.2 90B |

### 认证方式

```bash
# 方式 1：环境变量
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...

# 方式 2：AWS 配置文件
# ~/.aws/credentials
# ~/.aws/config

# 方式 3：IAM Role（EC2/ECS/Lambda）
# 自动从元数据服务获取临时凭证
```

---

## Azure AI Foundry｜Microsoft AI 云

Azure AI Foundry（前称 Azure AI Studio）提供 OpenAI 兼容 API 及 Azure 自有模型。

### 配置 Azure AI Foundry

```yaml
# ~/.hermes/config.yaml
model:
  provider: azure
  name: gpt-4o

providers:
  azure:
    endpoint: https://YOUR-RESOURCE.openai.azure.com
    api_version: "2024-02-01"
    # 或使用环境变量
    # AZURE_OPENAI_ENDPOINT
    # AZURE_OPENAI_API_KEY
```

在 `~/.hermes/.env` 中：

```bash
AZURE_OPENAI_ENDPOINT=https://YOUR-RESOURCE.openai.azure.com
AZURE_OPENAI_API_KEY=your-api-key
AZURE_OPENAI_API_VERSION=2024-02-01
```

### 可用模型

| 模型 | 说明 |
|------|------|
| `gpt-4o` | OpenAI GPT-4o |
| `gpt-4-turbo` | GPT-4 Turbo |
| `gpt-35-turbo` | GPT-3.5 Turbo |
| `o1-preview` | OpenAI o1 Preview |
| `o1-mini` | OpenAI o1 Mini |

---

## Run Local LLMs on Mac｜Mac 本地模型

在 Mac 上本地运行 LLM，保护隐私、零 API 成本、无网络依赖。

### 为什么要本地运行？

| 优势 | 说明 |
|------|------|
| 隐私 | 数据不离开本地 |
| 成本 | 无 API 调用费用 |
| 离线 | 无网络依赖 |
| 定制 | 可运行微调模型 |

### 安装 Ollama

```bash
# macOS
brew install ollama

# 或下载安装包
# https://ollama.ai/download
```

启动 Ollama：

```bash
ollama serve
```

### 配置 Hermes 使用 Ollama

```yaml
# ~/.hermes/config.yaml
model:
  provider: ollama
  name: llama3.2

providers:
  ollama:
    base_url: http://localhost:11434
```

或环境变量：

```bash
OLLAMA_BASE_URL=http://localhost:11434
```

### 推荐模型

| 模型 | 参数 | 适用场景 | Mac 建议 |
|------|------|----------|----------|
| `llama3.2` | 3B | 日常任务、代码补全 | ✅ 轻松运行 |
| `codellama` | 7B | 代码生成、调试 | ✅ 推荐 |
| `mistral` | 7B | 通用对话 | ✅ 推荐 |
| `llama3.2` | 70B | 复杂推理 | ⚠️ 仅限高配 Mac |
| `phi3` | 3.8B | 轻量任务 | ✅ 最佳性价比 |

```bash
# 拉取模型
ollama pull llama3.2
ollama pull codellama

# 查看已安装模型
ollama list
```

### 性能提示

| 建议 | 说明 |
|------|------|
| 使用 Apple Silicon | Ollama 自动利用 GPU |
| 量化模型 | 使用 Q4_K_M 量化平衡质量和速度 |
| 调整上下文窗口 | `OLLAMA_NUM_PARALLEL=4` 限制并发 |
| 关闭不必要的应用 | 释放内存给 Ollama |

---

## 快速参考

| 云平台 | 配置项 | 认证 |
|--------|--------|------|
| AWS Bedrock | `providers.aws-bedrock.region` | AWS 凭证/IAM Role |
| Azure AI Foundry | `providers.azure.endpoint` | API Key |
| 本地 Ollama | `providers.ollama.base_url: http://localhost:11434` | 无需认证 |

**选择建议**：
- **快速原型 / 隐私敏感** → 本地 Ollama
- **生产环境 / 多模型** → AWS Bedrock
- **已有 Azure 投资** → Azure AI Foundry
- **预算有限** → 本地或 Bedrock（按 token 计费）
