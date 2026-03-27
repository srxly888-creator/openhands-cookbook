# OpenHands API 参考手册

> 完整的 Python API 参考

---

## 📋 目录

- [核心类](#核心类)
- [配置类](#配置类)
- [工具类](#工具类)
- [异常类](#异常类)

---

## 🎯 核心类

### Agent

主 Agent 类。

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",  # 模型名称
    max_tokens=4096,            # 最大 token
    temperature=0.7,            # 温度
    enable_cache=True,          # 启用缓存
    cache_ttl=3600,             # 缓存时间（秒）
    enable_parallel=True,       # 启用并行
    max_workers=4,              # 最大并发数
    timeout=300,                # 超时时间（秒）
    max_retries=3,              # 最大重试次数
    debug=False                 # 调试模式
)
```

**方法**:

#### run(task: str) -> Result

同步执行任务。

```python
result = agent.run("创建 FastAPI 项目")
print(result.output)
```

**参数**:
- `task` - 任务描述

**返回**:
- `Result` - 执行结果

---

#### run_async(task: str) -> Result

异步执行任务。

```python
import asyncio

async def main():
    result = await agent.run_async("创建 FastAPI 项目")
    print(result.output)

asyncio.run(main())
```

**参数**:
- `task` - 任务描述

**返回**:
- `Result` - 执行结果

---

#### run_stream(task: str) -> AsyncGenerator

流式执行任务。

```python
async def main():
    async for chunk in agent.run_stream("创建 FastAPI 项目"):
        print(chunk, end="", flush=True)

asyncio.run(main())
```

**参数**:
- `task` - 任务描述

**返回**:
- `AsyncGenerator` - 流式结果

---

#### batch(tasks: List[str]) -> List[Result]

批量执行任务。

```python
tasks = ["任务1", "任务2", "任务3"]
results = agent.batch(tasks)

for result in results:
    print(result.output)
```

**参数**:
- `tasks` - 任务列表

**返回**:
- `List[Result]` - 结果列表

---

#### clear_cache()

清除缓存。

```python
agent.clear_cache()
```

---

### Result

执行结果类。

```python
from openhands import Result

result = agent.run("任务")

# 属性
result.output     # 输出内容
result.status     # 状态（success/failed）
result.duration   # 执行时间（秒）
result.tokens     # Token 使用量
result.cost       # 成本（美元）
result.metadata   # 元数据
```

**方法**:

#### to_dict() -> Dict

转换为字典。

```python
data = result.to_dict()
```

---

#### to_json() -> str

转换为 JSON。

```python
json_str = result.to_json()
```

---

## ⚙️ 配置类

### Config

配置类。

```python
from openhands import Config

config = Config(
    api_key="sk-ant-xxx",
    default_model="claude-3.5-sonnet",
    max_tokens=4096,
    temperature=0.7,
    timeout=300,
    max_retries=3,
    enable_cache=True,
    cache_ttl=3600,
    debug=False,
    log_level="INFO",
    log_file="~/.openhands/logs/openhands.log"
)
```

**方法**:

#### load(path: str) -> Config

从文件加载配置。

```python
config = Config.load("config.yaml")
```

---

#### save(path: str)

保存配置到文件。

```python
config.save("config.yaml")
```

---

#### get(key: str) -> Any

获取配置值。

```python
api_key = config.get("api_key")
```

---

#### set(key: str, value: Any)

设置配置值。

```python
config.set("api_key", "sk-ant-xxx")
```

---

### ModelConfig

模型配置类。

```python
from openhands import ModelConfig

model_config = ModelConfig(
    name="claude-3.5-sonnet",
    max_tokens=4096,
    temperature=0.7,
    top_p=0.9,
    frequency_penalty=0.0,
    presence_penalty=0.0
)
```

---

## 🛠️ 工具类

### CacheManager

缓存管理器。

```python
from openhands import CacheManager

cache = CacheManager(
    max_size=1000,  # 最大缓存数量
    ttl=3600        # 缓存时间（秒）
)
```

**方法**:

#### get(key: str) -> Optional[Any]

获取缓存。

```python
value = cache.get("task_hash")
```

---

#### set(key: str, value: Any)

设置缓存。

```python
cache.set("task_hash", result)
```

---

#### clear()

清除所有缓存。

```python
cache.clear()
```

---

### TaskQueue

任务队列。

```python
from openhands import TaskQueue

queue = TaskQueue(
    max_workers=4,  # 最大并发数
    timeout=300     # 超时时间（秒）
)
```

**方法**:

#### add(task: str)

添加任务。

```python
queue.add("任务1")
queue.add("任务2")
```

---

#### execute() -> List[Result]

执行所有任务。

```python
results = queue.execute()

for result in results:
    print(result.output)
```

---

### Logger

日志记录器。

```python
from openhands import Logger

logger = Logger(
    name="openhands",
    level="INFO",
    format="json",  # json/text
    file="~/.openhands/logs/openhands.log"
)
```

**方法**:

#### info(message: str, **kwargs)

记录信息日志。

```python
logger.info("Task created", task_id="123")
```

---

#### error(message: str, **kwargs)

记录错误日志。

```python
logger.error("Task failed", task_id="123", error="Timeout")
```

---

## 🚨 异常类

### OpenHandsError

基础异常类。

```python
from openhands import OpenHandsError

try:
    result = agent.run("任务")
except OpenHandsError as e:
    print(f"Error: {e}")
```

---

### TaskTimeoutError

任务超时异常。

```python
from openhands import TaskTimeoutError

try:
    result = agent.run("任务")
except TaskTimeoutError as e:
    print(f"Timeout: {e}")
```

---

### ModelNotFoundError

模型未找到异常。

```python
from openhands import ModelNotFoundError

try:
    result = agent.run("任务")
except ModelNotFoundError as e:
    print(f"Model not found: {e}")
```

---

### APIError

API 错误异常。

```python
from openhands import APIError

try:
    result = agent.run("任务")
except APIError as e:
    print(f"API error: {e}")
```

---

## 📊 完整示例

### 示例 1：基础使用

```python
from openhands import Agent

# 创建 Agent
agent = Agent(model="claude-3.5-sonnet")

# 执行任务
result = agent.run("创建 FastAPI 项目")

# 输出结果
print(result.output)
print(f"Duration: {result.duration}s")
print(f"Tokens: {result.tokens}")
print(f"Cost: ${result.cost}")
```

---

### 示例 2：异步执行

```python
import asyncio
from openhands import Agent

async def main():
    agent = Agent(model="claude-3.5-sonnet")
    
    # 异步执行
    result = await agent.run_async("创建 FastAPI 项目")
    
    print(result.output)

asyncio.run(main())
```

---

### 示例 3：批量执行

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    enable_parallel=True,
    max_workers=4
)

# 批量任务
tasks = [
    "创建项目结构",
    "实现用户认证",
    "编写单元测试"
]

# 批量执行
results = agent.batch(tasks)

for i, result in enumerate(results):
    print(f"Task {i+1}: {result.status}")
```

---

### 示例 4：流式执行

```python
import asyncio
from openhands import Agent

async def main():
    agent = Agent(model="claude-3.5-sonnet")
    
    # 流式执行
    async for chunk in agent.run_stream("创建 FastAPI 项目"):
        print(chunk, end="", flush=True)

asyncio.run(main())
```

---

### 示例 5：使用配置

```python
from openhands import Agent, Config

# 加载配置
config = Config.load("config.yaml")

# 创建 Agent
agent = Agent(
    model=config.get("default_model"),
    max_tokens=config.get("max_tokens"),
    temperature=config.get("temperature")
)

# 执行任务
result = agent.run("创建 FastAPI 项目")
print(result.output)
```

---

### 示例 6：错误处理

```python
from openhands import Agent, TaskTimeoutError, APIError

agent = Agent(
    model="claude-3.5-sonnet",
    timeout=300,
    max_retries=3
)

try:
    result = agent.run("创建 FastAPI 项目")
    print(result.output)

except TaskTimeoutError as e:
    print(f"Task timeout: {e}")
    # 重试或降级

except APIError as e:
    print(f"API error: {e}")
    # 检查 API Key 或网络

except Exception as e:
    print(f"Unexpected error: {e}")
    # 记录日志
```

---

## 📚 完整 API 列表

### 核心类
- `Agent` - 主 Agent 类
- `Result` - 执行结果类

### 配置类
- `Config` - 配置类
- `ModelConfig` - 模型配置类

### 工具类
- `CacheManager` - 缓存管理器
- `TaskQueue` - 任务队列
- `Logger` - 日志记录器

### 异常类
- `OpenHandsError` - 基础异常
- `TaskTimeoutError` - 任务超时
- `ModelNotFoundError` - 模型未找到
- `APIError` - API 错误

---

<div align="center">
  <p>📖 完整 API 参考，随时查阅！</p>
</div>
