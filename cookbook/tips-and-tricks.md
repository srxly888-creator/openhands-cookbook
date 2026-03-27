# OpenHands 使用技巧集

> 高效使用 OpenHands 的技巧和窍门

---

## 📋 目录

- [任务描述技巧](#任务描述技巧)
- [快捷键和命令](#快捷键和命令)
- [工作流优化](#工作流优化)
- [故障排查技巧](#故障排查技巧)

---

## 💬 任务描述技巧

### 1. 清晰具体

❌ **不好的描述**：
```
优化代码
```

✅ **好的描述**：
```
优化 main.py 中的数据处理函数：
1. 减少内存占用（目标：<100MB）
2. 提高处理速度（目标：<1秒）
3. 保持代码可读性
4. 添加单元测试
```

### 2. 提供上下文

❌ **不好的描述**：
```
修复 Bug
```

✅ **好的描述**：
```
修复用户登录 Bug：

错误信息：
RuntimeError: Cannot connect to database

复现步骤：
1. 访问 /login
2. 输入用户名和密码
3. 点击登录

环境：
- Python 3.11
- PostgreSQL 15
- FastAPI 0.104

期望结果：
成功登录并跳转到首页
```

### 3. 分步骤执行

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 第一步：创建项目结构
agent.run("创建 FastAPI 项目结构")

# 第二步：实现用户认证
agent.run("实现用户注册和登录功能")

# 第三步：添加测试
agent.run("为用户认证功能编写单元测试")
```

### 4. 使用模板

```python
from openhands import Agent

# 创建模板
template = """
创建 {project_type} 项目：
1. 项目名称：{name}
2. 主要功能：{features}
3. 技术栈：{tech_stack}
4. 测试要求：{test_requirement}
"""

agent = Agent(model="claude-3.5-sonnet")

# 使用模板
result = agent.run(template.format(
    project_type="FastAPI",
    name="my-api",
    features="用户认证、数据库、API 文档",
    tech_stack="FastAPI、PostgreSQL、Redis",
    test_requirement="单元测试覆盖率 > 80%"
))
```

---

## ⌨️ 快捷键和命令

### CLI 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd + Enter` | 发送消息 |
| `Cmd + C` | 取消当前任务 |
| `Cmd + L` | 清屏 |
| `Cmd + ↑` | 上一条命令 |
| `Cmd + ↓` | 下一条命令 |
| `Tab` | 自动补全 |

### 常用命令

```bash
# 查看帮助
openhands --help

# 查看版本
openhands --version

# 查看配置
openhands config show

# 查看日志
openhands logs --tail 100

# 清理缓存
openhands cache clear

# 更新
pip install --upgrade openhands
```

### GUI 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd + N` | 新建会话 |
| `Cmd + S` | 保存会话 |
| `Cmd + W` | 关闭会话 |
| `Cmd + /` | 显示帮助 |
| `Cmd + K` | 搜索会话 |

---

## 🔄 工作流优化

### 1. Git 集成工作流

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 1. 创建分支
agent.execute_command("git checkout -b feature/user-auth")

# 2. 执行任务
result = agent.run("实现用户登录功能")

# 3. 运行测试
test_result = agent.execute_command("pytest tests/")

# 4. 提交代码
agent.execute_command("git add .")
agent.execute_command('git commit -m "feat: 实现用户登录功能"')

# 5. 创建 PR
agent.create_pr(
    title="feat: 实现用户登录功能",
    body="详细描述..."
)
```

### 2. 批量处理工作流

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 获取所有 Python 文件
files = agent.list_files("./src", pattern="*.py")

# 批量优化
for file in files:
    agent.run(f"优化 {file} 的性能")

# 批量测试
agent.execute_command("pytest tests/ -v")
```

### 3. 自动化测试工作流

```python
from openhands import Agent

def automated_testing():
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 拉取最新代码
    agent.execute_command("git pull origin main")
    
    # 2. 安装依赖
    agent.execute_command("pip install -r requirements.txt")
    
    # 3. 运行测试
    result = agent.execute_command("pytest tests/ --cov=src")
    
    # 4. 生成报告
    if "passed" in result:
        print("✅ 测试通过")
    else:
        print("❌ 测试失败")
        # 发送通知
        agent.execute_command("python notify.py 'Tests failed'")

# 定时执行
import schedule
schedule.every().day.at("09:00").do(automated_testing)
```

---

## 🔍 故障排查技巧

### 1. 日志分析

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    log_level="DEBUG",
    log_file="~/.openhands/debug.log"
)

# 运行任务
result = agent.run("创建 FastAPI 项目")

# 查看日志
agent.show_logs()
```

### 2. 错误重试

```python
from openhands import Agent, OpenHandsError

agent = Agent(
    model="claude-3.5-sonnet",
    max_retries=3,
    retry_delay=5
)

try:
    result = agent.run("创建项目")
except OpenHandsError as e:
    print(f"错误: {e}")
    # 降级到更简单的模型
    agent.model = "gpt-4o"
    result = agent.run("创建项目")
```

### 3. 调试模式

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    debug=True,  # 启用调试模式
    verbose=True  # 详细输出
)

# 每一步都会输出详细信息
result = agent.run("创建 FastAPI 项目")
```

### 4. 性能分析

```python
from openhands import Agent, Profiler

profiler = Profiler()
agent = Agent(
    model="claude-3.5-sonnet",
    profiler=profiler
)

result = agent.run("创建 FastAPI 项目")

# 查看性能报告
report = profiler.get_report()
print(f"总耗时: {report['duration']}s")
print(f"慢操作: {report['slow_operations']}")
```

---

## 🎯 实用技巧

### 1. 智能模型选择

```python
from openhands import Agent

def smart_agent(task):
    # 根据任务复杂度选择模型
    if "架构" in task or "设计" in task:
        return Agent(model="claude-3.5-sonnet")
    elif "中文" in task:
        return Agent(model="glm-4")
    else:
        return Agent(model="gpt-4o")

# 使用
agent = smart_agent("设计微服务架构")
result = agent.run("设计微服务架构")
```

### 2. 代码片段复用

```python
from openhands import Agent

# 保存常用代码片段
snippets = {
    "fastapi_basic": """
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
    """,
    "pytest_basic": """
import pytest

def test_example():
    assert True
    """
}

agent = Agent(model="claude-3.5-sonnet")

# 使用代码片段
result = agent.run(f"""
创建 FastAPI 项目，包含以下代码：

{snippets['fastapi_basic']}
""")
```

### 3. 环境检测

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 检测环境
env_check = agent.run("""
检测当前环境：
1. Python 版本
2. 已安装的包
3. 系统信息
4. 网络连接
""")

print(env_check.output)
```

---

## 📚 相关资源

- [官方文档](https://docs.openhands.dev)
- [最佳实践](https://docs.openhands.dev/best-practices)
- [社区分享](https://github.com/OpenHands/OpenHands/discussions)

---

<div align="center">
  <p>🎉 掌握技巧，提升效率！</p>
</div>
