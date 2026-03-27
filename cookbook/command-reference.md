# OpenHands 命令参考手册

> 完整的命令行参考

---

## 📋 目录

- [基础命令](#基础命令)
- [高级命令](#高级命令)
- [配置命令](#配置命令)
- [管理命令](#管理命令)

---

## 🚀 基础命令

### openhands run

执行任务。

```bash
openhands run "任务描述"
```

**参数**:
- `--model` - 指定模型
- `--timeout` - 超时时间（秒）
- `--max-retries` - 最大重试次数
- `--cache` - 启用缓存
- `--debug` - 启用调试模式
- `--directory` - 工作目录
- `--template` - 使用模板

**示例**:
```bash
# 基础任务
openhands run "创建 FastAPI 项目"

# 指定模型
openhands run --model claude-3.5-sonnet "任务"

# 设置超时
openhands run --timeout 600 "复杂任务"

# 启用缓存
openhands run --cache "任务"

# 使用模板
openhands run --template api.yaml "任务"
```

---

### openhands init

初始化项目。

```bash
openhands init
```

**参数**:
- `--force` - 强制初始化
- `--template` - 使用模板

**示例**:
```bash
# 初始化
openhands init

# 强制初始化
openhands init --force

# 使用模板
openhands init --template fastapi
```

---

### openhands config

配置管理。

```bash
# 设置配置
openhands config set key value

# 获取配置
openhands config get key

# 显示所有配置
openhands config show

# 重置配置
openhands config reset
```

**示例**:
```bash
# 设置 API Key
openhands config set api-key sk-ant-xxx

# 设置默认模型
openhands config set default-model claude-3.5-sonnet

# 获取配置
openhands config get api-key

# 显示所有配置
openhands config show

# 重置配置
openhands config reset
```

---

### openhands models

模型管理。

```bash
# 列出可用模型
openhands models list

# 获取模型详情
openhands models get model-name

# 设置默认模型
openhands models set-default model-name
```

**示例**:
```bash
# 列出所有模型
openhands models list

# 获取 Claude 详情
openhands models get claude-3.5-sonnet

# 设置默认模型
openhands models set-default claude-3.5-sonnet
```

---

## 🎯 高级命令

### openhands batch

批量执行任务。

```bash
# 批量执行
openhands batch tasks.txt

# 并行执行
openhands batch --parallel tasks.txt

# 从文件读取
openhands batch --file tasks.txt
```

**示例**:
```bash
# 批量执行
openhands batch tasks.txt

# 并行执行（4 个并发）
openhands batch --parallel --max-workers 4 tasks.txt

# 从文件读取
openhands batch --file tasks.txt
```

---

### openhands template

模板管理。

```bash
# 列出模板
openhands template list

# 创建模板
openhands template create template-name

# 使用模板
openhands template use template-name
```

**示例**:
```bash
# 列出所有模板
openhands template list

# 创建模板
openhands template create api-project

# 使用模板
openhands template use api-project
```

---

### openhands workflow

工作流管理。

```bash
# 执行工作流
openhands workflow run workflow.yaml

# 验证工作流
openhands workflow validate workflow.yaml

# 列出工作流
openhands workflow list
```

**示例**:
```bash
# 执行工作流
openhands workflow run deploy.yaml

# 验证工作流
openhands workflow validate deploy.yaml

# 列出所有工作流
openhands workflow list
```

---

## ⚙️ 配置命令

### openhands cache

缓存管理。

```bash
# 清除缓存
openhands cache clear

# 查看缓存
openhands cache show

# 缓存统计
openhands cache stats
```

**示例**:
```bash
# 清除所有缓存
openhands cache clear

# 查看缓存内容
openhands cache show

# 查看缓存统计
openhands cache stats
```

---

### openhands logs

日志管理。

```bash
# 查看日志
openhands logs

# 实时查看
openhands logs --tail 100 -f

# 搜索日志
openhands logs --grep "ERROR"
```

**示例**:
```bash
# 查看最近日志
openhands logs

# 实时查看（最近 100 行）
openhands logs --tail 100 -f

# 搜索错误日志
openhands logs --grep "ERROR"

# 查看今天的日志
openhands logs --since today
```

---

## 🔧 管理命令

### openhands service

服务管理。

```bash
# 启动服务
openhands service start

# 停止服务
openhands service stop

# 重启服务
openhands service restart

# 服务状态
openhands service status
```

**示例**:
```bash
# 启动服务
openhands service start

# 停止服务
openhands service stop

# 重启服务
openhands service restart

# 查看服务状态
openhands service status
```

---

### openhands agent

Agent 管理。

```bash
# 列出 Agent
openhands agent list

# 创建 Agent
openhands agent create agent-name

# 删除 Agent
openhands agent delete agent-id

# Agent 状态
openhands agent status agent-id
```

**示例**:
```bash
# 列出所有 Agent
openhands agent list

# 创建 Agent
openhands agent create my-agent

# 删除 Agent
openhands agent delete agent-123

# 查看 Agent 状态
openhands agent status agent-123
```

---

### openhands task

任务管理。

```bash
# 列出任务
openhands task list

# 获取任务
openhands task get task-id

# 取消任务
openhands task cancel task-id

# 重试任务
openhands task retry task-id
```

**示例**:
```bash
# 列出所有任务
openhands task list

# 获取任务详情
openhands task get task-123

# 取消任务
openhands task cancel task-123

# 重试任务
openhands task retry task-123
```

---

## 📊 监控命令

### openhands monitor

监控管理。

```bash
# 启动监控
openhands monitor start

# 停止监控
openhands monitor stop

# 查看监控
openhands monitor show
```

**示例**:
```bash
# 启动监控
openhands monitor start

# 停止监控
openhands monitor stop

# 查看监控数据
openhands monitor show

# 导出监控数据
openhands monitor export --format json
```

---

### openhands metrics

指标管理。

```bash
# 查看指标
openhands metrics show

# 导出指标
openhands metrics export
```

**示例**:
```bash
# 查看指标
openhands metrics show

# 导出指标（JSON）
openhands metrics export --format json

# 导出指标（Prometheus）
openhands metrics export --format prometheus
```

---

## 🆘 帮助命令

### openhands help

显示帮助。

```bash
# 显示帮助
openhands help

# 显示命令帮助
openhands help command-name
```

**示例**:
```bash
# 显示帮助
openhands help

# 显示 run 命令帮助
openhands help run

# 显示 config 命令帮助
openhands help config
```

---

### openhands version

显示版本。

```bash
# 显示版本
openhands version

# 显示详细信息
openhands version --verbose
```

**示例**:
```bash
# 显示版本
openhands version

# 显示详细信息
openhands version --verbose
```

---

## 🔗 完整命令列表

| 命令 | 说明 |
|------|------|
| `openhands run` | 执行任务 |
| `openhands init` | 初始化项目 |
| `openhands config` | 配置管理 |
| `openhands models` | 模型管理 |
| `openhands batch` | 批量执行 |
| `openhands template` | 模板管理 |
| `openhands workflow` | 工作流管理 |
| `openhands cache` | 缓存管理 |
| `openhands logs` | 日志管理 |
| `openhands service` | 服务管理 |
| `openhands agent` | Agent 管理 |
| `openhands task` | 任务管理 |
| `openhands monitor` | 监控管理 |
| `openhands metrics` | 指标管理 |
| `openhands help` | 显示帮助 |
| `openhands version` | 显示版本 |

---

<div align="center">
  <p>📖 完整命令参考，随时查阅！</p>
</div>
