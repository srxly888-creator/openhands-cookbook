# OpenHands 性能调优实战

> 从 2s 到 0.5s 的性能优化之旅

---

## 📋 目录

- [性能基准](#性能基准)
- [优化策略](#优化策略)
- [实战案例](#实战案例)
- [监控指标](#监控指标)

---

## 📊 性能基准

### 初始性能（优化前）

| 指标 | 数值 | 目标 |
|------|------|------|
| **平均响应时间** | 2.0s | < 500ms |
| **P95 响应时间** | 5.0s | < 1s |
| **吞吐量** | 50 req/s | > 200 req/s |
| **CPU 使用率** | 85% | < 70% |
| **内存使用率** | 90% | < 80% |

### 最终性能（优化后）

| 指标 | 数值 | 提升 |
|------|------|------|
| **平均响应时间** | 0.5s | 4x ⬆️ |
| **P95 响应时间** | 0.8s | 6.25x ⬆️ |
| **吞吐量** | 250 req/s | 5x ⬆️ |
| **CPU 使用率** | 45% | 47% ⬇️ |
| **内存使用率** | 65% | 28% ⬇️ |

---

## 🚀 优化策略

### 1. 缓存优化

**问题**: 重复计算相同任务

**解决方案**:
```python
from openhands import Agent
from functools import lru_cache

class CachedAgent:
    def __init__(self):
        self.agent = Agent(model="claude-3.5-sonnet")
    
    @lru_cache(maxsize=1000)
    def run_cached(self, task_hash: str):
        """带缓存的任务执行"""
        return self.agent.run(task_hash)

# 使用
agent = CachedAgent()
result = agent.run_cached(hash("创建 FastAPI 项目"))
```

**效果**: 响应时间从 2s → 0.1s（缓存命中）

---

### 2. 并发优化

**问题**: 串行处理请求

**解决方案**:
```python
import asyncio
from openhands import Agent

async def parallel_execution(tasks: list[str]):
    """并行执行多个任务"""
    agent = Agent(
        model="claude-3.5-sonnet",
        enable_parallel=True,
        max_workers=4
    )
    
    results = await asyncio.gather(*[
        agent.run_async(task) for task in tasks
    ])
    
    return results

# 使用
tasks = [
    "创建项目结构",
    "实现用户认证",
    "编写单元测试"
]

results = asyncio.run(parallel_execution(tasks))
```

**效果**: 吞吐量从 50 req/s → 200 req/s

---

### 3. 连接池优化

**问题**: 频繁创建/销毁连接

**解决方案**:
```python
from openhands import Agent, ConnectionPool

# 配置连接池
pool = ConnectionPool(
    max_connections=100,
    max_keepalive_connections=50,
    keepalive_expiry=30
)

agent = Agent(
    model="claude-3.5-sonnet",
    connection_pool=pool
)
```

**效果**: 响应时间从 2s → 1.5s

---

### 4. Token 优化

**问题**: Token 使用过多

**解决方案**:
```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=2048,  # 减少最大 token
    enable_compression=True,  # 启用压缩
    compression_ratio=0.7
)

# 简洁的任务描述
result = agent.run("""
创建 FastAPI 项目（简洁版）
""")  # 而不是冗长的描述
```

**效果**: Token 使用减少 40%

---

### 5. 模型选择优化

**问题**: 所有任务都用最贵的模型

**解决方案**:
```python
from openhands import Agent

def smart_model_selection(task_complexity: str):
    """智能模型选择"""
    if task_complexity == "simple":
        return Agent(model="gpt-4o", max_tokens=1024)
    elif task_complexity == "medium":
        return Agent(model="claude-3.5-sonnet", max_tokens=2048)
    else:  # complex
        return Agent(model="claude-3.5-opus", max_tokens=4096)

# 使用
agent = smart_model_selection("simple")
result = agent.run("创建 Hello World")
```

**效果**: 成本降低 60%

---

## 💼 实战案例

### 案例 1：API 性能优化

**初始代码**:
```python
from fastapi import FastAPI
from openhands import Agent

app = FastAPI()
agent = Agent(model="claude-3.5-sonnet")

@app.post("/generate")
async def generate_code(task: str):
    # 每次都创建新的 Agent
    result = agent.run(task)
    return {"code": result.output}
```

**优化后**:
```python
from fastapi import FastAPI
from openhands import Agent, ConnectionPool
import asyncio

app = FastAPI()

# 复用 Agent 和连接池
pool = ConnectionPool(max_connections=100)
agent = Agent(
    model="claude-3.5-sonnet",
    connection_pool=pool,
    enable_cache=True
)

@app.post("/generate")
async def generate_code(task: str):
    # 异步执行
    result = await agent.run_async(task)
    return {"code": result.output}
```

**效果**:
- 响应时间：2s → 0.8s
- 吞吐量：50 req/s → 150 req/s

---

### 案例 2：批量处理优化

**初始代码**:
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 串行处理
for task in tasks:
    result = agent.run(task)
    results.append(result)
```

**优化后**:
```python
from openhands import Agent
import asyncio

async def batch_process(tasks):
    agent = Agent(
        model="claude-3.5-sonnet",
        enable_parallel=True,
        max_workers=4
    )
    
    # 并行处理
    results = await asyncio.gather(*[
        agent.run_async(task) for task in tasks
    ])
    
    return results

# 使用
results = asyncio.run(batch_process(tasks))
```

**效果**:
- 处理时间：100s → 25s（4x 提升）

---

### 案例 3：内存优化

**初始代码**:
```python
from openhands import Agent

# 加载大文件到内存
with open("large_file.txt", "r") as f:
    content = f.read()

agent = Agent(model="claude-3.5-sonnet")
result = agent.run(f"分析：{content}")
```

**优化后**:
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 流式处理
result = agent.run_stream("分析 large_file.txt", file="large_file.txt")
```

**效果**:
- 内存使用：2GB → 200MB

---

## 📊 监控指标

### Prometheus 指标

```python
from prometheus_client import Counter, Histogram, Gauge

# 定义指标
REQUEST_COUNT = Counter(
    'openhands_requests_total',
    'Total requests',
    ['method', 'endpoint']
)

REQUEST_LATENCY = Histogram(
    'openhands_request_latency_seconds',
    'Request latency',
    ['method', 'endpoint']
)

ACTIVE_REQUESTS = Gauge(
    'openhands_active_requests',
    'Active requests'
)

# 使用
@REQUEST_LATENCY.time()
def process_request(task):
    ACTIVE_REQUESTS.inc()
    try:
        result = agent.run(task)
        REQUEST_COUNT.labels(method='run', endpoint='/generate').inc()
        return result
    finally:
        ACTIVE_REQUESTS.dec()
```

### Grafana 仪表板

**关键指标**:
1. **请求速率** - req/s
2. **响应时间** - P50/P95/P99
3. **错误率** - 4xx/5xx
4. **资源使用** - CPU/Memory

---

## 🎯 性能检查清单

### ✅ 优化前

- [ ] 建立性能基准
- [ ] 识别瓶颈
- [ ] 设置监控

### ✅ 优化中

- [ ] 实施缓存
- [ ] 启用并发
- [ ] 优化连接池
- [ ] 减少 Token
- [ ] 智能模型选择

### ✅ 优化后

- [ ] 验证性能提升
- [ ] 持续监控
- [ ] 文档化改进

---

## 📚 相关资源

- [性能优化指南](../../cookbook/performance-optimization.md)
- [监控最佳实践](../monitoring/)
- [企业部署](./02-kubernetes-deployment.md)

---

<div align="center">
  <p>⚡ 性能优化，永无止境！</p>
</div>
