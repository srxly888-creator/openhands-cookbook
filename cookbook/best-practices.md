# OpenHands 最佳实践

> 经过验证的使用模式和最佳实践

---

## 📋 目录

- [项目组织](#项目组织)
- [代码质量](#代码质量)
- [团队协作](#团队协作)
- [安全最佳实践](#安全最佳实践)
- [成本优化](#成本优化)

---

## 📁 项目组织

### 1. 目录结构

```
project/
├── src/
│   ├── api/
│   ├── models/
│   ├── services/
│   └── utils/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/
│   ├── api.md
│   └── README.md
├── .openhands.toml
├── requirements.txt
└── README.md
```

### 2. 配置管理

```toml
# .openhands.toml
[agent]
model = "claude-3.5-sonnet"
max_tokens = 4096
temperature = 0.3

[workspace]
exclude = ["node_modules", "venv", "__pycache__"]

[logging]
level = "INFO"
file = "~/.openhands/openhands.log"

[monitoring]
enabled = true
metrics = true
```

### 3. 环境变量

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-xxx
DATABASE_URL=postgresql://...
REDIS_URL=redis://...

# .env.example（提交到 Git）
ANTHROPIC_API_KEY=your-key-here
DATABASE_URL=your-db-url
REDIS_URL=your-redis-url
```

---

## 💎 代码质量

### 1. 代码审查清单

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# 自动代码审查
result = agent.run("""
审查以下代码：

代码：
[粘贴代码]

审查清单：
1. ✅ 代码风格（PEP 8）
2. ✅ 类型注解
3. ✅ 文档字符串
4. ✅ 错误处理
5. ✅ 单元测试
6. ✅ 性能优化
7. ✅ 安全检查
8. ✅ 可维护性
""")

print(result.output)
```

### 2. 自动化测试

```python
from openhands import Agent

def automated_testing_workflow():
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 生成测试
    agent.run("为所有 Python 文件生成单元测试")
    
    # 2. 运行测试
    result = agent.execute_command("pytest tests/ --cov=src --cov-report=html")
    
    # 3. 检查覆盖率
    if "TOTAL" in result:
        # 提取覆盖率
        coverage = extract_coverage(result)
        if coverage < 80:
            print(f"⚠️ 覆盖率过低: {coverage}%")
            # 自动补充测试
            agent.run("补充单元测试，提高覆盖率到 80% 以上")
    
    return result

# 使用
automated_testing_workflow()
```

### 3. 持续集成

```yaml
# .github/workflows/quality-check.yml
name: Quality Check
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Code Quality Check
        env:
          OPENHANDS_API_KEY: ${{ secrets.OPENHANDS_API_KEY }}
        run: |
          pip install openhands
          python scripts/quality_check.py
```

---

## 👥 团队协作

### 1. 代码规范

```python
# team_standards.py

# 1. 命名规范
# - 变量：snake_case
# - 函数：snake_case
# - 类：PascalCase
# - 常量：UPPER_CASE

# 2. 文档规范
def calculate_total(prices: list[float]) -> float:
    """
    计算总价
    
    Args:
        prices: 价格列表
    
    Returns:
        总价
    
    Example:
        >>> calculate_total([10.0, 20.0])
        30.0
    """
    return sum(prices)

# 3. 测试规范
def test_calculate_total():
    # Given
    prices = [10.0, 20.0, 30.0]
    
    # When
    total = calculate_total(prices)
    
    # Then
    assert total == 60.0
```

### 2. Code Review 流程

```python
from openhands import Agent

def code_review_workflow(pr_number):
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 获取 PR 变更
    changes = agent.get_pr_changes(pr_number)
    
    # 2. 自动审查
    review = agent.run(f"""
    审查 Pull Request #{pr_number}：
    
    变更：
    {changes}
    
    审查要点：
    1. 代码质量
    2. 测试覆盖
    3. 性能影响
    4. 安全风险
    5. 文档完整性
    """)
    
    # 3. 发布评论
    agent.create_pr_comment(pr_number, review.output)
    
    # 4. 自动批准（可选）
    if "✅ 通过" in review.output:
        agent.approve_pr(pr_number)

# 使用
code_review_workflow(123)
```

### 3. 知识共享

```python
from openhands import Agent

def create_knowledge_base():
    agent = Agent(model="claude-3.5-sonnet")
    
    # 生成项目文档
    agent.run("""
    为项目生成完整的知识库：
    
    1. README.md - 项目简介
    2. CONTRIBUTING.md - 贡献指南
    3. ARCHITECTURE.md - 架构说明
    4. API.md - API 文档
    5. FAQ.md - 常见问题
    """)
    
    print("✅ 知识库已生成")

# 使用
create_knowledge_base()
```

---

## 🔒 安全最佳实践

### 1. API Key 管理

```python
import os
from openhands import Agent

# ✅ 正确：使用环境变量
agent = Agent(
    model="claude-3.5-sonnet",
    api_key=os.getenv("ANTHROPIC_API_KEY")
)

# ❌ 错误：硬编码
# agent = Agent(
#     model="claude-3.5-sonnet",
#     api_key="sk-ant-xxx"  # 不要这样做！
# )
```

### 2. 敏感数据处理

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")

# ✅ 正确：使用占位符
result = agent.run("""
连接到数据库：

连接字符串：postgresql://user:***@localhost:5432/db
""")

# ❌ 错误：暴露真实密码
# result = agent.run("""
# 连接到数据库：
# 
# 连接字符串：postgresql://admin:password123@localhost:5432/db
# """)
```

### 3. 权限控制

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    disabled_tools=["delete_file", "git_push"],  # 禁用危险工具
    require_confirmation=True  # 需要确认
)

# 危险操作需要确认
result = agent.run("删除所有测试文件")
# 会提示：确认删除？(y/n)
```

### 4. 审计日志

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    audit_log=True,  # 启用审计日志
    audit_log_file="~/.openhands/audit.log"
)

# 所有操作都会被记录
result = agent.run("创建 FastAPI 项目")

# 查看审计日志
agent.show_audit_log()
```

---

## 💰 成本优化

### 1. 模型选择策略

```python
from openhands import Agent

def cost_optimized_agent(task):
    # 简单任务 - 便宜模型
    if is_simple_task(task):
        return Agent(model="gpt-4o", max_tokens=2048)
    
    # 中文任务 - 国产模型
    elif is_chinese_task(task):
        return Agent(model="glm-4", max_tokens=2048)
    
    # 复杂任务 - 高级模型
    else:
        return Agent(model="claude-3.5-sonnet", max_tokens=4096)

# 使用
agent = cost_optimized_agent("创建 Hello World")
result = agent.run("创建 Hello World")
```

### 2. Token 优化

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=2048,  # 减少 token
    enable_compression=True,  # 启用压缩
    enable_cache=True  # 启用缓存
)

# 简洁的任务描述
result = agent.run("""
创建 FastAPI 项目（简洁版）
""")  # 而不是冗长的描述
```

### 3. 缓存策略

```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    enable_cache=True,
    cache_ttl=7200,  # 2 小时
    cache_dir="~/.openhands/cache"
)

# 相同任务使用缓存
result1 = agent.run("创建 FastAPI 项目")  # 第一次 - 慢
result2 = agent.run("创建 FastAPI 项目")  # 第二次 - 快（使用缓存）
```

### 4. 批量处理

```python
from openhands import Agent
import asyncio

async def batch_processing():
    agent = Agent(
        model="claude-3.5-sonnet",
        enable_parallel=True,
        max_workers=4
    )
    
    tasks = [
        "创建项目结构",
        "实现用户认证",
        "添加数据库连接",
        "编写单元测试"
    ]
    
    # 并行处理（节省时间）
    results = await asyncio.gather(*[
        agent.run_async(task) for task in tasks
    ])
    
    return results

# 使用
results = asyncio.run(batch_processing())
```

---

## 📊 监控和指标

### 1. 性能监控

```python
from openhands import Agent, Monitor

monitor = Monitor()
agent = Agent(
    model="claude-3.5-sonnet",
    monitor=monitor
)

# 运行任务
result = agent.run("创建 FastAPI 项目")

# 查看指标
metrics = monitor.get_metrics()
print(f"响应时间: {metrics['response_time']}ms")
print(f"Token 使用: {metrics['tokens']}")
print(f"成本: ${metrics['cost']}")
```

### 2. 成本追踪

```python
from openhands import CostTracker

tracker = CostTracker()

# 记录每次使用的成本
tracker.record(
    model="claude-3.5-sonnet",
    tokens=1000,
    cost=0.015
)

# 查看月度成本
monthly_cost = tracker.get_monthly_cost()
print(f"本月成本: ${monthly_cost}")

# 设置预算
tracker.set_budget(100)  # $100
if tracker.is_over_budget():
    print("⚠️ 超出预算！")
```

---

## 📚 相关资源

- [最佳实践文档](https://docs.openhands.dev/best-practices)
- [安全指南](https://docs.openhands.dev/security)
- [成本优化](https://docs.openhands.dev/cost-optimization)

---

<div align="center">
  <p>🎉 遵循最佳实践，提升开发质量！</p>
</div>
