# OpenHands 故障排查手册

> 常见问题和解决方案

---

## 📋 目录

- [安装问题](#安装问题)
- [配置问题](#配置问题)
- [执行问题](#执行问题)
- [性能问题](#性能问题)

---

## 🔧 安装问题

### 问题 1: pip 安装失败

**错误信息**:
```
ERROR: Could not find a version that satisfies the requirement openhands
```

**解决方案**:
```bash
# 方式一：升级 pip
pip install --upgrade pip

# 方式二：使用 conda
conda install -c conda-forge openhands

# 方式三：从源码安装
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands
pip install -e .
```

---

### 问题 2: 依赖冲突

**错误信息**:
```
ERROR: Cannot install openhands because these package versions have conflicting dependencies
```

**解决方案**:
```bash
# 创建新的虚拟环境
python -m venv openhands-env
source openhands-env/bin/activate  # Linux/Mac
# 或
openhands-env\Scripts\activate  # Windows

# 重新安装
pip install openhands
```

---

## ⚙️ 配置问题

### 问题 1: API Key 无效

**错误信息**:
```
Error: Invalid API key
```

**解决方案**:
```bash
# 1. 检查 API Key 格式
echo $ANTHROPIC_API_KEY
# 应该显示：sk-ant-xxx

# 2. 重新设置
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 3. 验证
openhands config show
```

---

### 问题 2: 模型不可用

**错误信息**:
```
Error: Model claude-3.5-sonnet not found
```

**解决方案**:
```bash
# 1. 检查可用模型
openhands models list

# 2. 使用正确的模型名称
openhands run --model claude-3.5-sonnet "任务"

# 3. 或使用别名
openhands run --model claude "任务"
```

---

## 🚀 执行问题

### 问题 1: 任务超时

**错误信息**:
```
Error: Task execution timeout after 300 seconds
```

**解决方案**:
```bash
# 方式一：增加超时时间
openhands run --timeout 600 "复杂任务"

# 方式二：分解任务
openhands run "创建项目结构"
openhands run "实现用户认证"
openhands run "编写单元测试"

# 方式三：使用更快的模型
openhands run --model gpt-4o "任务"
```

---

### 问题 2: 内存不足

**错误信息**:
```
Error: Out of memory
```

**解决方案**:
```bash
# 方式一：减少并发
openhands run --max-workers 1 "任务"

# 方式二：使用流式处理
openhands run --stream "大文件处理"

# 方式三：清理缓存
openhands cache clear
```

---

### 问题 3: 网络错误

**错误信息**:
```
Error: Connection timeout
```

**解决方案**:
```bash
# 方式一：使用代理
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"

# 方式二：增加超时时间
openhands run --timeout 600 "任务"

# 方式三：重试
openhands run --max-retries 3 "任务"
```

---

## ⚡ 性能问题

### 问题 1: 响应慢

**症状**:
- 任务执行时间 > 10 秒
- 频繁超时

**解决方案**:
```python
from openhands import Agent

# 1. 使用缓存
agent = Agent(
    model="claude-3.5-sonnet",
    enable_cache=True,
    cache_ttl=3600
)

# 2. 启用并行
agent = Agent(
    model="claude-3.5-sonnet",
    enable_parallel=True,
    max_workers=4
)

# 3. 减少 token
agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=2048
)
```

---

### 问题 2: CPU 使用率高

**症状**:
- CPU 使用率 > 90%
- 系统卡顿

**解决方案**:
```python
from openhands import Agent

# 1. 限制并发
agent = Agent(
    model="claude-3.5-sonnet",
    max_workers=2
)

# 2. 使用更轻量的模型
agent = Agent(model="gpt-3.5-turbo")

# 3. 优化任务描述
result = agent.run("创建项目（简洁版）")
```

---

### 问题 3: 内存泄漏

**症状**:
- 内存使用持续增长
- 系统变慢

**解决方案**:
```python
from openhands import Agent

# 1. 定期清理
agent = Agent(model="claude-3.5-sonnet")

for i in range(100):
    result = agent.run(f"任务 {i}")
    # 定期清理
    if i % 10 == 0:
        agent.clear_cache()

# 2. 使用上下文管理器
with Agent(model="claude-3.5-sonnet") as agent:
    result = agent.run("任务")
# 自动清理资源
```

---

## 🔍 调试技巧

### 1. 启用调试日志

```bash
# 启用详细日志
openhands run --debug "任务"

# 查看日志
tail -f ~/.openhands/logs/openhands.log
```

### 2. 性能分析

```python
from openhands import Agent, Profiler

profiler = Profiler()
agent = Agent(
    model="claude-3.5-sonnet",
    profiler=profiler
)

result = agent.run("任务")

# 查看性能报告
print(profiler.get_report())
```

### 3. 错误追踪

```python
from openhands import Agent
import traceback

try:
    agent = Agent(model="claude-3.5-sonnet")
    result = agent.run("任务")
except Exception as e:
    # 打印完整错误信息
    traceback.print_exc()
    
    # 查看详细日志
    print(agent.get_last_error())
```

---

## 📊 故障排查流程

```
1. 确认问题
   ↓
2. 查看错误日志
   ↓
3. 检查配置
   ↓
4. 尝试基本解决方案
   ↓
5. 启用调试模式
   ↓
6. 性能分析
   ↓
7. 寻求帮助
```

---

## 🆘 寻求帮助

### 官方支持

- **GitHub Issues**: https://github.com/OpenHands/OpenHands/issues
- **Discord**: https://discord.gg/openhands
- **文档**: https://docs.openhands.dev

### 社区支持

- **Stack Overflow**: [openhands] 标签
- **Reddit**: r/OpenHands
- **中文社区**: https://github.com/srxly888-creator/openhands-cookbook

---

## 📚 相关资源

- [性能优化指南](../cookbook/performance-optimization.md)
- [测试框架](../cookbook/testing-guide.md)
- [最佳实践](../cookbook/best-practices.md)

---

<div align="center">
  <p>🔧 快速定位问题，高效解决！</p>
</div>
