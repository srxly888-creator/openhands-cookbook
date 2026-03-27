# OpenHands 快速索引

> 一页纸速查表

---

## 🚀 快速开始

### 安装

```bash
# CLI
pip install openhands

# Docker
docker pull openhands/openhands

# Cloud
访问 https://app.all-hands.dev
```

### 配置

```bash
# API Key
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 或
openhands config set api-key sk-ant-xxx
```

### 执行任务

```bash
# 基础任务
openhands run "创建 FastAPI 项目"

# 指定模型
openhands run --model claude-3.5-sonnet "任务"

# 设置超时
openhands run --timeout 600 "复杂任务"
```

---

## 📝 核心命令

### 基础命令

| 命令 | 说明 |
|------|------|
| `openhands run` | 执行任务 |
| `openhands init` | 初始化项目 |
| `openhands config` | 配置管理 |
| `openhands models` | 模型管理 |

### 高级命令

| 命令 | 说明 |
|------|------|
| `openhands batch` | 批量执行 |
| `openhands template` | 模板管理 |
| `openhands workflow` | 工作流管理 |

### 管理命令

| 命令 | 说明 |
|------|------|
| `openhands cache` | 缓存管理 |
| `openhands logs` | 日志管理 |
| `openhands service` | 服务管理 |

---

## 💻 Python API

### 基础使用

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet")
result = agent.run("创建 FastAPI 项目")
print(result.output)
```

### 异步使用

```python
import asyncio
from openhands import Agent

async def main():
    agent = Agent(model="claude-3.5-sonnet")
    result = await agent.run_async("创建项目")
    print(result.output)

asyncio.run(main())
```

### 批量执行

```python
from openhands import Agent

agent = Agent(model="claude-3.5-sonnet", enable_parallel=True)
tasks = ["任务1", "任务2", "任务3"]
results = agent.batch(tasks)
```

---

## 🎯 常用配置

### 环境变量

```bash
# API Key
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 默认模型
export OPENHANDS_MODEL="claude-3.5-sonnet"

# 超时时间
export OPENHANDS_TIMEOUT="300"

# 调试模式
export OPENHANDS_DEBUG="true"
```

### 配置文件

```yaml
# config.yaml
model: claude-3.5-sonnet
max_tokens: 4096
temperature: 0.7
timeout: 300
enable_cache: true
debug: false
```

---

## 📊 性能优化

### 1. 启用缓存

```python
agent = Agent(
    model="claude-3.5-sonnet",
    enable_cache=True,
    cache_ttl=3600
)
```

### 2. 并行处理

```python
agent = Agent(
    model="claude-3.5-sonnet",
    enable_parallel=True,
    max_workers=4
)
```

### 3. Token 优化

```python
agent = Agent(
    model="claude-3.5-sonnet",
    max_tokens=2048
)
```

---

## 🔧 故障排查

### 常见问题

| 问题 | 解决方案 |
|------|---------|
| **API Key 无效** | `export ANTHROPIC_API_KEY="sk-ant-xxx"` |
| **任务超时** | `openhands run --timeout 600 "任务"` |
| **内存不足** | `openhands run --max-workers 1 "任务"` |
| **网络错误** | `export HTTP_PROXY="http://127.0.0.1:7890"` |

### 调试模式

```bash
# 启用调试
openhands run --debug "任务"

# 查看日志
tail -f ~/.openhands/logs/openhands.log
```

---

## 📚 学习路径

### 初学者

1. ✅ 快速开始
2. ✅ 基础命令
3. ✅ 第一个任务
4. ✅ 常见问题

### 进阶用户

1. ✅ 高级命令
2. ✅ Python API
3. ✅ 性能优化
4. ✅ 最佳实践

### 企业用户

1. ✅ 企业部署
2. ✅ CI/CD 集成
3. ✅ 监控告警
4. ✅ 安全加固

---

## 🔗 重要链接

- **官方文档**: https://docs.openhands.dev
- **GitHub**: https://github.com/OpenHands/OpenHands
- **Discord**: https://discord.gg/openhands
- **中文教程**: https://github.com/srxly888-creator/openhands-cookbook

---

## 💡 快速技巧

### 1. 任务分解

```bash
# ❌ 不好
openhands run "创建完整的电商系统"

# ✅ 好
openhands run "创建项目结构"
openhands run "实现用户认证"
openhands run "实现商品管理"
```

### 2. 使用模板

```bash
# 创建模板
openhands template create api-project

# 使用模板
openhands run --template api-project "任务"
```

### 3. 缓存复用

```bash
# 启用缓存
openhands run --cache "任务"

# 清除缓存
openhands cache clear
```

---

## 🎯 最佳实践

### ✅ DO

- 使用类型注解
- 参数化查询
- 启用缓存
- 任务分解

### ❌ DON'T

- 直接拼接 SQL
- 一次性复杂任务
- 忽略错误处理
- 硬编码密钥

---

<div align="center">
  <p>📖 一页纸速查，随时参考！</p>
</div>
