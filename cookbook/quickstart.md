# OpenHands 快速开始指南

> 5 分钟上手 OpenHands，开始你的 AI 驱动开发之旅！

---

## 📋 前置要求

- Python 3.8+
- Git
- API Key（Claude / OpenAI / 其他 LLM）

---

## 🚀 三种使用方式

### 方式一：CLI 模式（推荐）

**优点**：
- ✅ 安装简单
- ✅ 快速迭代
- ✅ 轻量级

**安装**：
```bash
pip install openhands
```

**使用**：
```bash
# 交互式会话
openhands run

# 指定任务
openhands run "帮我创建一个 FastAPI 项目，包含用户认证"

# 使用特定模型
openhands run --model claude-3.5-sonnet "优化这段代码"
```

---

### 方式二：Local GUI

**优点**：
- ✅ 可视化界面
- ✅ 实时预览
- ✅ 多项目管理

**安装**：
```bash
# 克隆仓库
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands

# 构建镜像
make build

# 启动服务
make run

# 访问界面
open http://localhost:3000
```

**配置 API Key**：
1. 访问 http://localhost:3000
2. 点击右上角设置图标
3. 输入 API Key
4. 选择模型

---

### 方式三：Cloud 模式（最简单）

**优点**：
- ✅ 无需安装
- ✅ 免费试用
- ✅ 即开即用

**步骤**：
1. 访问 https://app.all-hands.dev
2. 使用 GitHub / GitLab 登录
3. 选择 Minimax 模型（免费）
4. 开始对话

---

## 🎯 第一个任务

### 示例 1：创建 Web 项目

**CLI 模式**：
```bash
openhands run "创建一个 FastAPI 项目，包含：
1. 用户注册/登录 API
2. JWT 认证
3. SQLite 数据库
4. 单元测试"
```

**GUI/Cloud 模式**：
直接在对话框输入：
```
创建一个 FastAPI 项目，包含：
1. 用户注册/登录 API
2. JWT 认证
3. SQLite 数据库
4. 单元测试
```

**预期结果**：
- ✅ 自动生成项目结构
- ✅ 创建所有必需文件
- ✅ 编写单元测试
- ✅ 提供运行说明

---

### 示例 2：代码优化

**CLI 模式**：
```bash
openhands run "优化这段 Python 代码的性能：
[粘贴你的代码]"
```

**预期结果**：
- ✅ 分析代码瓶颈
- ✅ 提供优化建议
- ✅ 重构代码
- ✅ 对比性能差异

---

### 示例 3：Bug 修复

**CLI 模式**：
```bash
openhands run "修复这个错误：
RuntimeError: CUDA out of memory

代码：
[粘贴你的代码]"
```

**预期结果**：
- ✅ 诊断问题原因
- ✅ 提供解决方案
- ✅ 修复代码
- ✅ 添加错误处理

---

## 🔑 API Key 配置

### Claude API（推荐）

1. 访问 https://console.anthropic.com
2. 创建 API Key
3. 设置环境变量：
```bash
export ANTHROPIC_API_KEY="your-api-key"
```

### OpenAI API

1. 访问 https://platform.openai.com
2. 创建 API Key
3. 设置环境变量：
```bash
export OPENAI_API_KEY="your-api-key"
```

### GLM API（国产推荐）

1. 访问 https://open.bigmodel.cn
2. 创建 API Key
3. 设置环境变量：
```bash
export GLM_API_KEY="your-api-key"
```

---

## 📊 模型选择建议

| 模型 | 推荐场景 | 成本 | 中文支持 |
|------|---------|------|---------|
| **Claude 3.5 Sonnet** | 复杂推理、代码生成 | $$$ | ⭐⭐⭐⭐ |
| **GPT-4o** | 通用任务 | $$$ | ⭐⭐⭐ |
| **GLM-4** | 中文任务、性价比 | $$ | ⭐⭐⭐⭐⭐ |
| **Minimax** | 免费试用 | Free | ⭐⭐⭐⭐ |

---

## 🎓 学习路径

### 第 1 天
- ✅ 安装 OpenHands
- ✅ 配置 API Key
- ✅ 完成 3 个基础任务

### 第 1 周
- ✅ 掌握 CLI 模式
- ✅ 尝试 GUI 模式
- ✅ 完成 10 个不同类型任务

### 第 1 月
- ✅ 学习 SDK 编程
- ✅ 创建自定义 Agent
- ✅ 集成到工作流

---

## ❓ 常见问题

### Q: OpenHands 支持哪些编程语言？
**A**: 支持所有主流语言：Python、JavaScript、TypeScript、Java、Go、Rust 等。

### Q: 如何选择模型？
**A**: 
- 复杂任务 → Claude 3.5 Sonnet
- 中文任务 → GLM-4
- 免费试用 → Minimax

### Q: 数据安全吗？
**A**: 
- CLI/GUI 模式：数据在本地
- Cloud 模式：数据加密传输

### Q: 如何提高响应质量？
**A**: 
- 提供详细的任务描述
- 包含错误信息和日志
- 明确预期输出

---

## 📚 下一步

- 📖 [CLI 详细指南](cli-quickstart.md)
- 📖 [GUI 详细指南](gui-quickstart.md)
- 📖 [Cloud 详细指南](cloud-quickstart.md)
- 🎯 [基础案例](../examples/basic/)

---

## 🤝 需要帮助？

- 💬 GitHub Issues: https://github.com/srxly888-creator/openhands-cookbook/issues
- 📖 官方文档: https://docs.openhands.dev
- 💬 社区 Slack: https://dub.sh/openhands

---

<div align="center">
  <p>🎉 恭喜！你已经掌握了 OpenHands 的基础使用！</p>
</div>
