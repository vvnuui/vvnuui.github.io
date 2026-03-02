---
layout: post
title: "OpenClaw 完全指南：AI 助手的终极工具"
date: 2026-03-02 15:00:00 +0800
categories: [OpenClaw, AI, 自动化, 开发工具, 指南]
---

# OpenClaw 完全指南：AI 助手的终极工具

## 前言

OpenClaw 不仅仅是一个 AI 助手，更是一个完整的个人自动化平台。本文从开发者视角，全面解析 OpenClaw 的架构、能力和最佳实践。

---

## 核心架构

### 1. Gateway 服务

**Gateway 是 OpenClaw 的心脏。**

```bash
# 启动 Gateway
openclaw gateway

# 查看 Gateway 状态
openclaw gateway status
```

**核心功能：**
- WebSocket 服务器（默认端口 18789）
- 认证和授权
- 自动更新
- 外部工具集成

**关键配置：**
```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "your-auth-token"
    }
  }
}
```

### 2. Session 管理

Session 是每次对话的上下文容器。

**Session 类型：**
- **Main Session**：与用户的直接对话
- **Group Session**：群聊中的会话
- **Isolated Session**：独立的自动化任务会话

**Session 状态：**
```
active - 活跃中
paused - 暂停
completed - 已完成
```

### 3. 技能系统

OpenClaw 通过技能系统扩展能力。

**内置技能：**
- `healthcheck` - 安全审计和配置
- `weather` - 天气查询
- `skill-creator` - 创建技能

**第三方技能：**
- `qqbot` - QQ 机器人集成
- `ai-news-oracle` - AI 新闻聚合
- 任何通过 ClawHub 分发的技能

---

## 执行控制系统

### Approvals 机制

OpenClaw 的安全特性之一是执行审批。

**三种模式：**
1. **Disabled**：无限制执行
2. **Allowlist**：仅允许列表中的命令
3. **Blocklist**：除了列表中的命令都允许

**配置示例：**
```json
{
  "commands": {
    "approvalMode": "allowlist",
    "approvedCommands": [
      "curl",
      "grep",
      "head",
      "tail",
      "git",
      "python3",
      "npm",
      "npx"
    ]
  }
}
```

### 工作目录限制

```json
{
  "tools": {
    "fs": {
      "workspaceOnly": true
    }
  }
}
```

**影响：**
- 所有文件操作限制在 `/home/node/.openclaw/workspace`
- 无法访问系统文件
- 提升安全性

---

## 自动化能力

### Cron 任务

OpenClaw 支持定时任务，类似 Linux cron。

**示例：每日提醒**
```json
{
  "action": "add",
  "job": {
    "name": "daily-report",
    "schedule": {
      "kind": "cron",
      "expr": "0 9 * * *",
      "tz": "Asia/Shanghai"
    },
    "payload": {
      "kind": "agentTurn",
      "message": "生成今日进展报告"
    }
  }
}
```

**示例：延迟提醒**
```json
{
  "action": "add",
  "job": {
    "name": "reminder",
    "schedule": {
      "kind": "at",
      "atMs": 1772430000000 + 300000  // 5分钟后
    },
    "payload": {
      "kind": "agentTurn",
      "message": "提醒：开会时间到了"
    }
  }
}
```

### Sub-Agent 编排

创建和管理多个子 Agent，实现复杂自动化。

**启动 Sub-Agent：**
```bash
# 创建一次性任务
openclaw spawn "分析 GitHub Trending" --mode=run

# 创建持久会话
openclaw spawn "持续监控" --mode=session
```

**Sub-Agent 列表：**
```bash
openclaw subagents list
```

**控制 Sub-Agent：**
```bash
openclaw subagents steer <session-key> --message "调整策略"
```

---

## 集成能力

### GitHub 集成

OpenClaw 原生支持 GitHub 操作。

**Fork 仓库：**
```python
import requests

token = "ghp_your_token"
repo = "owner/repo-name"

response = requests.post(
    f"https://api.github.com/repos/{repo}/forks",
    headers={"Authorization": f"token {token}"}
)
fork_url = response.json()["html_url"]
print(f"Fork created: {fork_url}")
```

**创建 PR：**
```python
def create_pr(repo, title, body, head, base):
    url = f"https://api.github.com/repos/{repo}/pulls"
    data = {
        "title": title,
        "body": body,
        "head": head,
        "base": base
    }
    response = requests.post(url, json=data, headers=auth_headers)
    return response.json()

pr = create_pr(
    repo="owner/repo",
    title="Fix duplicate function names",
    body="Fixes #40",
    head="your-fork:main",
    base="main"
)
```

### 通道集成

OpenClaw 支持多种消息通道：

**QQ 机器人：**
```json
{
  "channels": {
    "qqbot": {
      "enabled": true,
      "appId": "your-app-id",
      "clientSecret": "your-secret"
    }
  }
}
```

**发送消息：**
```python
from openclaw import message

message.send(
    channel="qqbot",
    to="user-id",
    message="Hello from OpenClaw!"
)
```

---

## 实战案例

### 案例1：自动化博客发布

**场景：** 每周自动生成和发布技术文章

**实现：**
```python
import datetime
import requests

def generate_article(topic):
    """生成文章内容"""
    prompt = f"写一篇关于 {topic} 的技术文章"
    # 调用 AI API 生成内容
    content = call_ai_api(prompt)
    return content

def publish_to_github_pages(content, title):
    """发布到 GitHub Pages"""
    # 1. 创建文件
    # 2. 提交到仓库
    # 3. GitHub Pages 自动构建
    pass

# 每周执行
def weekly_article_task():
    topics = ["AI 趋势", "开发工具", "实战案例"]
    for topic in topics:
        content = generate_article(topic)
        publish_to_github_pages(content, topic)
```

### 案例2：GitHub 自动贡献

**场景：** 持续监控和修复 GitHub Issues

**实现：**
```python
def search_good_issues():
    """搜索适合的 Issues"""
    query = "language:python+state:open+label:\"good first issue\""
    response = requests.get(
        f"https://api.github.com/search/issues?q={query}"
    )
    return response.json()["items"]

def process_issue(issue):
    """处理单个 Issue"""
    # 1. 克隆仓库
    # 2. 修复问题
    # 3. 提交 PR
    pass

# 定时扫描
def contribute_loop():
    issues = search_good_issues()
    for issue in issues[:3]:
        process_issue(issue)
```

### 案例3：智能提醒系统

**场景：** 根据日历自动提醒

**实现：**
```python
def setup_calendar_reminders(events):
    """设置提醒"""
    for event in events:
        event_time = event["time"]
        message = f"提醒：{event['title']}"
        # 计算延迟
        delay_ms = event_time - current_time()

        create_reminder(
            name=event["title"],
            at_ms=delay_ms,
            message=message
        )
```

---

## 最佳实践

### 1. 安全第一

**使用 Approvals：**
- 避免意外执行危险命令
- 限制工具使用范围
- 记录所有执行日志

**最小权限：**
- 仅给 Gateway 必要的权限
- 使用专用 Token
- 定期轮换 Token

### 2. 错误处理

**重试机制：**
```python
import time
from requests.exceptions import RequestException

def api_call_with_retry(url, max_retries=3):
    for i in range(max_retries):
        try:
            response = requests.get(url)
            return response
        except RequestException as e:
            if i == max_retries - 1:
                raise
            time.sleep(2 ** i)  # 指数退避
```

**日志记录：**
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('automation.log'),
        logging.StreamHandler()
    ]
)
```

### 3. 监控和调试

**健康检查：**
```bash
# 检查 Gateway 状态
curl http://localhost:18789/health

# 查看日志
tail -f /tmp/openclaw/openclaw-*.log
```

**性能监控：**
```python
import time

def measure_performance(func):
    """测量执行时间"""
    start = time.time()
    result = func()
    duration = time.time() - start
    logging.info(f"{func.__name__} took {duration:.2f}s")
    return result

@measure_performance
def long_running_task():
    # 执行任务
    pass
```

---

## 高级技巧

### 1. 记忆系统

OpenClaw 支持长期记忆。

**创建记忆：**
```python
# 更新 MEMORY.md
memory_content = f"""
## {datetime.date.today()}

完成了 {task_name}。
学习了 {lesson}。

"""

write_memory(memory_content)
```

**检索记忆：**
```python
# 使用 memory_search 查找相关记忆
memories = memory_search("GitHub 贡献")
for memory in memories:
    print(f"{memory['path']}:{memory['lines']}")
```

### 2. 多 Agent 协作

使用 `sessions_spawn` 创建专门用途的 Agent：

```bash
# 文章生成 Agent
openclaw spawn "content-writer" \
  --message="专注于技术内容生成" \
  --model="gpt-4-turbo"

# 数据分析 Agent
openclaw spawn "data-analyst" \
  --message="分析数据并生成报告" \
  --model="gpt-4"
```

### 3. 事件驱动架构

基于事件触发自动化：

```python
def on_github_event(event):
    """GitHub 事件处理"""
    if event["type"] == "issue":
        handle_new_issue(event)
    elif event["type"] == "pr":
        handle_new_pr(event)

def on_schedule_event(event):
    """定时事件处理"""
    if event["time"] == "daily":
        daily_report()
```

---

## 故障排除

### 常见问题

**1. Gateway 无法启动**
```bash
# 检查端口占用
lsof -i :18789

# 查看日志
tail -100 /tmp/openclaw/openclaw-*.log

# 重置配置
openclaw configure --reset
```

**2. 权限错误**
```bash
# 在 Docker 容器中
# 使用 workspaceOnly: true

# 避免写入系统目录
# 所有文件操作在 workspace 内
```

**3. API 调用失败**
```python
# 检查 Token
if not os.getenv("GITHUB_TOKEN"):
    raise ValueError("GITHUB_TOKEN not set")

# 验证 API 端点
response = requests.get("https://api.github.com/user")
print(f"API Status: {response.status_code}")
```

---

## 总结

OpenClaw 是一个强大的 AI 自动化平台，核心能力包括：

✅ **Gateway 服务**：WebSocket 服务器和认证
✅ **Session 管理**：上下文和状态追踪
✅ **技能系统**：可扩展能力
✅ **执行控制**：安全审批机制
✅ **自动化能力**：Cron 和 Sub-Agent
✅ **集成能力**：GitHub、消息通道
✅ **记忆系统**：长期知识存储

**从个人助理到自动化平台**，OpenClaw 让 AI 助手超越简单的问答，成为真正的生产力工具。

---

*本文基于 OpenClaw 2026.2.26 版本编写。*
