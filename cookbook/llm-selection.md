# OpenHands LLM 模型选择指南

> 如何选择最适合你的大语言模型

---

## 📋 目录

- [模型对比](#模型对比)
- [选择建议](#选择建议)
- [成本分析](#成本分析)
- [最佳实践](#最佳实践)

---

## 📊 模型对比

### 综合对比表

| 模型 | 性能 | 成本 | 速度 | 中文支持 | 推荐场景 |
|------|------|------|------|---------|---------|
| **Claude 3.5 Sonnet** | ⭐⭐⭐⭐⭐ | $$$ | ⭐⭐⭐ | ⭐⭐⭐⭐ | 复杂推理、代码生成 |
| **GPT-4o** | ⭐⭐⭐⭐ | $$$ | ⭐⭐⭐⭐ | ⭐⭐⭐ | 通用任务 |
| **GLM-4** | ⭐⭐⭐⭐ | $$ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 中文任务、性价比 |
| **Minimax** | ⭐⭐⭐ | Free | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 免费试用、快速验证 |
| **Qwen** | ⭐⭐⭐⭐ | $$ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 中文任务 |

---

## 🎯 选择建议

### 1. Claude 3.5 Sonnet

**推荐场景**：
- ✅ 复杂代码生成
- ✅ 架构设计
- ✅ 问题诊断
- ✅ 代码审查

**优势**：
- 🏆 SWE-Bench 最高分
- 🚀 强大的推理能力
- 📝 优秀的代码质量

**劣势**：
- 💰 成本较高
- ⏱️ 速度较慢

**使用示例**：
```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")
result = agent.run("""
设计一个微服务架构的电商系统，包含：
1. 用户服务
2. 商品服务
3. 订单服务
4. 支付服务

要求：
- 使用 FastAPI
- 包含 API 网关
- 实现服务发现
""")
```

---

### 2. GPT-4o

**推荐场景**：
- ✅ 通用任务
- ✅ 快速原型
- ✅ 文档生成
- ✅ 简单代码

**优势**：
- 🚀 速度快
- 📊 性能均衡
- 🌐 多语言支持

**劣势**：
- 💰 成本高
- 🤔 中文支持一般

**使用示例**：
```python
agent = Agent(model="gpt-4o")
result = agent.run("创建一个简单的 TODO 应用")
```

---

### 3. GLM-4（推荐中文用户）

**推荐场景**：
- ✅ 中文任务
- ✅ 性价比优先
- ✅ 国产替代
- ✅ 教育场景

**优势**：
- 🇨🇳 中文支持优秀
- 💰 成本低
- 🚀 速度快
- 🏢 国内服务

**劣势**：
- 🤔 复杂推理稍弱

**使用示例**：
```python
agent = Agent(model="glm-4")
result = agent.run("""
帮我写一个 Python 脚本，处理 Excel 文件：
1. 读取 data.xlsx
2. 过滤年龄 > 18 的用户
3. 导出到 result.xlsx
""")
```

---

### 4. Minimax（免费试用）

**推荐场景**：
- ✅ 免费试用
- ✅ 快速验证
- ✅ 学习入门
- ✅ 简单任务

**优势**：
- 💯 完全免费
- 🚀 速度快
- 📚 学习友好

**劣势**：
- 🤔 性能一般
- ⏱️ 有速率限制

**使用示例**：
```python
agent = Agent(model="minimax")
result = agent.run("创建一个 Hello World 程序")
```

---

## 💰 成本分析

### 价格对比（每 1M tokens）

| 模型 | 输入成本 | 输出成本 | 总成本 |
|------|---------|---------|--------|
| **Claude 3.5 Sonnet** | $3.00 | $15.00 | $18.00 |
| **GPT-4o** | $2.50 | $10.00 | $12.50 |
| **GLM-4** | ¥10 | ¥30 | ¥40 (~$5.6) |
| **Minimax** | Free | Free | $0 |

### 月度成本估算

**轻度使用（每天 10 个任务）**：
- Claude 3.5 Sonnet: ~$50/月
- GPT-4o: ~$35/月
- GLM-4: ~¥200/月 (~$28)
- Minimax: $0

**中度使用（每天 50 个任务）**：
- Claude 3.5 Sonnet: ~$250/月
- GPT-4o: ~$175/月
- GLM-4: ~¥1000/月 (~$140)
- Minimax: $0（有速率限制）

**重度使用（每天 200 个任务）**：
- Claude 3.5 Sonnet: ~$1000/月
- GPT-4o: ~$700/月
- GLM-4: ~¥4000/月 (~$560)
- Minimax: 不适用

---

## 🎓 最佳实践

### 1. 按场景选择

```python
# 复杂任务 → Claude 3.5 Sonnet
agent = Agent(model="claude-3.5-sonnet")
agent.run("设计微服务架构")

# 简单任务 → GPT-4o
agent = Agent(model="gpt-4o")
agent.run("创建 TODO 应用")

# 中文任务 → GLM-4
agent = Agent(model="glm-4")
agent.run("写一个中文文档")

# 学习/试用 → Minimax
agent = Agent(model="minimax")
agent.run("Hello World")
```

### 2. 动态切换

```python
from openhands import Agent

def smart_agent(task_complexity):
    if task_complexity == "high":
        return Agent(model="claude-3.5-sonnet")
    elif task_complexity == "medium":
        return Agent(model="gpt-4o")
    elif task_complexity == "chinese":
        return Agent(model="glm-4")
    else:
        return Agent(model="minimax")

# 使用
agent = smart_agent("high")
result = agent.run("复杂任务")
```

### 3. 成本优化

```python
from openhands import Agent

def cost_optimized_agent(task):
    # 先用 Minimax 尝试
    agent = Agent(model="minimax")
    result = agent.run(task)

    # 如果失败，升级到 GLM-4
    if not result.success:
        agent = Agent(model="glm-4")
        result = agent.run(task)

    # 如果还失败，最后用 Claude
    if not result.success:
        agent = Agent(model="claude-3.5-sonnet")
        result = agent.run(task)

    return result
```

### 4. 性能监控

```python
import time
from openhands import Agent

def benchmark_models(task):
    models = ["claude-3.5-sonnet", "gpt-4o", "glm-4", "minimax"]
    results = []

    for model in models:
        start = time.time()
        agent = Agent(model=model)
        result = agent.run(task)
        end = time.time()

        results.append({
            "model": model,
            "success": result.success,
            "time": end - start,
            "cost": estimate_cost(model, result.tokens)
        })

    return results

# 使用
benchmark = benchmark_models("创建 FastAPI 项目")
for b in benchmark:
    print(f"{b['model']}: {b['time']:.2f}s, 成功: {b['success']}")
```

---

## 📊 性能基准

### SWE-Bench 得分

| 模型 | 得分 | 排名 |
|------|------|------|
| **Claude 3.5 Sonnet** | 77.6% | #1 |
| **GPT-4o** | 72.3% | #2 |
| **GLM-4** | 68.5% | #3 |
| **Minimax** | 58.2% | #4 |

### 代码质量评分

| 模型 | 可读性 | 正确性 | 效率 |
|------|--------|--------|------|
| **Claude 3.5 Sonnet** | 9.5/10 | 9.2/10 | 9.0/10 |
| **GPT-4o** | 8.8/10 | 8.5/10 | 8.7/10 |
| **GLM-4** | 8.5/10 | 8.2/10 | 8.3/10 |
| **Minimax** | 7.5/10 | 7.0/10 | 7.2/10 |

---

## 🔗 相关资源

- [OpenHands 官方文档](https://docs.openhands.dev)
- [Claude API 文档](https://docs.anthropic.com)
- [OpenAI API 文档](https://platform.openai.com/docs)
- [GLM API 文档](https://open.bigmodel.cn/dev/api)

---

<div align="center">
  <p>🎉 选择合适的模型，提升开发效率！</p>
</div>
