# OpenHands 微服务架构案例

> 基于 DDD 和 Event Sourcing 的微服务设计

---

## 📋 目录

- [架构设计](#架构设计)
- [服务拆分](#服务拆分)
- [通信机制](#通信机制)
- [数据一致性](#数据一致性)

---

## 🏗️ 架构设计

### 整体架构

```
┌─────────────────────────────────────────┐
│           API Gateway (Kong)            │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│ Agent  │  │ Task   │  │ Workflow│
│Service │  │Service │  │ Service │
└───┬────┘  └───┬────┘  └───┬────┘
    │           │            │
    └───────────┼────────────┘
                │
        ┌───────▼────────┐
        │ Message Queue  │
        │ (RabbitMQ)     │
        └───────┬────────┘
                │
    ┌───────────┼───────────┐
    │           │           │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│ Event  │  │ Logger │  │ Notifier│
│ Store  │  │Service │  │ Service │
└────────┘  └────────┘  └─────────┘
```

---

## 🔧 服务拆分

### 1. Agent Service

**职责**: 管理 Agent 生命周期

```python
# agent_service/main.py
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional
import httpx

app = FastAPI()

class AgentConfig(BaseModel):
    name: str
    model: str
    capabilities: list[str]
    max_workers: int = 4

class Agent:
    def __init__(self, config: AgentConfig):
        self.config = config
        self.status = "idle"
        self.current_task = None

    async def execute(self, task: str):
        """执行任务"""
        self.status = "running"
        self.current_task = task
        
        # 调用 Task Service
        async with httpx.AsyncClient() as client:
            response = await client.post(
                "http://task-service/api/v1/tasks",
                json={"task": task, "agent": self.config.name}
            )
        
        self.status = "idle"
        self.current_task = None
        return response.json()

# REST API
@app.post("/api/v1/agents")
async def create_agent(config: AgentConfig):
    """创建 Agent"""
    agent = Agent(config)
    # 保存到数据库
    return {"id": "agent-123", "status": "created"}

@app.get("/api/v1/agents/{agent_id}")
async def get_agent(agent_id: str):
    """获取 Agent 状态"""
    return {
        "id": agent_id,
        "status": "idle",
        "current_task": None
    }

@app.post("/api/v1/agents/{agent_id}/execute")
async def execute_task(agent_id: str, task: str):
    """执行任务"""
    # 获取 Agent
    agent = await get_agent_from_db(agent_id)
    result = await agent.execute(task)
    return result
```

---

### 2. Task Service

**职责**: 任务调度和执行

```python
# task_service/main.py
from fastapi import FastAPI
from pydantic import BaseModel
from enum import Enum
import asyncio

app = FastAPI()

class TaskStatus(str, Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"

class Task(BaseModel):
    id: str
    task: str
    agent: str
    status: TaskStatus = TaskStatus.PENDING
    result: Optional[str] = None

# 任务队列
task_queue = asyncio.Queue()

@app.post("/api/v1/tasks")
async def create_task(task: str, agent: str):
    """创建任务"""
    task_obj = Task(
        id=f"task-{uuid.uuid4()}",
        task=task,
        agent=agent
    )
    
    # 添加到队列
    await task_queue.put(task_obj)
    
    # 发布事件
    await publish_event("task.created", task_obj.dict())
    
    return task_obj

@app.get("/api/v1/tasks/{task_id}")
async def get_task(task_id: str):
    """获取任务状态"""
    return await get_task_from_db(task_id)

@app.get("/api/v1/tasks")
async def list_tasks(status: Optional[TaskStatus] = None):
    """列出任务"""
    tasks = await list_tasks_from_db(status)
    return tasks

# 后台任务处理器
async def task_worker():
    """后台任务处理"""
    while True:
        task = await task_queue.get()
        
        try:
            task.status = TaskStatus.RUNNING
            await update_task_in_db(task)
            
            # 执行任务
            result = await execute_task(task)
            
            task.status = TaskStatus.COMPLETED
            task.result = result
            await update_task_in_db(task)
            
            # 发布完成事件
            await publish_event("task.completed", task.dict())
            
        except Exception as e:
            task.status = TaskStatus.FAILED
            task.result = str(e)
            await update_task_in_db(task)
            
            # 发布失败事件
            await publish_event("task.failed", task.dict())

@app.on_event("startup")
async def startup():
    """启动后台任务处理器"""
    asyncio.create_task(task_worker())
```

---

### 3. Workflow Service

**职责**: 工作流编排

```python
# workflow_service/main.py
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List, Dict
import asyncio

app = FastAPI()

class WorkflowStep(BaseModel):
    name: str
    task: str
    agent: str
    depends_on: List[str] = []

class Workflow(BaseModel):
    id: str
    name: str
    steps: List[WorkflowStep]
    status: str = "pending"

# 工作流引擎
class WorkflowEngine:
    def __init__(self):
        self.workflows = {}
    
    async def execute(self, workflow: Workflow):
        """执行工作流"""
        workflow.status = "running"
        
        # 构建依赖图
        graph = self.build_dependency_graph(workflow.steps)
        
        # 拓扑排序
        sorted_steps = self.topological_sort(graph)
        
        # 执行步骤
        results = {}
        for step in sorted_steps:
            # 等待依赖完成
            for dep in step.depends_on:
                await self.wait_for_completion(dep, results)
            
            # 执行步骤
            result = await self.execute_step(step)
            results[step.name] = result
        
        workflow.status = "completed"
        return results
    
    def build_dependency_graph(self, steps: List[WorkflowStep]):
        """构建依赖图"""
        graph = {step.name: step for step in steps}
        return graph
    
    def topological_sort(self, graph: Dict):
        """拓扑排序"""
        # 实现拓扑排序算法
        visited = set()
        result = []
        
        def visit(name):
            if name in visited:
                return
            visited.add(name)
            step = graph[name]
            for dep in step.depends_on:
                visit(dep)
            result.append(step)
        
        for name in graph:
            visit(name)
        
        return result
    
    async def execute_step(self, step: WorkflowStep):
        """执行单个步骤"""
        async with httpx.AsyncClient() as client:
            response = await client.post(
                "http://task-service/api/v1/tasks",
                json={"task": step.task, "agent": step.agent}
            )
            return response.json()
    
    async def wait_for_completion(self, step_name: str, results: Dict):
        """等待步骤完成"""
        while step_name not in results:
            await asyncio.sleep(0.1)

@app.post("/api/v1/workflows")
async def create_workflow(workflow: Workflow):
    """创建工作流"""
    engine = WorkflowEngine()
    results = await engine.execute(workflow)
    return {"workflow_id": workflow.id, "results": results}
```

---

## 🔗 通信机制

### 1. 同步通信（REST）

```python
# 使用 httpx 进行服务间调用
import httpx

async def call_task_service(task: str):
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "http://task-service/api/v1/tasks",
            json={"task": task}
        )
        return response.json()
```

### 2. 异步通信（Message Queue）

```python
# 使用 RabbitMQ 进行异步通信
import aio_pika

async def publish_event(event_type: str, data: dict):
    """发布事件"""
    connection = await aio_pika.connect_robust(
        "amqp://guest:guest@rabbitmq/"
    )
    
    async with connection:
        channel = await connection.channel()
        
        await channel.default_exchange.publish(
            aio_pika.Message(
                body=json.dumps({
                    "type": event_type,
                    "data": data
                }).encode()
            ),
            routing_key="events"
        )

async def consume_events():
    """消费事件"""
    connection = await aio_pika.connect_robust(
        "amqp://guest:guest@rabbitmq/"
    )
    
    async with connection:
        channel = await connection.channel()
        queue = await channel.declare_queue("events")
        
        async with queue.iterator() as queue_iter:
            async for message in queue_iter:
                async with message.process():
                    event = json.loads(message.body)
                    await handle_event(event)
```

---

## 💾 数据一致性

### 1. Event Sourcing

```python
# event_store.py
from datetime import datetime
from typing import List, Dict
import json

class Event:
    def __init__(self, event_type: str, data: dict):
        self.id = str(uuid.uuid4())
        self.type = event_type
        self.data = data
        self.timestamp = datetime.utcnow()
    
    def to_dict(self):
        return {
            "id": self.id,
            "type": self.type,
            "data": self.data,
            "timestamp": self.timestamp.isoformat()
        }

class EventStore:
    def __init__(self):
        self.events = []
    
    async def append(self, event: Event):
        """追加事件"""
        self.events.append(event)
        # 持久化到数据库
        await self.save_to_db(event)
    
    async def get_events(self, aggregate_id: str) -> List[Event]:
        """获取聚合的事件"""
        return [e for e in self.events if e.data.get("aggregate_id") == aggregate_id]
    
    async def save_to_db(self, event: Event):
        """保存到数据库"""
        # 实现数据库持久化
        pass

# 使用示例
event_store = EventStore()

async def create_task(task_data: dict):
    """创建任务（使用 Event Sourcing）"""
    event = Event("task.created", {
        "aggregate_id": task_data["id"],
        "data": task_data
    })
    await event_store.append(event)
```

### 2. CQRS

```python
# cqrs.py
from typing import Protocol

class Command(Protocol):
    """命令接口"""
    async def execute(self) -> dict:
        ...

class Query(Protocol):
    """查询接口"""
    async def execute(self) -> dict:
        ...

# 命令处理器
class CreateTaskCommand:
    def __init__(self, task_data: dict):
        self.task_data = task_data
    
    async def execute(self) -> dict:
        """执行命令（写操作）"""
        # 写入数据库
        task = await write_to_db(self.task_data)
        
        # 发布事件
        await publish_event("task.created", task)
        
        return task

# 查询处理器
class GetTaskQuery:
    def __init__(self, task_id: str):
        self.task_id = task_id
    
    async def execute(self) -> dict:
        """执行查询（读操作）"""
        # 从读库查询
        task = await read_from_db(self.task_id)
        return task

# CQRS 总线
class CQRSBus:
    def __init__(self):
        self.commands = {}
        self.queries = {}
    
    def register_command(self, name: str, handler: Command):
        self.commands[name] = handler
    
    def register_query(self, name: str, handler: Query):
        self.queries[name] = handler
    
    async def execute_command(self, name: str, *args, **kwargs):
        return await self.commands[name](*args, **kwargs).execute()
    
    async def execute_query(self, name: str, *args, **kwargs):
        return await self.queries[name](*args, **kwargs).execute()
```

---

## 🎯 最佳实践

### 1. 服务拆分原则
- 单一职责
- 高内聚低耦合
- 独立部署
- 故障隔离

### 2. 通信策略
- 同步：REST/gRPC（简单查询）
- 异步：Message Queue（复杂流程）
- 事件：Event Bus（状态变更）

### 3. 数据一致性
- Event Sourcing（事件溯源）
- CQRS（命令查询分离）
- Saga（分布式事务）

### 4. 容错设计
- 熔断器（Circuit Breaker）
- 重试机制（Retry）
- 超时控制（Timeout）
- 降级策略（Fallback）

---

<div align="center">
  <p>🏗️ 微服务架构，可扩展设计！</p>
</div>
