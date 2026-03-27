# OpenHands 进阶技巧 50+

> 从新手到专家的进阶技巧

---

## 📋 目录

- [效率提升技巧](#效率提升技巧)
- [代码质量技巧](#代码质量技巧)
- [性能优化技巧](#性能优化技巧)
- [故障排查技巧](#故障排查技巧)

---

## 🚀 效率提升技巧

### 技巧 1：使用别名

```bash
# ~/.bashrc 或 ~/.zshrc
alias oh='openhands'
alias ohr='openhands run'
alias ohc='openhands config'
alias ohm='openhands models'

# 使用
ohr "创建项目"  # 等同于 openhands run "创建项目"
```

---

### 技巧 2：批量任务

```bash
# 创建任务文件
cat > tasks.txt << EOF
创建项目结构
实现用户认证
编写单元测试
生成文档
EOF

# 批量执行
while read task; do
  openhands run "$task"
done < tasks.txt
```

---

### 技巧 3：环境变量管理

```bash
# 使用 direnv
# .envrc
export ANTHROPIC_API_KEY="sk-ant-xxx"
export OPENHANDS_MODEL="claude-3.5-sonnet"
export OPENHANDS_TIMEOUT="300"

# 自动加载
direnv allow
```

---

### 技巧 4：模板复用

```yaml
# templates/api.yaml
name: API 项目模板
prompt: |
  创建 {framework} API 项目：
  
  1. 框架：{framework}
  2. 数据库：{database}
  3. 认证：{auth}
  4. 文档：OpenAPI
  5. 测试：pytest

# 使用
openhands run --template templates/api.yaml \
  --param framework=FastAPI \
  --param database=PostgreSQL \
  --param auth=JWT
```

---

### 技巧 5：任务队列

```python
from openhands import Agent
import asyncio

async def task_queue(tasks, max_workers=4):
    """任务队列"""
    agent = Agent(
        model="claude-3.5-sonnet",
        enable_parallel=True,
        max_workers=max_workers
    )
    
    results = await asyncio.gather(*[
        agent.run_async(task) for task in tasks
    ])
    
    return results

# 使用
tasks = ["任务1", "任务2", "任务3", "任务4"]
results = asyncio.run(task_queue(tasks))
```

---

## 💎 代码质量技巧

### 技巧 6：自动代码审查

```python
from openhands import Agent

def code_review(file_path):
    """代码审查"""
    agent = Agent(model="claude-3.5-sonnet")
    
    with open(file_path, 'r') as f:
        code = f.read()
    
    result = agent.run(f"""
    审查以下代码，提出改进建议：
    
    {code}
    
    检查项目：
    1. 代码风格
    2. 性能问题
    3. 安全隐患
    4. 最佳实践
    """)
    
    return result.output

# 使用
review = code_review("main.py")
print(review)
```

---

### 技巧 7：自动生成文档

```python
from openhands import Agent

def generate_docs(code_path):
    """生成文档"""
    agent = Agent(model="claude-3.5-sonnet")
    
    with open(code_path, 'r') as f:
        code = f.read()
    
    result = agent.run(f"""
    为以下代码生成文档：
    
    {code}
    
    包含：
    1. 模块说明
    2. 函数文档
    3. 参数说明
    4. 使用示例
    """)
    
    # 保存文档
    doc_path = code_path.replace('.py', '_docs.md')
    with open(doc_path, 'w') as f:
        f.write(result.output)
    
    return doc_path

# 使用
doc_file = generate_docs("main.py")
```

---

### 技巧 8：自动重构

```python
from openhands import Agent

def refactor_code(code_path, refactoring_type):
    """代码重构"""
    agent = Agent(model="claude-3.5-sonnet")
    
    with open(code_path, 'r') as f:
        code = f.read()
    
    result = agent.run(f"""
    重构以下代码（{refactoring_type}）：
    
    {code}
    
    重构类型：{refactoring_type}
    
    要求：
    1. 保持功能不变
    2. 提高代码质量
    3. 遵循最佳实践
    """)
    
    # 保存重构后的代码
    with open(code_path, 'w') as f:
        f.write(result.output)
    
    return code_path

# 使用
refactor_code("main.py", "extract_method")
```

---

## ⚡ 性能优化技巧

### 技巧 9：缓存策略

```python
from openhands import Agent
import hashlib

class CachedAgent:
    def __init__(self):
        self.agent = Agent(
            model="claude-3.5-sonnet",
            enable_cache=True,
            cache_ttl=3600
        )
        self.cache = {}
    
    def run(self, task):
        """带缓存的任务执行"""
        task_hash = hashlib.md5(task.encode()).hexdigest()
        
        if task_hash in self.cache:
            print("✅ Cache hit")
            return self.cache[task_hash]
        
        result = self.agent.run(task)
        self.cache[task_hash] = result
        
        return result

# 使用
agent = CachedAgent()
result1 = agent.run("创建项目")  # 实际执行
result2 = agent.run("创建项目")  # 从缓存读取
```

---

### 技巧 10：Token 优化

```python
from openhands import Agent

def optimize_prompt(prompt):
    """优化 Prompt"""
    # 1. 移除冗余空格
    prompt = ' '.join(prompt.split())
    
    # 2. 压缩重复
    # 3. 简化表达
    
    return prompt

# 使用
long_prompt = """
请创建一个 FastAPI 项目，
包含用户认证和数据库连接。
"""

short_prompt = optimize_prompt(long_prompt)

agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=2048  # 限制 token
)

result = agent.run(short_prompt)
```

---

### 技巧 11：并行处理

```python
from openhands import Agent
import asyncio

async def parallel_execution(tasks):
    """并行执行"""
    agent = Agent(
        model="claude-3.5-sonnet",
        enable_parallel=True,
        max_workers=4
    )
    
    results = await asyncio.gather(*[
        agent.run_async(task) for task in tasks
    ], return_exceptions=True)
    
    return results

# 使用
tasks = ["任务1", "任务2", "任务3", "任务4"]
results = asyncio.run(parallel_execution(tasks))
```

---

### 技巧 12：模型选择

```python
from openhands import Agent
from enum import Enum

class TaskComplexity(Enum):
    SIMPLE = "simple"
    MEDIUM = "medium"
    COMPLEX = "complex"

def select_model(complexity):
    """智能模型选择"""
    models = {
        TaskComplexity.SIMPLE: "gpt-3.5-turbo",
        TaskComplexity.MEDIUM: "gpt-4o-mini",
        TaskComplexity.COMPLEX: "claude-3.5-sonnet"
    }
    
    return models[complexity]

# 使用
complexity = TaskComplexity.SIMPLE
model = select_model(complexity)
agent = Agent(model=model)
```

---

## 🔧 故障排查技巧

### 技巧 13：启用调试

```python
from openhands import Agent
import logging

# 启用调试日志
logging.basicConfig(level=logging.DEBUG)

agent = Agent(
    model="claude-3.5-sonnet",
    debug=True
)

result = agent.run("创建项目")
```

---

### 技巧 14：错误重试

```python
from openhands import Agent
import time

def retry_task(task, max_retries=3):
    """错误重试"""
    agent = Agent(
        model="claude-3.5-sonnet",
        max_retries=max_retries
    )
    
    for i in range(max_retries):
        try:
            result = agent.run(task)
            return result
        except Exception as e:
            print(f"Attempt {i+1} failed: {e}")
            time.sleep(2 ** i)  # 指数退避
    
    raise Exception("All retries failed")

# 使用
result = retry_task("创建项目", max_retries=3)
```

---

### 技巧 15：性能监控

```python
from openhands import Agent
import time

class MonitoredAgent:
    def __init__(self):
        self.agent = Agent(model="claude-3.5-sonnet")
    
    def run(self, task):
        """带监控的任务执行"""
        start_time = time.time()
        
        result = self.agent.run(task)
        
        duration = time.time() - start_time
        
        print(f"Duration: {duration:.2f}s")
        print(f"Tokens: {result.tokens}")
        print(f"Cost: ${result.cost:.4f}")
        
        return result

# 使用
agent = MonitoredAgent()
result = agent.run("创建项目")
```

---

## 📊 更多技巧（50+）

### 效率提升（16-25）

16. 使用 Git Hooks 自动化
17. 配置文件分离
18. 使用 Makefile
19. Docker 集成
20. CI/CD 集成
21. 环境变量管理
22. 任务优先级
23. 依赖管理
24. 日志管理
25. 监控告警

### 代码质量（26-35）

26. 自动格式化
27. 静态分析
28. 单元测试
29. 集成测试
30. E2E 测试
31. 代码覆盖率
32. 依赖检查
33. 安全扫描
34. 性能测试
35. 代码审查

### 性能优化（36-45）

36. 缓存策略
37. 并行处理
38. 异步执行
39. Token 优化
40. 模型选择
41. 数据库优化
42. API 优化
43. 内存管理
44. CPU 优化
45. 网络优化

### 故障排查（46-55）

46. 日志分析
47. 性能分析
48. 错误追踪
49. 监控告警
50. 自动恢复
51. 回滚机制
52. 备份策略
53. 容灾设计
54. 压力测试
55. 混沌工程

---

<div align="center">
  <p>💎 进阶技巧，持续提升！</p>
</div>
