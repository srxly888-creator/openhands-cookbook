# OpenHands CLI 模式详细指南

> 命令行界面，轻量级、快速迭代的最佳选择

---

## 📋 目录

- [安装配置](#安装配置)
- [基础使用](#基础使用)
- [高级功能](#高级功能)
- [最佳实践](#最佳实践)
- [故障排查](#故障排查)

---

## 🔧 安装配置

### 系统要求

- Python 3.8+
- pip 或 conda
- Git

### 安装方式

#### 方式一：pip 安装（推荐）

```bash
pip install openhands
```

#### 方式二：从源码安装

```bash
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands
pip install -e .
```

### 配置 API Key

#### Claude API

```bash
# 方式一：环境变量
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 方式二：配置文件
echo "ANTHROPIC_API_KEY=sk-ant-xxx" > ~/.openhands.env
```

#### OpenAI API

```bash
export OPENAI_API_KEY="sk-xxx"
```

#### GLM API

```bash
export GLM_API_KEY="xxx.xxx"
```

### 验证安装

```bash
openhands --version
openhands config show
```

---

## 🚀 基础使用

### 1. 交互式会话

```bash
openhands run
```

**示例对话**：
```
You: 帮我创建一个 FastAPI 项目
Agent: 好的，我将创建一个 FastAPI 项目。请告诉我项目名称和主要功能。

You: 项目名 my-api，需要用户认证
Agent: [开始创建项目...]
```

### 2. 单次任务

```bash
# 基础任务
openhands run "创建一个 FastAPI 项目"

# 带文件的任务
openhands run "优化这个代码的性能" --file main.py

# 带目录的任务
openhands run "重构这个项目" --directory ./my-project
```

### 3. 指定模型

```bash
# 使用 Claude 3.5 Sonnet
openhands run --model claude-3.5-sonnet "任务描述"

# 使用 GPT-4o
openhands run --model gpt-4o "任务描述"

# 使用 GLM-4
openhands run --model glm-4 "任务描述"
```

### 4. 工作目录

```bash
# 指定工作目录
openhands run --directory /path/to/project "任务描述"

# 使用当前目录
openhands run --directory . "任务描述"
```

---

## 🎯 高级功能

### 1. 多文件操作

```bash
# 批量优化多个文件
openhands run "优化所有 Python 文件的性能" --directory ./src

# 重构整个项目
openhands run "将项目从 Flask 迁移到 FastAPI" --directory ./my-flask-app
```

### 2. Git 集成

```bash
# 自动提交更改
openhands run --git-commit "添加用户认证功能"

# 创建 PR
openhands run --git-pr "实现用户登录功能"
```

### 3. 配置文件

创建 `.openhands.toml`：

```toml
[default]
model = "claude-3.5-sonnet"
max_tokens = 4096
temperature = 0.7

[git]
auto_commit = true
commit_message_template = "feat: {task_description}"

[logging]
level = "INFO"
file = "~/.openhands/openhands.log"
```

### 4. 自定义提示词

```bash
# 使用自定义提示词
openhands run --prompt-file custom_prompt.txt "任务描述"
```

**custom_prompt.txt 示例**：
```
你是一个专业的 Python 开发者。
- 遵循 PEP 8 规范
- 优先使用类型注解
- 添加详细的文档字符串
```

---

## 📊 最佳实践

### 1. 任务描述技巧

❌ **不好的描述**：
```
优化代码
```

✅ **好的描述**：
```
优化 main.py 中的数据处理函数：
1. 减少内存占用
2. 提高处理速度
3. 保持代码可读性
```

### 2. 分步骤执行

```bash
# 第一步：创建项目结构
openhands run "创建 FastAPI 项目结构，包含 routers、models、schemas 目录"

# 第二步：实现用户认证
openhands run "在 routers/auth.py 中实现用户注册和登录功能"

# 第三步：添加测试
openhands run "为用户认证功能编写单元测试"
```

### 3. 使用 Git

```bash
# 在执行任务前创建分支
git checkout -b feature/user-auth

# 执行任务
openhands run "实现用户认证功能"

# 检查更改
git diff

# 提交
git add .
git commit -m "feat: 实现用户认证功能"
```

### 4. 错误处理

```bash
# 如果任务失败，查看日志
openhands logs --tail 100

# 重试任务
openhands run --retry "任务描述"
```

---

## 🔧 故障排查

### 问题 1：API Key 无效

**错误信息**：
```
Error: Invalid API key
```

**解决方案**：
```bash
# 检查环境变量
echo $ANTHROPIC_API_KEY

# 重新设置
export ANTHROPIC_API_KEY="your-valid-key"
```

### 问题 2：网络连接失败

**错误信息**：
```
Error: Connection timeout
```

**解决方案**：
```bash
# 使用代理
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"
```

### 问题 3：内存不足

**错误信息**：
```
Error: Out of memory
```

**解决方案**：
```bash
# 减少并发数
openhands run --max-workers 1 "任务描述"

# 使用更小的模型
openhands run --model gpt-3.5-turbo "任务描述"
```

### 问题 4：权限错误

**错误信息**：
```
Error: Permission denied
```

**解决方案**：
```bash
# 检查文件权限
ls -la

# 修改权限
chmod 755 script.sh
```

---

## 📚 实用命令速查

```bash
# 查看帮助
openhands --help

# 查看版本
openhands --version

# 查看配置
openhands config show

# 查看日志
openhands logs --tail 100

# 清理缓存
openhands cache clear

# 更新 OpenHands
pip install --upgrade openhands
```

---

## 🎓 进阶技巧

### 1. 批量任务

创建 `tasks.txt`：
```
创建项目结构
实现用户认证
添加数据库迁移
编写单元测试
生成 API 文档
```

执行：
```bash
while read task; do
  openhands run "$task"
done < tasks.txt
```

### 2. 自定义工作流

创建 `workflow.sh`：
```bash
#!/bin/bash
# OpenHands 自动化工作流

# 1. 创建分支
git checkout -b feature/$1

# 2. 执行任务
openhands run "$2"

# 3. 运行测试
pytest tests/

# 4. 提交代码
git add .
git commit -m "feat: $1"

# 5. 创建 PR
gh pr create --title "feat: $1" --body "$2"
```

使用：
```bash
chmod +x workflow.sh
./workflow.sh user-auth "实现用户认证功能"
```

---

## 📖 下一步

- 📖 [GUI 模式指南](gui-quickstart.md)
- 📖 [Cloud 模式指南](cloud-quickstart.md)
- 🎯 [基础案例](../examples/basic/)
- 📖 [SDK 编程指南](sdk-guide.md)

---

<div align="center">
  <p>🎉 掌握 CLI 模式，提升开发效率！</p>
</div>
