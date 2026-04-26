# 4.1 内置工具详解

## 概述

Hermes Agent 内置 **47 个工具**，涵盖终端操作、文件管理、Web 浏览、媒体处理、消息发送等核心功能。

---

## 工具分类总览

| 类别 | 数量 | 主要工具 |
|------|------|----------|
| 核心工具 | 12 | terminal, read_file, write_file, search_files, patch |
| 自动化 | 8 | cronjob, process, webhook_subscriptions |
| 媒体/Web | 10 | browser_*, vision_analyze, text_to_speech |
| 消息 | 8 | send_message, telegram_send, discord_send, email_send |
| 数据处理 | 6 | json_parse, csv_read, api_request |
| 其他 | 3 | delegate_task, skill_manage, memory |

---

## 核心工具（Core Tools）

### terminal

在本地或远程执行 shell 命令。

```bash
# 基本用法
hermes$ terminal ls -la

# 带超时
hermes$ terminal "curl -s https://api.example.com" --timeout 30

# 后台运行
hermes$ terminal "npm run dev" --background
```

**参数：**
- `command`：要执行的命令
- `timeout`：超时时间（秒）
- `workdir`：工作目录
- `background`：是否后台运行

---

### read_file

读取文件内容。

```bash
# 基本读取
hermes$ read_file ~/.bashrc

# 指定行范围
hermes$ read_file ~/.bashrc --offset 1 --limit 100
```

**参数：**
- `path`：文件路径
- `offset`：起始行（默认 1）
- `limit`：最大行数（默认 500）

**注意：** 大文件会被截断，使用 offset/limit 读取特定部分。

---

### write_file

写入或创建文件。

```bash
# 创建/覆盖文件
hermes$ write_file --path /tmp/test.txt --content "Hello World"
```

**参数：**
- `path`：文件路径（会自动创建父目录）
- `content`：文件内容（完全覆盖）

**注意：** 会完全覆盖现有文件，使用前请确认。

---

### patch

智能查找替换，比 sed/awk 更安全。

```bash
# 基本替换
hermes$ patch --path main.py --old_string "def old_func():" --new_string "def new_func():"
```

**参数：**
- `path`：文件路径
- `old_string`：要替换的文本
- `new_string`：替换后的文本
- `replace_all`：是否全部替换（默认 false）

---

### search_files

搜索文件内容或查找文件名。

```bash
# 搜索文件内容
hermes$ search_files --pattern "TODO" --path .

# 查找文件名
hermes$ search_files --pattern "*.py" --target files

# 带上下文
hermes$ search_files --pattern "class User" --context 3
```

**参数：**
- `pattern`：正则表达式或 glob 模式
- `target`：content（内容）或 files（文件名）
- `path`：搜索路径
- `context`：上下文行数

---

### execute_code

执行 Python 代码，可在代码中调用 Hermes 工具。

```python
from hermes_tools import terminal, read_file

# 在代码中使用工具
result = terminal("ls -la")
print(result)

# 文件操作
content = read_file("/tmp/example.txt")
```

**注意：** 需要导入 hermes_tools 模块。

---

## 自动化工具（Automation）

### cronjob

创建和管理定时任务。

```bash
# 创建定时任务
hermes$ cronjob create --name "daily-report" --schedule "0 9 * * *" --prompt "生成每日报告"

# 列出所有任务
hermes$ cronjob list

# 删除任务
hermes$ cronjob delete <job-id>
```

**调度格式：**
- `30m`：30 分钟后
- `2h`：2 小时后
- `every 2h`：每 2 小时
- `0 9 * * *`：每天 9 点（cron 格式）

---

### process

管理后台进程。

```bash
# 列出运行中的进程
hermes$ process list

# 查看进程日志
hermes$ process log <session-id>

# 终止进程
hermes$ process kill <session-id>

# 等待进程完成
hermes$ process wait <session-id> --timeout 60
```

---

## 媒体工具（Media）

### browser_navigate

打开网页。

```bash
hermes$ browser_navigate "https://github.com"
```

---

### browser_snapshot

获取页面快照。

```bash
# 紧凑模式（默认）
hermes$ browser_snapshot

# 完整页面内容
hermes$ browser_snapshot --full true
```

---

### browser_click

点击页面元素。

```bash
# 使用 ref ID 点击
hermes$ browser_click --ref e5
```

**提示：** 先运行 `browser_snapshot` 获取元素 ref ID。

---

### browser_type

在输入框中输入文本。

```bash
hermes$ browser_type --ref e3 --text "Hello World"
```

---

### vision_analyze

分析图片内容。

```bash
hermes$ vision_analyze --image_url "/tmp/screenshot.png" --question "这张图里有什么？"
```

---

### text_to_speech

文字转语音。

```bash
hermes$ text_to_speech "你好，这是语音测试"
```

---

## 消息工具（Messaging）

### send_message

发送消息到已连接的平台。

```bash
# 发送到默认频道
hermes$ send_message "Hello!"

# 发送到特定平台/频道
hermes$ send_message --target telegram:@username "消息内容"

# 发送图片
hermes$ send_message --target discord:#general --message "图片来了" --media /path/to/image.png
```

**目标格式：**
- `telegram:@username` 或 `telegram:chat_id`
- `discord:#channel-name`
- `slack:#channel`
- `email:user@example.com`

---

## 数据工具（Data）

### json_parse

解析 JSON 字符串。

```python
# 在 execute_code 中使用
from hermes_tools import json_parse

data = json_parse('{"name": "test", "value": 123}')
print(data["name"])  # test
```

---

## 工具使用最佳实践

### 1. 优先使用专用工具

```
❌ terminal("cat file.txt")
✅ read_file("file.txt")
```

### 2. 使用 patch 而非 sed

```
❌ terminal("sed -i 's/old/new/g' file.txt")
✅ patch("file.txt", "old", "new")
```

### 3. 设置合理的超时

```bash
# 对于可能耗时的操作
hermes$ terminal "pip install large-package" --timeout 300
```

### 4. 检查工具返回

工具调用会返回结果，确保检查返回内容。

---

## 错误处理

| 错误 | 原因 | 解决 |
|------|------|------|
| `Permission denied` | 权限不足 | 检查文件权限或使用 sudo |
| `File not found` | 文件不存在 | 检查路径是否正确 |
| `Timeout` | 操作超时 | 增加 timeout 参数 |
| `Connection failed` | 网络问题 | 检查网络连接 |