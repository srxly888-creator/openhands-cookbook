# Agent 配置指南

> 如何配置和优化 OpenHands Agent

---

## 📋 目录

- [基础配置](#基础配置)
- [高级配置](#高级配置)
- [性能优化](#性能优化)
- [最佳实践](#最佳实践)

---

## 🔧 基础配置

### 1. 创建配置文件

**`.openhands.toml`**：
```toml
[agent]
model = "claude-3.5-sonnet"
max_tokens = 4096
temperature = 0.7

[execution]
timeout = 300
max_retries = 3
retry_delay = 5

[logging]
level = "INFO"
file = "~/.openhands/openhands.log"

[workspace]
directory = "."
exclude = ["node_modules", "venv", "__pycache__"]
```

### 2. 环境变量配置

**`.env`**：
```bash
# API Keys
ANTHROPIC_API_KEY=sk-ant-xxx
OPENAI_API_KEY=sk-xxx
GLM_API_KEY=xxx.xxx

# Agent 配置
OPENHANDS_MODEL=claude-3.5-sonnet
OPENHANDS_MAX_TOKENS=4096
OPENHANDS_TEMPERATURE=0.7

# 执行配置
OPENHANDS_TIMEOUT=300
OPENHANDS_MAX_RETRIES=3
```

### 3. Python 代码配置

```python
from openhands import Agent, Config

# 方式一：直接配置
agent = Agent(
    model="claude-3.5-sonnet",
    api_key="sk-ant-xxx",
    max_tokens=4096,
    temperature=0.7
)

# 方式二：使用配置文件
config = Config.from_file(".openhands.toml")
agent = Agent(config=config)

# 方式三：环境变量
agent = Agent()  # 自动读取环境变量
```

---

## 🎯 高级配置

### 1. 模型参数

```python
agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=4096,
    temperature=0.7,
    top_p=0.9,
    frequency_penalty=0.0,
    presence_penalty=0.0,
    stop_sequences=["```", "---"]
)
```

**参数说明**：
- **max_tokens**: 最大输出 token 数
- **temperature**: 随机性（0-2，越高越随机）
- **top_p**: 核采样（0-1）
- **frequency_penalty**: 频率惩罚（-2 to 2）
- **presence_penalty**: 存在惩罚（-2 to 2）

### 2. 执行配置

```python
agent = Agent(
    model="claude-3.5-sonnet",
    timeout=300,  # 超时时间（秒）
    max_retries=3,  # 最大重试次数
    retry_delay=5,  # 重试延迟（秒）
    enable_parallel=True,  # 启用并行执行
    max_workers=4  # 最大并行数
)
```

### 3. 工作区配置

```python
agent = Agent(
    model="claude-3.5-sonnet",
    working_dir="./my-project",
    exclude_patterns=[
        "node_modules/*",
        "venv/*",
        "__pycache__/*",
        "*.pyc",
        ".git/*"
    ],
    include_patterns=[
        "src/**/*.py",
        "tests/**/*.py"
    ]
)
```

### 4. 工具配置

```python
from openhands import Agent, Tool

# 禁用特定工具
agent = Agent(
    model="claude-3.5-sonnet",
    disabled_tools=["git_push", "delete_file"]
)

# 自定义工具
class MyTool(Tool):
    name = "my_custom_tool"
    description = "我的自定义工具"

    def run(self, arg):
        return f"处理 {arg}"

agent = Agent(model="claude-3.5-sonnet")
agent.register_tool(MyTool())
```

---

## ⚡ 性能优化

### 1. 缓存配置

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    enable_cache=True,
    cache_ttl=3600,  # 缓存有效期（秒）
    cache_dir="~/.openhands/cache"
)
```

### 2. 并行执行

```python
from openhands import Agent
import asyncio

async def parallel_tasks():
    agent = Agent(
        model="claude-3.5-sonnet",
        enable_parallel=True,
        max_workers=4
    )
    
    tasks = [
        agent.run_async("任务1"),
        agent.run_async("任务2"),
        agent.run_async("任务3")
    ]
    
    results = await asyncio.gather(*tasks)
    return results

results = asyncio.run(parallel_tasks())
```

### 3. 流式处理

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 流式处理
for chunk in agent.stream("创建 FastAPI 项目"):
    print(chunk, end='', flush=True)
```

### 4. Token 优化

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=2048,  # 减少最大 token
    enable_compression=True,  # 启用压缩
    compression_ratio=0.7  # 压缩比例
)
```

---

## 🎓 最佳实践

### 1. 按场景配置

```python
# 复杂任务 - 高质量
agent_complex = Agent(
    model="claude-3.5-sonnet",
    max_tokens=4096,
    temperature=0.5
)

# 简单任务 - 快速
agent_simple = Agent(
    model="gpt-4o",
    max_tokens=2048,
    temperature=0.3
)

# 中文任务 - 性价比
agent_chinese = Agent(
    model="glm-4",
    max_tokens=2048,
    temperature=0.6
)
```

### 2. 错误处理

```python
from openhands import Agent, OpenHandsError

agent = Agent(
    model="claude-3.5-sonnet",
    max_retries=3,
    retry_delay=5,
    enable_fallback=True,
    fallback_model="gpt-4o"
)

try:
    result = agent.run("任务")
except OpenHandsError as e:
    print(f"错误: {e}")
    # 使用备用模型
    agent.model = "gpt-4o"
    result = agent.run("任务")
```

### 3. 监控和日志

```python
from openhands import Agent, Monitor

# 启用监控
monitor = Monitor()
agent = Agent(
    model="claude-3.5-sonnet",
    enable_monitoring=True,
    monitor=monitor
)

# 运行任务
result = agent.run("任务")

# 查看监控数据
stats = monitor.get_stats()
print(f"Token 使用: {stats['tokens']}")
print(f"执行时间: {stats['duration']}")
print(f"成本: ${stats['cost']}")
```

### 4. 配置管理

```python
from openhands import ConfigManager

# 加载多个配置文件
config_manager = ConfigManager()
config_manager.load(".openhands.toml")
config_manager.load(".env", override=True)

# 获取配置
config = config_manager.get_config()
agent = Agent(config=config)

# 动态更新配置
config_manager.set("agent.temperature", 0.8)
agent.update_config(config)
```

---

## 📊 配置示例

### 开发环境配置

```toml
# .openhands.dev.toml
[agent]
model = "gpt-4o"
max_tokens = 2048
temperature = 0.5

[execution]
timeout = 60
max_retries = 1

[logging]
level = "DEBUG"
```

### 生产环境配置

```toml
# .openhands.prod.toml
[agent]
model = "claude-3.5-sonnet"
max_tokens = 4096
temperature = 0.3

[execution]
timeout = 300
max_retries = 3

[logging]
level = "WARNING"

[monitoring]
enabled = true
metrics = true
```

---

## 🔧 故障排查

### 问题 1：配置不生效

**解决方案**：
```bash
# 检查配置文件位置
ls -la .openhands.toml

# 验证配置格式
openhands config validate

# 查看当前配置
openhands config show
```

### 问题 2：性能不佳

**解决方案**：
```python
# 启用缓存
agent = Agent(enable_cache=True)

# 启用并行
agent = Agent(enable_parallel=True, max_workers=4)

# 减少超时时间
agent = Agent(timeout=60)
```

### 问题 3：成本过高

**解决方案**：
```python
# 使用更便宜的模型
agent = Agent(model="glm-4")

# 减少 token
agent = Agent(max_tokens=2048)

# 启用缓存
agent = Agent(enable_cache=True)
```

---

## 📚 相关资源

- [配置文件参考](https://docs.openhands.dev/config/reference)
- [性能优化指南](https://docs.openhands.dev/performance)
- [最佳实践](https://docs.openhands.dev/best-practices)

---

<div align="center">
  <p>🎉 优化配置，提升性能！</p>
</div>
