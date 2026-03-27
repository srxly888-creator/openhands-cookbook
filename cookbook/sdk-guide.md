# OpenHands SDK 编程指南

> 使用 Python SDK 编程控制 Agent

---

## 📋 目录

- [快速开始](#快速开始)
- [基础概念](#基础概念)
- [核心 API](#核心-api)
- [实战案例](#实战案例)

---

## 🚀 快速开始

### 安装 SDK

```bash
pip install openhands-sdk
```

### 第一个程序

```python
from openhands import Agent

# 创建 Agent
agent = Agent(model="claude-3.5-sonnet")

# 执行任务
result = agent.run("创建一个 FastAPI 项目")

print(result)
```

---

## 📖 基础概念

### 1. Agent

**Agent 是核心概念**：
- 代表一个 AI 智能体
- 可以执行各种任务
- 支持多种模型

**创建 Agent**：
```python
from openhands import Agent

# 基础创建
agent = Agent(model="claude-3.5-sonnet")

# 带配置创建
agent = Agent(
    model="claude-3.5-sonnet",
    api_key="your-api-key",
    max_tokens=4096,
    temperature=0.7
)
```

### 2. Task

**Task 代表一个任务**：
- 包含任务描述
- 可选参数
- 预期输出

**创建 Task**：
```python
from openhands import Task

# 基础任务
task = Task("创建 FastAPI 项目")

# 带参数任务
task = Task(
    description="创建 FastAPI 项目",
    working_dir="./my-project",
    files=["requirements.txt"],
    timeout=300
)
```

### 3. Result

**Result 包含执行结果**：
- 成功/失败状态
- 输出内容
- 错误信息

**处理 Result**：
```python
result = agent.run(task)

if result.success:
    print("✅ 成功:", result.output)
else:
    print("❌ 失败:", result.error)
```

---

## 🔧 核心 API

### 1. 运行任务

```python
# 同步运行
result = agent.run("创建 FastAPI 项目")

# 异步运行
import asyncio

async def run_task():
    result = await agent.run_async("创建 FastAPI 项目")
    return result

result = asyncio.run(run_task())
```

### 2. 文件操作

```python
# 读取文件
content = agent.read_file("main.py")

# 写入文件
agent.write_file("main.py", "print('Hello, World!')")

# 删除文件
agent.delete_file("old_file.py")

# 列出文件
files = agent.list_files("./src")
```

### 3. 终端操作

```python
# 执行命令
output = agent.execute_command("pip install fastapi")

# 执行并等待
output = agent.execute_command("pytest tests/", timeout=60)

# 后台执行
process = agent.execute_command_async("python server.py")
```

### 4. Git 操作

```python
# 初始化仓库
agent.git_init()

# 添加文件
agent.git_add(".")

# 提交
agent.git_commit("feat: 添加新功能")

# 推送
agent.git_push("origin", "main")

# 创建 PR
agent.create_pr(
    title="feat: 添加新功能",
    body="详细描述"
)
```

---

## 🎯 实战案例

### 案例 1：自动化项目创建

```python
from openhands import Agent

def create_fastapi_project(project_name):
    agent = Agent(model="claude-3.5-sonnet")

    # 创建项目
    result = agent.run(f"""
    创建一个 FastAPI 项目 {project_name}，包含：
    1. 用户注册/登录 API
    2. JWT 认证
    3. SQLite 数据库
    4. 单元测试
    5. README.md
    """)

    if result.success:
        print(f"✅ 项目 {project_name} 创建成功！")
        return True
    else:
        print(f"❌ 创建失败: {result.error}")
        return False

# 使用
create_fastapi_project("my-api")
```

### 案例 2：批量代码审查

```python
import os
from openhands import Agent

def review_code(directory):
    agent = Agent(model="claude-3.5-sonnet")

    # 获取所有 Python 文件
    python_files = [
        f for f in os.listdir(directory)
        if f.endswith('.py')
    ]

    reviews = []
    for file in python_files:
        # 读取文件
        content = agent.read_file(os.path.join(directory, file))

        # 审查代码
        result = agent.run(f"""
        审查以下代码并提供改进建议：

        文件：{file}
        代码：
        {content}
        """)

        reviews.append({
            'file': file,
            'review': result.output
        })

    return reviews

# 使用
reviews = review_code("./src")
for review in reviews:
    print(f"\n📄 {review['file']}:")
    print(review['review'])
```

### 案例 3：自动化测试

```python
from openhands import Agent

def generate_tests(source_file):
    agent = Agent(model="claude-3.5-sonnet")

    # 读取源代码
    code = agent.read_file(source_file)

    # 生成测试
    result = agent.run(f"""
    为以下代码生成单元测试：

    代码：
    {code}

    要求：
    1. 测试覆盖率 > 90%
    2. 包含边界条件测试
    3. 包含异常测试
    """)

    if result.success:
        # 保存测试文件
        test_file = source_file.replace('.py', '_test.py')
        agent.write_file(test_file, result.output)
        print(f"✅ 测试文件已生成: {test_file}")
        return test_file
    else:
        print(f"❌ 生成失败: {result.error}")
        return None

# 使用
generate_tests("main.py")
```

### 案例 4：持续集成

```python
from openhands import Agent
import subprocess

def ci_pipeline():
    agent = Agent(model="claude-3.5-sonnet")

    # 1. 拉取最新代码
    agent.git_pull("origin", "main")

    # 2. 安装依赖
    agent.execute_command("pip install -r requirements.txt")

    # 3. 运行测试
    result = agent.execute_command("pytest tests/")

    if "passed" in result:
        print("✅ 测试通过")

        # 4. 部署
        agent.execute_command("python deploy.py")
        print("✅ 部署成功")
    else:
        print("❌ 测试失败")
        # 发送通知
        agent.execute_command("python notify.py 'Tests failed'")

# 使用
ci_pipeline()
```

---

## 🎓 高级用法

### 1. 自定义工具

```python
from openhands import Agent, Tool

# 定义自定义工具
class MyTool(Tool):
    name = "my_tool"
    description = "我的自定义工具"

    def run(self, arg1, arg2):
        # 实现工具逻辑
        return f"处理 {arg1} 和 {arg2}"

# 注册工具
agent = Agent(model="claude-3.5-sonnet")
agent.register_tool(MyTool())

# 使用工具
agent.run("使用 my_tool 处理数据")
```

### 2. 流式输出

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 流式处理
for chunk in agent.stream("创建 FastAPI 项目"):
    print(chunk, end='', flush=True)
```

### 3. 错误处理

```python
from openhands import Agent, OpenHandsError

agent = Agent(model="claude-3.5-sonnet")

try:
    result = agent.run("创建项目")
except OpenHandsError as e:
    print(f"❌ 错误: {e}")
    # 重试逻辑
    result = agent.run("创建项目")
```

---

## 📚 API 参考

### Agent 类

```python
class Agent:
    def __init__(
        self,
        model: str = "claude-3.5-sonnet",
        api_key: str = None,
        max_tokens: int = 4096,
        temperature: float = 0.7
    ): ...

    def run(self, task: str) -> Result: ...
    def run_async(self, task: str) -> Result: ...
    def read_file(self, path: str) -> str: ...
    def write_file(self, path: str, content: str): ...
    def execute_command(self, cmd: str) -> str: ...
```

### Task 类

```python
class Task:
    def __init__(
        self,
        description: str,
        working_dir: str = ".",
        files: List[str] = None,
        timeout: int = 300
    ): ...
```

### Result 类

```python
class Result:
    success: bool
    output: str
    error: str
    metadata: dict
```

---

## 🔗 相关资源

- [官方 SDK 文档](https://docs.openhands.dev/sdk)
- [GitHub 仓库](https://github.com/OpenHands/software-agent-sdk)
- [API 参考](https://docs.openhands.dev/api)

---

<div align="center">
  <p>🎉 掌握 SDK，打造自动化工作流！</p>
</div>
