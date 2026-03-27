# OpenHands 集成指南

> 与 Slack、Jira、Linear 等工具的集成

---

## 📋 目录

- [Slack 集成](#slack-集成)
- [Jira 集成](#jira-集成)
- [Linear 集成](#linear-集成)
- [GitHub 集成](#github-集成)
- [GitLab 集成](#gitlab-集成)

---

## 💬 Slack 集成

### 配置步骤

**1. 创建 Slack App**
```bash
1. 访问 https://api.slack.com/apps
2. 点击 "Create New App"
3. 选择 "From scratch"
4. 输入 App 名称和工作区
```

**2. 配置权限**
```json
{
  "scopes": [
    "chat:write",
    "channels:read",
    "groups:read",
    "mpim:read",
    "im:read"
  ]
}
```

**3. 安装到工作区**
```bash
1. 点击 "Install to Workspace"
2. 授权应用
3. 获取 Bot Token
```

**4. 配置 OpenHands**
```bash
# 在 OpenHands Cloud
Settings → Integrations → Slack
→ 输入 Bot Token
→ 选择默认频道
```

### 使用方式

**在 Slack 中使用**：
```bash
# 创建任务
/openhands create FastAPI 项目，包含用户认证

# 查看任务列表
/openhands list

# 查看任务详情
/openhands show <session-id>

# 取消任务
/openhands cancel <session-id>
```

**自动通知**：
```python
# 任务完成后自动通知
Settings → Notifications → Slack
→ 启用 "Task Completed"
→ 选择通知频道
```

---

## 📋 Jira 集成

### 配置步骤

**1. 获取 Jira API Token**
```bash
1. 登录 Jira
2. 访问 https://id.atlassian.com/manage-profile/security/api-tokens
3. 点击 "Create API token"
4. 输入标签并创建
5. 保存 Token
```

**2. 配置 OpenHands**
```bash
Settings → Integrations → Jira
→ 输入 Jira URL
→ 输入邮箱
→ 输入 API Token
→ 点击 "Test Connection"
```

**3. 选择项目**
```bash
→ 选择要集成的项目
→ 配置默认 Issue 类型
→ 设置字段映射
```

### 使用方式

**自动创建 Issue**：
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")
result = agent.run("""
创建 FastAPI 项目

Jira 配置：
- 项目：PROJ
- 类型：Task
- 标签：backend
""")

# Agent 完成后会自动创建 Jira Issue
```

**自动更新状态**：
```python
# 任务完成后自动更新
Settings → Jira → Automation
→ 启用 "Update on Complete"
→ 配置状态转换
```

---

## 📊 Linear 集成

### 配置步骤

**1. 获取 Linear API Key**
```bash
1. 登录 Linear
2. 访问 Settings → API
3. 点击 "Create API Key"
4. 选择权限范围
5. 保存 Key
```

**2. 配置 OpenHands**
```bash
Settings → Integrations → Linear
→ 输入 API Key
→ 选择团队
→ 配置默认项目
```

### 使用方式

**自动创建 Task**：
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")
result = agent.run("""
优化数据库查询性能

Linear 配置：
- 项目：Backend
- 优先级：High
- 标签：performance
""")
```

---

## 🐙 GitHub 集成

### 配置步骤

**1. 创建 GitHub Token**
```bash
1. 访问 https://github.com/settings/tokens
2. 点击 "Generate new token (classic)"
3. 选择权限：
   - repo
   - workflow
   - write:packages
4. 生成并保存 Token
```

**2. 配置 OpenHands**
```bash
Settings → Integrations → GitHub
→ 输入 Token
→ 选择仓库
→ 配置权限
```

### 使用方式

**自动创建 PR**：
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")
result = agent.run("""
实现用户登录功能

Git 配置：
- 分支：feature/user-login
- 提交信息：feat: 实现用户登录
- PR 标题：feat: 添加用户登录功能
""")

# Agent 会自动：
# 1. 创建分支
# 2. 编写代码
# 3. 提交更改
# 4. 创建 PR
```

**自动代码审查**：
```python
# PR 创建后自动审查
Settings → GitHub → Code Review
→ 启用 "Auto Review"
→ 配置审查规则
```

---

## 🦊 GitLab 集成

### 配置步骤

**1. 创建 GitLab Token**
```bash
1. 登录 GitLab
2. 访问 Settings → Access Tokens
3. 选择权限：
   - api
   - write_repository
4. 生成并保存 Token
```

**2. 配置 OpenHands**
```bash
Settings → Integrations → GitLab
→ 输入 GitLab URL
→ 输入 Token
→ 选择项目
```

### 使用方式

**自动创建 MR**：
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")
result = agent.run("""
添加单元测试

GitLab 配置：
- 分支：test/unit-tests
- MR 标题：test: 添加单元测试
- 标签：testing
""")
```

---

## 🔧 高级配置

### 1. 多工作区集成

```python
from openhands import Agent

# 配置多个 Slack 工作区
agent = Agent(model="claude-3.5-sonnet")
agent.configure_slack([
    {
        "workspace": "company-a",
        "token": "xoxb-...",
        "default_channel": "#dev"
    },
    {
        "workspace": "company-b",
        "token": "xoxb-...",
        "default_channel": "#engineering"
    }
])
```

### 2. 条件触发

```python
# 根据任务类型选择集成
def smart_integration(task_type):
    if task_type == "bug":
        # 自动创建 Jira Bug
        return {"jira": {"type": "Bug"}}
    elif task_type == "feature":
        # 自动创建 GitHub PR
        return {"github": {"draft": True}}
    else:
        return {}
```

### 3. 通知规则

```python
# 配置通知规则
notification_rules = {
    "on_start": ["slack"],
    "on_complete": ["slack", "jira", "email"],
    "on_error": ["slack", "pagerduty"],
    "on_timeout": ["slack"]
}
```

---

## 📊 监控和分析

### 1. 使用统计

```python
from openhands import Analytics

# 获取集成使用统计
analytics = Analytics()
stats = analytics.get_integration_stats()

print(f"Slack 消息数: {stats['slack']['messages']}")
print(f"Jira Issues 数: {stats['jira']['issues']}")
print(f"GitHub PRs 数: {stats['github']['prs']}")
```

### 2. 性能监控

```python
# 监控集成性能
from openhands import Monitor

monitor = Monitor()
monitor.track_integration("slack", "message_sent")
monitor.track_integration("jira", "issue_created")
```

---

## 🔒 安全最佳实践

### 1. Token 管理

```python
# 使用环境变量
import os

SLACK_TOKEN = os.getenv("SLACK_BOT_TOKEN")
JIRA_TOKEN = os.getenv("JIRA_API_TOKEN")

# 定期轮换
# 每 90 天轮换一次 Token
```

### 2. 权限最小化

```bash
# 只授予必要的权限
Slack: chat:write, channels:read
Jira: project:read, issue:create
GitHub: repo, workflow
```

### 3. 审计日志

```python
# 启用审计日志
Settings → Security → Audit Log
→ 启用 "Log all integration activities"
→ 配置保留时间：90 天
```

---

## 🆘 故障排查

### 问题 1：Slack 通知未发送

**解决方案**：
```bash
1. 检查 Bot Token 是否有效
2. 确认 Bot 在频道中
3. 检查权限配置
4. 查看错误日志
```

### 问题 2：Jira Issue 创建失败

**解决方案**：
```bash
1. 检查 API Token 是否过期
2. 确认项目权限
3. 检查字段映射
4. 验证必填字段
```

### 问题 3：GitHub PR 创建失败

**解决方案**：
```bash
1. 检查 Token 权限
2. 确认仓库访问权限
3. 检查分支保护规则
4. 验证提交签名
```

---

## 📚 相关资源

- [Slack API 文档](https://api.slack.com)
- [Jira API 文档](https://developer.atlassian.com/cloud/jira)
- [Linear API 文档](https://developers.linear.app)
- [GitHub API 文档](https://docs.github.com/rest)
- [GitLab API 文档](https://docs.gitlab.com/ee/api)

---

<div align="center">
  <p>🎉 集成你的工具，提升工作效率！</p>
</div>
