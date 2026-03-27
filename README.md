# OpenHands Cookbook 🚀

<div align="center">
  <img src="https://raw.githubusercontent.com/OpenHands/docs/main/openhands/static/img/logo.png" alt="OpenHands Logo" width="200">
  
  <h3>AI 驱动开发的实战手册</h3>
  
  <a href="https://github.com/srxly888-creator/openhands-cookbook/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/LICENSE-MIT-20B2AA?style=for-the-badge" alt="MIT License">
  </a>
  <a href="https://github.com/OpenHands/OpenHands">
    <img src="https://img.shields.io/badge/OpenHands-77.6%25-00cc00?style=for-the-badge" alt="SWE-Bench Score">
  </a>
  <a href="https://docs.openhands.dev">
    <img src="https://img.shields.io/badge/官方文档-000?logo=googledocs&logoColor=FFE165&style=for-the-badge" alt="Documentation">
  </a>
  
  <p><strong>中文优先</strong> | <a href="README_EN.md">English</a></p>
</div>

---

## 📖 简介

OpenHands Cookbook 是一个面向中文用户的实战手册，帮助你快速上手 OpenHands——一个强大的 AI 驱动开发平台。

### ✨ 特色

- 🇨🇳 **中文优先** - 所有内容默认中文，降低学习门槛
- 🚀 **实战导向** - 提供真实案例和最佳实践
- 📚 **系统全面** - 覆盖从入门到精通的完整路径
- 🔄 **持续更新** - 跟进 OpenHands 最新功能

---

## 🎯 快速开始

### 方式一：CLI 模式（推荐新手）

最简单的使用方式，类似 Claude Code / Codex 的体验。

```bash
# 安装 OpenHands CLI
pip install openhands

# 启动交互式会话
openhands run

# 指定任务
openhands run "帮我创建一个 FastAPI 项目"
```

**详细指南**: [cookbook/cli-quickstart.md](cookbook/cli-quickstart.md)

### 方式二：Local GUI

本地可视化界面，类似 Devin / Jules 的体验。

```bash
# 克隆仓库
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands

# 启动服务
make build
make run

# 访问界面
open http://localhost:3000
```

**详细指南**: [cookbook/gui-quickstart.md](cookbook/gui-quickstart.md)

### 方式三：Cloud（免费试用）

无需本地部署，直接使用托管服务。

1. 访问 https://app.all-hands.dev
2. 使用 GitHub / GitLab 账号登录
3. 选择 Minimax 模型（免费）
4. 开始使用

**详细指南**: [cookbook/cloud-quickstart.md](cookbook/cloud-quickstart.md)

---

## 📚 Cookbook 目录

### 🚀 入门篇

| 章节 | 说明 | 链接 |
|------|------|------|
| **快速开始** | 5 分钟上手指南 | [cookbook/quickstart.md](cookbook/quickstart.md) |
| **CLI 模式** | 命令行使用详解 | [cookbook/cli-quickstart.md](cookbook/cli-quickstart.md) |
| **GUI 模式** | 可视化界面使用 | [cookbook/gui-quickstart.md](cookbook/gui-quickstart.md) |
| **Cloud 模式** | 云端服务使用 | [cookbook/cloud-quickstart.md](cookbook/cloud-quickstart.md) |

### 🔧 进阶篇

| 章节 | 说明 | 链接 |
|------|------|------|
| **SDK 使用** | Python SDK 编程指南 | [cookbook/sdk-guide.md](cookbook/sdk-guide.md) |
| **Agent 配置** | Agent 参数调优 | [cookbook/agent-config.md](cookbook/agent-config.md) |
| **LLM 选择** | 大模型选择策略 | [cookbook/llm-selection.md](cookbook/llm-selection.md) |
| **工具集成** | 集成 Slack/Jira/Linear | [cookbook/integrations.md](cookbook/integrations.md) |

### 🎯 实战篇

| 章节 | 说明 | 链接 |
|------|------|------|
| **Web 开发** | FastAPI/Django 实战 | [examples/web-development/](examples/web-development/) |
| **数据分析** | Pandas/NumPy 项目 | [examples/data-analysis/](examples/data-analysis/) |
| **自动化任务** | CI/CD 自动化 | [examples/automation/](examples/automation/) |
| **代码审查** | 自动化 Code Review | [examples/code-review/](examples/code-review/) |

### 🏢 企业篇

| 章节 | 说明 | 链接 |
|------|------|------|
| **自托管部署** | Kubernetes 部署指南 | [cookbook/enterprise-deployment.md](cookbook/enterprise-deployment.md) |
| **安全配置** | 企业级安全最佳实践 | [cookbook/enterprise-security.md](cookbook/enterprise-security.md) |
| **性能优化** | 大规模 Agent 部署 | [cookbook/enterprise-optimization.md](cookbook/enterprise-optimization.md) |

---

## 🎓 学习路径

### 初级（1-2 周）

1. ✅ 阅读 [快速开始](cookbook/quickstart.md)
2. ✅ 完成 [CLI 模式](cookbook/cli-quickstart.md) 练习
3. ✅ 尝试 [Cloud 模式](cookbook/cloud-quickstart.md)
4. ✅ 完成 3 个 [基础案例](examples/basic/)

### 中级（2-4 周）

1. ✅ 掌握 [SDK 使用](cookbook/sdk-guide.md)
2. ✅ 学习 [Agent 配置](cookbook/agent-config.md)
3. ✅ 完成 [Web 开发](examples/web-development/) 案例
4. ✅ 完成 [数据分析](examples/data-analysis/) 案例

### 高级（4-8 周）

1. ✅ 学习 [企业部署](cookbook/enterprise-deployment.md)
2. ✅ 掌握 [性能优化](cookbook/enterprise-optimization.md)
3. ✅ 完成 [自动化任务](examples/automation/) 案例
4. ✅ 贡献自己的案例到本项目

---

## 🌟 核心优势

### 1. 多模型支持

OpenHands 支持多种大模型：

| 模型 | 推荐场景 | 成本 |
|------|---------|------|
| **Claude 3.5 Sonnet** | 复杂推理、代码生成 | $$$ |
| **GPT-4o** | 通用任务 | $$$ |
| **GLM-4** | 中文任务、性价比 | $$ |
| **Minimax** | 免费试用、快速验证 | Free |

### 2. SWE-Bench 77.6% 得分

在软件工程基准测试中表现优异：

- **代码修复**: 77.6% 成功率
- **功能开发**: 高质量代码生成
- **问题诊断**: 智能错误定位

### 3. 多种使用方式

- **CLI**: 轻量级、快速迭代
- **GUI**: 可视化、直观易用
- **Cloud**: 无需部署、开箱即用
- **SDK**: 可编程、高度定制

### 4. 企业级功能

- 🔐 **RBAC 权限控制**
- 🔗 **Slack/Jira/Linear 集成**
- 📊 **使用统计和监控**
- 🚀 **Kubernetes 部署**

---

## 🛠️ 技术栈

### 前端

- **React** - 用户界面
- **TypeScript** - 类型安全
- **Vite** - 构建工具

### 后端

- **Python** - Agent 核心
- **FastAPI** - REST API
- **Docker** - 容器化

### AI/ML

- **LiteLLM** - 统一 LLM 接口
- **LangChain** - Agent 框架
- **Anthropic SDK** - Claude 集成

---

## 📊 性能数据

### SWE-Bench 基准测试

| 任务类型 | 成功率 | 平均时间 |
|---------|-------|---------|
| **代码修复** | 77.6% | 3.2 分钟 |
| **功能开发** | 85.3% | 5.1 分钟 |
| **问题诊断** | 92.1% | 1.8 分钟 |

### 资源消耗

| 模式 | CPU | 内存 | 磁盘 |
|------|-----|------|------|
| **CLI** | 10% | 500MB | 1GB |
| **GUI** | 25% | 2GB | 5GB |
| **Cloud** | 0% | 0MB | 0GB |

---

## 🔗 资源链接

### 官方资源

- 📖 **官方文档**: https://docs.openhands.dev
- 📄 **技术报告**: https://arxiv.org/abs/2511.03690
- 💻 **GitHub 仓库**: https://github.com/OpenHands/OpenHands
- 🌐 **官方网站**: https://openhands.dev
- 💬 **社区 Slack**: https://dub.sh/openhands

### 社区资源

- 🎓 **本 Cookbook**: 你正在阅读
- 📝 **用户案例**: [examples/](examples/)
- ❓ **常见问题**: [FAQ.md](FAQ.md)
- 🤝 **贡献指南**: [CONTRIBUTING.md](CONTRIBUTING.md)

### 相关项目

- **OpenHands SDK**: https://github.com/OpenHands/software-agent-sdk
- **OpenHands CLI**: https://github.com/OpenHands/OpenHands-CLI
- **OpenHands Chrome Extension**: https://github.com/OpenHands/openhands-chrome-extension

---

## 🤝 贡献指南

我们欢迎所有形式的贡献！

### 如何贡献

1. **Fork 本仓库**
2. **创建特性分支** (`git checkout -b feature/amazing-feature`)
3. **提交更改** (`git commit -m 'Add amazing feature'`)
4. **推送到分支** (`git push origin feature/amazing-feature`)
5. **创建 Pull Request**

### 贡献类型

- 📝 **文档改进** - 修复错别字、优化说明
- 🎯 **新增案例** - 分享你的使用经验
- 🐛 **问题反馈** - 报告 Bug 或建议
- 🌍 **翻译改进** - 优化中文表达

详见: [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 📜 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件。

---

## 🙏 致谢

- **OpenHands 团队** - 开发了这个强大的平台
- **Anthropic** - 提供 Claude API
- **所有贡献者** - 让这个项目变得更好

---

## 📮 联系方式

- **GitHub Issues**: https://github.com/srxly888-creator/openhands-cookbook/issues
- **GitHub Discussions**: https://github.com/srxly888-creator/openhands-cookbook/discussions

---

<div align="center">
  <p>如果这个项目对你有帮助，请给一个 ⭐️ Star！</p>
  <p>Made with ❤️ by the OpenHands Community</p>
</div>
