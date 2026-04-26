# 4.4 MCP 集成

## 什么是 MCP？

MCP（Model Context Protocol）是一种开放协议，允许 AI 应用连接到外部数据源和工具。Hermes Agent 支持 MCP 客户端模式，可以连接任何兼容的 MCP 服务器。

---

## MCP 工作原理

```
┌─────────────────────────────────────────────────────────────┐
│                    Hermes Agent                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    MCP Client                          │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐               │  │
│  │  │ 工具过滤 │  │ 请求路由 │  │ 响应处理 │               │  │
│  │  └─────────┘  └─────────┘  └─────────┘               │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ MCP 协议
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   MCP 服务器                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                    │
│  │Server A │  │Server B │  │Server C │                    │
│  │(文件系统)│  │(GitHub) │  │(数据库) │                    │
│  └─────────┘  └─────────┘  └─────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

---

## MCP 服务器

### 官方 MCP 服务器

| 服务器 | 功能 | 安装 |
|--------|------|------|
| `mcp-server-filesystem` | 文件系统访问 | `npx mcp-server-filesystem` |
| `mcp-server-github` | GitHub API | `npx mcp-server-github` |
| `mcp-server-brave` | 网页搜索 | `npx mcp-server-brave` |
| `mcp-server-slack` | Slack 集成 | `npx mcp-server-slack` |
| `mcp-server-sqlite` | SQLite 数据库 | `npx mcp-server-sqlite` |

---

## 配置 MCP

### 基础配置

编辑 `~/.hermes/config.yaml`：

```yaml
mcp:
  # 启用 MCP
  enabled: true

  # MCP 服务器列表
  servers:
    # 文件系统服务器
    - name: filesystem
      command: npx
      args:
        - mcp-server-filesystem
        - /home/user/allowed-projects

    # GitHub 服务器
    - name: github
      command: npx
      args:
        - mcp-server-github
      env:
        GITHUB_TOKEN: ${GITHUB_TOKEN}  # 从环境变量引用
```

### 工具过滤

```yaml
mcp:
  servers:
    - name: github
      command: npx
      args:
        - mcp-server-github
      # 只允许特定工具
      allowed_tools:
        - github_repos_list
        - github_issues_list
        - github_pulls_list
      # 禁用某些工具
      denied_tools:
        - github_repo_delete
```

### 传输方式

```yaml
mcp:
  servers:
    # STDIO 传输（默认）
    - name: local-server
      command: node
      args:
        - /path/to/server.js
      transport: stdio

    # HTTP/SSE 传输
    - name: remote-server
      command: python
      args:
        - mcp_server.py
      transport: http
      url: http://localhost:8080/mcp
```

---

## 常用 MCP 服务器配置示例

### 1. 文件系统

```yaml
mcp:
  servers:
    - name: projects
      command: npx
      args:
        - mcp-server-filesystem
        - /home/user/projects
        - /home/user/documents
```

### 2. GitHub

```yaml
mcp:
  servers:
    - name: github
      command: npx
      args:
        - mcp-server-github
      env:
        GITHUB_TOKEN: ${GITHUB_TOKEN}
      allowed_tools:
        - github_repos_list
        - github_issues_list
        - github_pulls_create
```

### 3. Brave 搜索

```yaml
mcp:
  servers:
    - name: brave-search
      command: npx
      args:
        - mcp-server-brave
      env:
        BRAVE_API_KEY: ${BRAVE_API_KEY}
```

### 4. Slack

```yaml
mcp:
  servers:
    - name: slack
      command: npx
      args:
        - mcp-server-slack
      env:
        SLACK_BOT_TOKEN: ${SLACK_BOT_TOKEN}
        SLACK_TEAM_ID: ${SLACK_TEAM_ID}
```

---

## MCP 安全最佳实践

> ⚠️ MCP 服务器可以执行本地命令和访问数据，请务必：

### 1. 最小权限原则

```yaml
# 只允许必要的工具
mcp:
  servers:
    - name: github
      allowed_tools:
        - github_repos_list  # 只允许列表操作
        - github_issues_list
        - github_pulls_list
        # 不允许删除、推送等危险操作
```

### 2. 限制访问路径

```yaml
mcp:
  servers:
    - name: filesystem
      args:
        # 只允许访问特定目录
        - /home/user/projects
        # 不要允许整个 home 目录！
        # - /home/user  ❌
```

### 3. 定期审计

```bash
# 查看已配置的 MCP 服务器
hermes mcp list

# 查看每个服务器被允许的工具
hermes mcp tools <server-name>

# 审计日志
hermes mcp logs --recent 100
```

---

## CLI 命令

```bash
# 列出已配置的 MCP 服务器
hermes mcp list

# 测试 MCP 服务器连接
hermes mcp test <server-name>

# 查看可用工具
hermes mcp tools <server-name>

# 添加 MCP 服务器
hermes mcp add <server-config>

# 移除 MCP 服务器
hermes mcp remove <server-name>
```

---

## MCP 服务器开发

### 创建自定义 MCP 服务器

```javascript
// server.js
const { Server } = require('@modelcontextprotocol/sdk');
const { StdioServerTransport } = require('@modelcontextprotocol/sdk/server/stdio');

const server = new Server(
  {
    name: 'my-mcp-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

server.setRequestHandler('tools/list', async () => {
  return {
    tools: [
      {
        name: 'my_tool',
        description: '我的自定义工具',
        inputSchema: {
          type: 'object',
          properties: {
            param: { type: 'string' },
          },
        },
      },
    ],
  };
});

server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  if (name === 'my_tool') {
    return {
      content: [
        {
          type: 'text',
          text: `执行了 my_tool，参数: ${args.param}`,
        },
      ],
    };
  }
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main();
```

### 运行自定义服务器

```yaml
mcp:
  servers:
    - name: custom
      command: node
      args:
        - /path/to/server.js
      transport: stdio
```

---

## 故障排除

| 问题 | 解决方案 |
|------|----------|
| 连接失败 | 检查服务器命令和路径 |
| 工具不工作 | 查看 `hermes mcp logs` 排查 |
| 权限不足 | 检查 `allowed_tools` 配置 |
| 超时 | 增加 timeout 配置 |

---

## 更多资源

- [MCP 官方文档](https://modelcontextprotocol.io)
- [MCP 服务器列表](https://github.com/modelcontextprotocol/servers)

---

## 下一步

- [4.5 语音模式](./voice-mode.md) - 配置实时语音交互
- [第五章：消息平台](../ch05-messaging/README.md) - 连接 Telegram、Discord