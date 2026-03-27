# 常见问题（FAQ）

> OpenHands 使用中的常见问题和解决方案

---

## 📋 目录

- [基础问题](#基础问题)
- [安装配置](#安装配置)
- [使用问题](#使用问题)
- [性能优化](#性能优化)
- [故障排查](#故障排查)
- [企业部署](#企业部署)

---

## 基础问题

### Q1: OpenHands 是什么？

**A**: OpenHands 是一个 AI 驱动的开发平台，提供：
- **CLI 工具** - 命令行界面
- **GUI 界面** - 可视化操作
- **Cloud 服务** - 云端托管
- **SDK** - 可编程 API

---

### Q2: OpenHands 支持哪些编程语言？

**A**: 支持所有主流语言：
- **Python** - 推荐 ⭐⭐⭐⭐⭐
- **JavaScript/TypeScript** - ⭐⭐⭐⭐⭐
- **Java** - ⭐⭐⭐⭐
- **Go** - ⭐⭐⭐⭐
- **Rust** - ⭐⭐⭐⭐
- **其他** - ⭐⭐⭐

---

### Q3: OpenHands 和 GitHub Copilot 有什么区别？

**A**:

| 特性 | OpenHands | GitHub Copilot |
|------|-----------|----------------|
| **交互方式** | 对话式 | 代码补全 |
| **任务类型** | 复杂任务 | 代码片段 |
| **自主性** | 高 | 中 |
| **文件操作** | ✅ | ❌ |
| **终端访问** | ✅ | ❌ |

---

### Q4: OpenHands 收费吗？

**A**:
- **CLI/GUI 模式** - 免费（需要自己的 API Key）
- **Cloud 模式** - 免费试用（Minimax 模型）
- **Enterprise** - 付费（自托管 + 支持）

---

## 安装配置

### Q5: 如何安装 OpenHands？

**A**:

**CLI 模式**：
```bash
pip install openhands
```

**GUI 模式**：
```bash
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands
make build
make run
```

**Cloud 模式**：
访问 https://app.all-hands.dev

---

### Q6: 如何配置 API Key？

**A**:

**方式一：环境变量**
```bash
export ANTHROPIC_API_KEY="sk-ant-xxx"
```

**方式二：配置文件**
```bash
echo "ANTHROPIC_API_KEY=sk-ant-xxx" > ~/.openhands.env
```

**方式三：GUI 设置**
访问 http://localhost:3000 → Settings → API Keys

---

### Q7: 支持哪些 LLM 模型？

**A**:

| 模型 | 推荐场景 | 成本 |
|------|---------|------|
| **Claude 3.5 Sonnet** | 复杂任务 | $$$ |
| **GPT-4o** | 通用任务 | $$$ |
| **GLM-4** | 中文任务 | $$ |
| **Minimax** | 免费试用 | Free |
| **Qwen** | 中文任务 | $$ |

---

## 使用问题

### Q8: 如何描述任务效果最好？

**A**:

❌ **不好的描述**：
```
优化代码
```

✅ **好的描述**：
```
优化 main.py 中的数据处理函数：
1. 减少内存占用（目标：<100MB）
2. 提高处理速度（目标：<1秒）
3. 保持代码可读性
4. 添加单元测试
```

---

### Q9: OpenHands 能做什么？

**A**:

**✅ 擅长的任务**：
- 创建新项目
- 编写代码
- 修复 Bug
- 重构代码
- 编写测试
- 生成文档

**⚠️ 需要监督的任务**：
- 删除文件
- 修改配置
- 部署到生产环境

**❌ 不擅长的任务**：
- 需要人工判断的决策
- 需要本地环境的操作
- 需要实时交互的任务

---

### Q10: 如何查看 Agent 的操作？

**A**:

**CLI 模式**：
```bash
openhands logs --tail 100
```

**GUI 模式**：
界面右侧 → Actions 标签页

**Cloud 模式**：
界面底部 → Activity Log

---

## 性能优化

### Q11: 如何提高响应速度？

**A**:

1. **选择更快的模型**
   ```bash
   openhands run --model gpt-3.5-turbo "任务"
   ```

2. **减少任务复杂度**
   - 分步骤执行
   - 明确具体要求

3. **优化提示词**
   - 简洁明了
   - 避免歧义

---

### Q12: 如何减少 Token 消耗？

**A**:

1. **使用更小的模型**
   ```bash
   openhands run --model gpt-3.5-turbo
   ```

2. **限制上下文大小**
   ```bash
   openhands run --max-tokens 2048
   ```

3. **分批次处理**
   - 不要一次性处理太多文件
   - 分步骤执行任务

---

### Q13: 如何处理大型项目？

**A**:

1. **使用 Git 忽略**
   ```bash
   # .openhandsignore
   node_modules/
   venv/
   __pycache__/
   ```

2. **分模块处理**
   ```bash
   openhands run --directory ./module1 "任务"
   openhands run --directory ./module2 "任务"
   ```

3. **选择性处理**
   ```bash
   openhands run "只处理 src/ 目录下的 Python 文件"
   ```

---

## 故障排查

### Q14: API Key 无效怎么办？

**A**:

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

# 验证格式
# Claude: sk-ant-xxx
# OpenAI: sk-xxx
# GLM: xxx.xxx
```

---

### Q15: 网络连接失败怎么办？

**A**:

**错误信息**：
```
Error: Connection timeout
```

**解决方案**：
```bash
# 使用代理
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"

# 或使用国内镜像
export OPENAI_BASE_URL="https://api.openai-proxy.com/v1"
```

---

### Q16: 内存不足怎么办？

**A**:

**错误信息**：
```
Error: Out of memory
```

**解决方案**：
```bash
# 减少并发数
openhands run --max-workers 1

# 使用更小的模型
openhands run --model gpt-3.5-turbo

# 增加系统内存
# 或关闭其他应用
```

---

### Q17: 任务执行失败怎么办？

**A**:

**步骤**：

1. **查看日志**
   ```bash
   openhands logs --tail 100
   ```

2. **重试任务**
   ```bash
   openhands run --retry "任务描述"
   ```

3. **简化任务**
   - 分步骤执行
   - 提供更多上下文

4. **寻求帮助**
   - 创建 Issue
   - 加入社区 Slack

---

## 企业部署

### Q18: 如何自托管 OpenHands？

**A**:

**Kubernetes 部署**：
```bash
# 使用 Helm
helm install openhands ./helm-chart

# 或使用 YAML
kubectl apply -f k8s/
```

**Docker 部署**：
```bash
docker run -d \
  -p 3000:3000 \
  -e ANTHROPIC_API_KEY=xxx \
  openhands/openhands:latest
```

---

### Q19: 如何配置企业级安全？

**A**:

1. **RBAC 权限控制**
   - 配置角色和权限
   - 限制资源访问

2. **网络安全**
   - 使用 VPC
   - 配置防火墙规则

3. **数据加密**
   - 启用 TLS
   - 加密敏感数据

4. **审计日志**
   - 记录所有操作
   - 定期审计

---

### Q20: 如何监控 OpenHands？

**A**:

1. **Prometheus + Grafana**
   - 监控指标
   - 可视化仪表板

2. **日志收集**
   - ELK Stack
   - Fluentd

3. **告警配置**
   - Prometheus AlertManager
   - PagerDuty

---

## 🤝 更多问题？

### 没找到答案？

1. **搜索 Issues**
   https://github.com/srxly888-creator/openhands-cookbook/issues

2. **创建新 Issue**
   - 详细描述问题
   - 提供复现步骤
   - 附上错误日志

3. **加入社区**
   - Slack: https://dub.sh/openhands
   - GitHub Discussions

---

<div align="center">
  <p>❓ 还有问题？欢迎提问！</p>
</div>
