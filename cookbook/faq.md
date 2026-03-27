# OpenHands 常见问题 FAQ

> 100+ 常见问题及解决方案

---

## 📋 目录

- [安装问题](#安装问题)
- [配置问题](#配置问题)
- [使用问题](#使用问题)
- [性能问题](#性能问题)

---

## 📦 安装问题

### Q1: pip 安装失败怎么办？

**问题描述**:
```
ERROR: Could not find a version that satisfies the requirement openhands
```

**解决方案**:
```bash
# 方式一：升级 pip
pip install --upgrade pip

# 方式二：使用国内镜像
pip install openhands -i https://pypi.tuna.tsinghua.edu.cn/simple

# 方式三：从源码安装
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands
pip install -e .
```

---

### Q2: 依赖冲突怎么解决？

**问题描述**:
```
ERROR: Cannot install openhands because these package versions have conflicting dependencies
```

**解决方案**:
```bash
# 方式一：创建新的虚拟环境
python -m venv openhands-env
source openhands-env/bin/activate  # Linux/Mac
openhands-env\Scripts\activate  # Windows
pip install openhands

# 方式二：使用 conda
conda create -n openhands python=3.11
conda activate openhands
pip install openhands

# 方式三：强制重装
pip install --force-reinstall openhands
```

---

### Q3: Docker 安装失败？

**问题描述**:
```
docker: Error response from daemon: pull access denied
```

**解决方案**:
```bash
# 方式一：登录 Docker Hub
docker login

# 方式二：使用镜像加速
# 编辑 /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}

# 重启 Docker
sudo systemctl restart docker

# 方式三：手动拉取
docker pull openhands/openhands:latest
```

---

## ⚙️ 配置问题

### Q4: API Key 配置错误？

**问题描述**:
```
Error: Invalid API key
```

**解决方案**:
```bash
# 方式一：环境变量
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 方式二：配置文件
openhands config set api-key sk-ant-xxx

# 方式三：.env 文件
echo "ANTHROPIC_API_KEY=sk-ant-xxx" > .env

# 验证
openhands config show
```

---

### Q5: 模型不可用？

**问题描述**:
```
Error: Model claude-3.5-sonnet not found
```

**解决方案**:
```bash
# 方式一：检查可用模型
openhands models list

# 方式二：使用正确的模型名称
openhands run --model claude-3.5-sonnet "任务"

# 方式三：使用别名
openhands run --model claude "任务"

# 方式四：切换到其他模型
openhands run --model gpt-4o "任务"
```

---

### Q6: 数据库连接失败？

**问题描述**:
```
Error: Could not connect to database
```

**解决方案**:
```bash
# 方式一：检查数据库服务
# PostgreSQL
sudo systemctl status postgresql

# MySQL
sudo systemctl status mysql

# 方式二：验证连接字符串
export DATABASE_URL="postgresql://user:pass@localhost:5432/db"

# 方式三：测试连接
psql -h localhost -U user -d db

# 方式四：使用 Docker
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=pass postgres:15
```

---

## 🚀 使用问题

### Q7: 任务超时怎么办？

**问题描述**:
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

# 方式四：启用缓存
openhands run --cache "任务"
```

---

### Q8: 内存不足怎么办？

**问题描述**:
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

# 方式四：增加内存
# Docker 方式
docker run --memory=4g openhands/openhands

# Kubernetes 方式
kubectl set resources deployment/openhands \
  --limits=memory=4Gi \
  --requests=memory=2Gi
```

---

### Q9: 网络错误怎么办？

**问题描述**:
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

# 方式四：检查网络
ping api.anthropic.com
curl -I https://api.anthropic.com
```

---

### Q10: 权限错误怎么办？

**问题描述**:
```
Error: Permission denied
```

**解决方案**:
```bash
# 方式一：使用 sudo
sudo openhands run "任务"

# 方式二：修改文件权限
chmod 755 /path/to/file

# 方式三：修改所有者
chown -R $USER:$USER ~/.openhands

# 方式四：使用 Docker
docker run --user $(id -u):$(id -g) openhands/openhands
```

---

## ⚡ 性能问题

### Q11: 响应慢怎么办？

**问题描述**:
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

# 4. 使用更快的模型
agent = Agent(model="gpt-4o")
```

---

### Q12: CPU 使用率高怎么办？

**问题描述**:
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

# 4. 使用 Docker 资源限制
# docker run --cpus=2 openhands/openhands
```

---

### Q13: 内存泄漏怎么办？

**问题描述**:
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

# 3. 手动垃圾回收
import gc
gc.collect()
```

---

## 🔧 故障排查

### Q14: 如何启用调试模式？

**解决方案**:
```bash
# 方式一：命令行
openhands run --debug "任务"

# 方式二：环境变量
export OPENHANDS_DEBUG=true
openhands run "任务"

# 方式三：配置文件
openhands config set debug true

# 查看日志
tail -f ~/.openhands/logs/openhands.log
```

---

### Q15: 如何查看详细日志？

**解决方案**:
```bash
# 方式一：查看日志文件
tail -f ~/.openhands/logs/openhands.log

# 方式二：查看最近 100 行
tail -n 100 ~/.openhawls/logs/openhands.log

# 方式三：搜索关键词
grep "ERROR" ~/.openhands/logs/openhands.log

# 方式四：实时查看
openhands logs --tail 100 -f
```

---

### Q16: 如何重置配置？

**解决方案**:
```bash
# 方式一：重置所有配置
openhands config reset

# 方式二：删除配置文件
rm ~/.openhands/config.yaml

# 方式三：重新初始化
openhands init

# 方式四：清除缓存
openhands cache clear
```

---

## 🌐 部署问题

### Q17: Docker 部署失败？

**问题描述**:
```
docker: Error response from daemon
```

**解决方案**:
```bash
# 方式一：检查 Docker 状态
sudo systemctl status docker

# 方式二：重启 Docker
sudo systemctl restart docker

# 方式三：检查镜像
docker images | grep openhands

# 方式四：重新拉取
docker pull openhands/openhands:latest

# 方式五：查看日志
docker logs openhands-container
```

---

### Q18: Kubernetes 部署失败？

**问题描述**:
```
Error from server (BadRequest)
```

**解决方案**:
```bash
# 方式一：检查 Pod 状态
kubectl get pods -n openhands

# 方式二：查看 Pod 日志
kubectl logs -f deployment/openhands -n openhands

# 方式三：检查事件
kubectl get events -n openhands --sort-by='.lastTimestamp'

# 方式四：描述 Pod
kubectl describe pod openhands-pod -n openhands

# 方式五：重启部署
kubectl rollout restart deployment/openhands -n openhands
```

---

### Q19: 云部署失败？

**问题描述**:
```
Error: Failed to deploy to cloud
```

**解决方案**:
```bash
# AWS
# 方式一：检查权限
aws iam get-user

# 方式二：检查资源
aws ec2 describe-instances

# 方式三：查看日志
aws logs tail /aws/lambda/openhands --follow

# GCP
# 方式一：检查权限
gcloud auth list

# 方式二：查看日志
gcloud logging read "resource.type=gae_app"
```

---

### Q20: 负载均衡配置错误？

**问题描述**:
```
Error: 502 Bad Gateway
```

**解决方案**:
```bash
# Nginx
# 方式一：检查配置
sudo nginx -t

# 方式二：重启 Nginx
sudo systemctl restart nginx

# 方式三：查看日志
tail -f /var/log/nginx/error.log

# AWS ALB
# 方式一：检查目标组
aws elbv2 describe-target-groups

# 方式二：检查健康检查
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:...
```

---

## 🔒 安全问题

### Q21: 如何防止 API Key 泄露？

**解决方案**:
```bash
# 方式一：使用环境变量
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 方式二：使用 .env 文件（不提交到 Git）
echo ".env" >> .gitignore

# 方式三：使用密钥管理服务
# AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id openhands/api-key

# HashiCorp Vault
vault kv get secret/openhands/api-key

# 方式四：定期轮换密钥
# 每 90 天更换一次
```

---

### Q22: 如何防止 SQL 注入？

**解决方案**:
```python
# ✅ 好的实践：参数化查询
from sqlalchemy import text

query = text("SELECT * FROM users WHERE id = :user_id")
result = session.execute(query, {"user_id": user_id})

# ❌ 不好的实践：字符串拼接
query = f"SELECT * FROM users WHERE id = '{user_id}'"  # 危险！
```

---

### Q23: 如何防止 XSS 攻击？

**解决方案**:
```python
from markupsafe import escape

# ✅ 好的实践：转义用户输入
user_input = "<script>alert('XSS')</script>"
safe_input = escape(user_input)
# 输出: &lt;script&gt;alert('XSS')&lt;/script&gt;

# ❌ 不好的实践：直接输出
print(user_input)  # 危险！
```

---

## 📚 更多问题

### Q24: 如何获取帮助？

**解决方案**:
- **官方文档**: https://docs.openhands.dev
- **GitHub Issues**: https://github.com/OpenHands/OpenHands/issues
- **Discord**: https://discord.gg/openhands
- **Stack Overflow**: [openhands] 标签
- **中文社区**: https://github.com/srxly888-creator/openhands-cookbook

---

### Q25: 如何贡献代码？

**解决方案**:
```bash
# 1. Fork 仓库
git clone https://github.com/YOUR_USERNAME/OpenHands.git

# 2. 创建分支
git checkout -b feature/your-feature

# 3. 提交代码
git add .
git commit -m "feat: 添加新功能"

# 4. 推送
git push origin feature/your-feature

# 5. 创建 Pull Request
# 在 GitHub 上创建 PR
```

---

<div align="center">
  <p>❓ 有问题？先查 FAQ！</p>
</div>
