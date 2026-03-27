# OpenHands 最佳实践合集

> 从实战中总结的最佳实践

---

## 📋 目录

- [代码质量](#代码质量)
- [性能优化](#性能优化)
- [安全实践](#安全实践)
- [团队协作](#团队协作)

---

## 💎 代码质量

### 1. 代码规范

```python
# best_practices/code_style.py
from typing import List, Dict, Optional
from dataclasses import dataclass
from enum import Enum

# ✅ 好的实践：使用类型注解
class TaskStatus(str, Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"

@dataclass
class Task:
    """任务数据类"""
    id: str
    name: str
    status: TaskStatus = TaskStatus.PENDING
    result: Optional[str] = None

class TaskManager:
    """任务管理器"""
    
    def __init__(self):
        self.tasks: Dict[str, Task] = {}
    
    def create_task(self, name: str) -> Task:
        """创建任务"""
        task = Task(
            id=self._generate_id(),
            name=name
        )
        self.tasks[task.id] = task
        return task
    
    def get_task(self, task_id: str) -> Optional[Task]:
        """获取任务"""
        return self.tasks.get(task_id)
    
    def list_tasks(self, status: Optional[TaskStatus] = None) -> List[Task]:
        """列出任务"""
        tasks = list(self.tasks.values())
        
        if status:
            tasks = [t for t in tasks if t.status == status]
        
        return tasks
    
    def _generate_id(self) -> str:
        """生成唯一 ID"""
        import uuid
        return str(uuid.uuid4())

# ❌ 不好的实践：缺少类型注解
class BadTaskManager:
    def __init__(self):
        self.tasks = {}
    
    def create_task(self, name):
        task = {"id": "123", "name": name}
        self.tasks[task["id"]] = task
        return task
```

### 2. 错误处理

```python
# best_practices/error_handling.py
from typing import Optional
from fastapi import HTTPException, status

class TaskNotFoundError(Exception):
    """任务未找到异常"""
    pass

class TaskExecutionError(Exception):
    """任务执行异常"""
    pass

class ErrorHandler:
    """错误处理器"""
    
    @staticmethod
    def handle_task_not_found(task_id: str):
        """处理任务未找到"""
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Task {task_id} not found"
        )
    
    @staticmethod
    def handle_execution_error(error: Exception):
        """处理执行错误"""
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Task execution failed: {str(error)}"
        )
    
    @staticmethod
    def safe_execute(func, *args, **kwargs):
        """安全执行函数"""
        try:
            return func(*args, **kwargs)
        except TaskNotFoundError as e:
            ErrorHandler.handle_task_not_found(str(e))
        except TaskExecutionError as e:
            ErrorHandler.handle_execution_error(e)
        except Exception as e:
            # 记录未知错误
            import logging
            logging.error(f"Unexpected error: {e}")
            raise

# 使用示例
manager = TaskManager()
task = manager.create_task("Example Task")

# 安全获取
try:
    task = ErrorHandler.safe_execute(
        manager.get_task,
        "task-123"
    )
except HTTPException as e:
    print(f"Error: {e.detail}")
```

---

## ⚡ 性能优化

### 1. 异步处理

```python
# best_practices/async_processing.py
import asyncio
from typing import List
from openhands import Agent

class AsyncTaskProcessor:
    """异步任务处理器"""
    
    def __init__(self, max_workers: int = 4):
        self.agent = Agent(model="claude-3.5-sonnet")
        self.max_workers = max_workers
        self.semaphore = asyncio.Semaphore(max_workers)
    
    async def process_task(self, task: str):
        """处理单个任务"""
        async with self.semaphore:
            result = await self.agent.run_async(task)
            return result
    
    async def process_batch(self, tasks: List[str]):
        """批量处理任务"""
        # 并行执行
        results = await asyncio.gather(*[
            self.process_task(task) for task in tasks
        ])
        
        return results
    
    async def process_stream(self, tasks: List[str]):
        """流式处理任务"""
        for task in tasks:
            result = await self.process_task(task)
            yield result

# 使用示例
async def main():
    processor = AsyncTaskProcessor(max_workers=4)
    
    # 批量处理
    tasks = ["任务1", "任务2", "任务3", "任务4"]
    results = await processor.process_batch(tasks)
    
    for result in results:
        print(result)

# 运行
asyncio.run(main())
```

### 2. 缓存策略

```python
# best_practices/caching.py
from functools import lru_cache
from typing import Dict, Optional
import hashlib
import json

class CacheManager:
    """缓存管理器"""
    
    def __init__(self, max_size: int = 1000):
        self.cache: Dict[str, any] = {}
        self.max_size = max_size
    
    def get(self, key: str) -> Optional[any]:
        """获取缓存"""
        return self.cache.get(key)
    
    def set(self, key: str, value: any):
        """设置缓存"""
        if len(self.cache) >= self.max_size:
            # LRU 淘汰
            oldest_key = next(iter(self.cache))
            del self.cache[oldest_key]
        
        self.cache[key] = value
    
    def get_task_hash(self, task: str) -> str:
        """计算任务哈希"""
        return hashlib.md5(task.encode()).hexdigest()
    
    @lru_cache(maxsize=1000)
    def cached_computation(self, x: int, y: int) -> int:
        """缓存的计算"""
        # 模拟耗时计算
        import time
        time.sleep(1)
        return x + y

# 使用示例
cache = CacheManager()

# 缓存任务结果
task = "创建 FastAPI 项目"
task_hash = cache.get_task_hash(task)

# 检查缓存
cached_result = cache.get(task_hash)
if cached_result:
    print("从缓存读取")
else:
    # 执行任务
    result = execute_task(task)
    cache.set(task_hash, result)
```

---

## 🔒 安全实践

### 1. 输入验证

```python
# best_practices/input_validation.py
from pydantic import BaseModel, Field, validator
from typing import List
import re

class TaskInput(BaseModel):
    """任务输入验证"""
    name: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    priority: int = Field(1, ge=1, le=5)
    tags: List[str] = Field(default_factory=list)
    
    @validator('name')
    def validate_name(cls, v):
        """验证名称"""
        # 只允许字母、数字、空格、下划线
        if not re.match(r'^[\w\s]+$', v):
            raise ValueError('Name contains invalid characters')
        return v
    
    @validator('tags')
    def validate_tags(cls, v):
        """验证标签"""
        # 最多 5 个标签
        if len(v) > 5:
            raise ValueError('Maximum 5 tags allowed')
        
        # 每个标签最多 20 字符
        for tag in v:
            if len(tag) > 20:
                raise ValueError('Tag too long (max 20 characters)')
        
        return v

class UserInput(BaseModel):
    """用户输入验证"""
    email: str = Field(..., regex=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    password: str = Field(..., min_length=8)
    
    @validator('password')
    def validate_password(cls, v):
        """验证密码"""
        # 至少一个大写字母
        if not re.search(r'[A-Z]', v):
            raise ValueError('Password must contain at least one uppercase letter')
        
        # 至少一个数字
        if not re.search(r'\d', v):
            raise ValueError('Password must contain at least one digit')
        
        # 至少一个特殊字符
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', v):
            raise ValueError('Password must contain at least one special character')
        
        return v

# 使用示例
try:
    task = TaskInput(
        name="Example Task",
        description="This is an example",
        priority=3,
        tags=["python", "fastapi"]
    )
    print("✅ Validation passed")
except ValueError as e:
    print(f"❌ Validation failed: {e}")
```

### 2. SQL 注入防护

```python
# best_practices/sql_injection_protection.py
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker

engine = create_engine('postgresql://user:pass@localhost/db')
Session = sessionmaker(bind=engine)

class SafeDatabase:
    """安全的数据库操作"""
    
    def __init__(self):
        self.session = Session()
    
    def get_user_by_id(self, user_id: str):
        """安全查询（参数化查询）"""
        # ✅ 好的实践：使用参数化查询
        query = text("SELECT * FROM users WHERE id = :user_id")
        result = self.session.execute(query, {"user_id": user_id})
        return result.fetchone()
    
    def search_users(self, name: str):
        """安全搜索"""
        # ✅ 好的实践：使用 LIKE 时转义
        name = name.replace('%', r'\%').replace('_', r'\_')
        query = text("SELECT * FROM users WHERE name LIKE :name")
        result = self.session.execute(query, {"name": f"%{name}%"})
        return result.fetchall()
    
    def bad_example(self, user_input: str):
        """❌ 不好的实践：直接拼接 SQL"""
        # 危险！容易导致 SQL 注入
        query = f"SELECT * FROM users WHERE name = '{user_input}'"
        # 永远不要这样做！
        pass

# 使用示例
db = SafeDatabase()

# 安全查询
user = db.get_user_by_id("123")

# 安全搜索
users = db.search_users("Alice")
```

---

## 👥 团队协作

### 1. Git 工作流

```bash
# best_practices/git_workflow.sh

# 1. 功能分支工作流
# 创建功能分支
git checkout -b feature/new-feature

# 2. 提交代码
git add .
git commit -m "feat: 添加新功能"

# 3. 推送到远程
git push origin feature/new-feature

# 4. 创建 Pull Request
# 在 GitHub 上创建 PR

# 5. 代码审查
# 团队成员审查代码

# 6. 合并到主分支
# 审查通过后合并

# 7. 删除功能分支
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

### 2. 代码审查检查清单

```markdown
# best_practices/code_review_checklist.md

## ✅ 代码审查检查清单

### 功能性
- [ ] 代码实现了需求
- [ ] 所有测试通过
- [ ] 没有明显的 Bug

### 代码质量
- [ ] 代码可读性强
- [ ] 命名清晰
- [ ] 注释充分
- [ ] 遵循代码规范

### 性能
- [ ] 没有明显的性能问题
- [ ] 数据库查询优化
- [ ] 使用了缓存

### 安全
- [ ] 输入验证
- [ ] SQL 注入防护
- [ ] XSS 防护
- [ ] 敏感数据加密

### 测试
- [ ] 单元测试覆盖
- [ ] 集成测试覆盖
- [ ] 测试覆盖率 > 80%

### 文档
- [ ] API 文档更新
- [ ] README 更新
- [ ] 注释清晰
```

### 3. 文档规范

```markdown
# best_practices/documentation_template.md

# 功能名称

## 简介
简要描述功能的作用和目的。

## 快速开始

### 前置要求
- 列出依赖和环境要求

### 安装
```bash
# 安装命令
```

### 使用示例
```python
# 代码示例
```

## 详细说明

### 配置
说明可配置的选项。

### API 参考
列出主要的 API 接口。

### 最佳实践
提供使用建议。

### 常见问题
列出常见问题和解决方案。

## 贡献指南
说明如何贡献代码。

## 许可证
声明许可证类型。
```

---

## 📊 监控和日志

### 1. 结构化日志

```python
# best_practices/logging.py
import logging
import json
from datetime import datetime

class StructuredLogger:
    """结构化日志"""
    
    def __init__(self, name: str):
        self.logger = logging.getLogger(name)
        self.logger.setLevel(logging.INFO)
        
        # JSON 格式化器
        handler = logging.StreamHandler()
        handler.setFormatter(self.JSONFormatter())
        self.logger.addHandler(handler)
    
    class JSONFormatter(logging.Formatter):
        """JSON 格式化器"""
        def format(self, record):
            log_entry = {
                "timestamp": datetime.utcnow().isoformat(),
                "level": record.levelname,
                "logger": record.name,
                "message": record.getMessage(),
                "module": record.module,
                "function": record.funcName,
                "line": record.lineno
            }
            
            if record.exc_info:
                log_entry["exception"] = self.formatException(record.exc_info)
            
            return json.dumps(log_entry)
    
    def info(self, message: str, **kwargs):
        """信息日志"""
        self.logger.info(message, extra=kwargs)
    
    def error(self, message: str, **kwargs):
        """错误日志"""
        self.logger.error(message, extra=kwargs)

# 使用示例
logger = StructuredLogger("openhands")

logger.info("Task created", task_id="123", user_id="456")
logger.error("Task failed", task_id="123", error="Timeout")
```

### 2. 性能监控

```python
# best_practices/monitoring.py
import time
from functools import wraps
from prometheus_client import Counter, Histogram

# 定义指标
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint']
)

def monitor_performance(func):
    """性能监控装饰器"""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start_time = time.time()
        
        try:
            result = await func(*args, **kwargs)
            
            # 记录成功请求
            REQUEST_COUNT.labels(
                method="POST",
                endpoint=func.__name__,
                status=200
            ).inc()
            
            return result
        
        except Exception as e:
            # 记录失败请求
            REQUEST_COUNT.labels(
                method="POST",
                endpoint=func.__name__,
                status=500
            ).inc()
            
            raise
        
        finally:
            # 记录延迟
            duration = time.time() - start_time
            REQUEST_LATENCY.labels(
                method="POST",
                endpoint=func.__name__
            ).observe(duration)
    
    return wrapper

# 使用示例
@monitor_performance
async def create_task(task: str):
    """创建任务（带监控）"""
    # 执行任务
    result = await execute_task(task)
    return result
```

---

## 🎯 最佳实践检查清单

### ✅ 代码质量
- [ ] 使用类型注解
- [ ] 遵循代码规范
- [ ] 编写单元测试
- [ ] 代码审查

### ✅ 性能
- [ ] 异步处理
- [ ] 使用缓存
- [ ] 数据库优化
- [ ] 性能监控

### ✅ 安全
- [ ] 输入验证
- [ ] SQL 注入防护
- [ ] XSS 防护
- [ ] 数据加密

### ✅ 团队协作
- [ ] Git 工作流
- [ ] 代码审查
- [ ] 文档规范
- [ ] 沟通协作

---

<div align="center">
  <p>💎 最佳实践，持续改进！</p>
</div>
