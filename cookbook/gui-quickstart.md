# OpenHands GUI 模式详细指南

> 本地可视化界面，直观易用的操作体验

---

## 📋 目录

- [安装部署](#安装部署)
- [基础使用](#基础使用)
- [高级功能](#高级功能)
- [故障排查](#故障排查)

---

## 🔧 安装部署

### 系统要求

- Docker Desktop
- 8GB+ RAM
- 10GB+ 磁盘空间
- Python 3.8+（可选）

### 快速部署

#### 方式一：Docker Compose（推荐）

```bash
# 克隆仓库
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 访问界面
open http://localhost:3000
```

#### 方式二：Make 命令

```bash
# 构建镜像
make build

# 启动服务
make run

# 停止服务
make stop
```

### 配置 API Key

1. 访问 http://localhost:3000
2. 点击右上角 ⚙️ 设置图标
3. 选择 "API Keys"
4. 输入你的 API Key
5. 选择默认模型

---

## 🚀 基础使用

### 1. 创建新会话

1. 点击 "New Conversation"
2. 输入任务描述
3. 选择工作目录（可选）
4. 点击 "Start"

### 2. 与 Agent 交互

**对话界面**：
- 左侧：对话历史
- 右侧：Agent 操作日志
- 底部：输入框

**示例对话**：
```
You: 帮我创建一个 FastAPI 项目，包含用户认证

Agent: 好的，我将创建一个 FastAPI 项目。
[开始创建目录结构...]
[创建文件：main.py, requirements.txt...]
[配置 JWT 认证...]

✅ 项目创建完成！
```

### 3. 查看操作历史

**Actions 标签页**：
- 显示所有操作步骤
- 包含命令执行结果
- 文件修改记录

**Terminal 标签页**：
- 实时终端输出
- 命令执行日志
- 错误信息

### 4. 文件管理

**Files 标签页**：
- 浏览项目文件
- 查看文件内容
- 编辑文件
- 上传文件

---

## 🎯 高级功能

### 1. 多项目管理

```bash
# 切换项目
Settings → Workspaces → Add Workspace

# 配置多个工作目录
/workspace1
/workspace2
/workspace3
```

### 2. 模型切换

```bash
# 在对话中切换模型
Settings → Model → Select Model

# 可用模型：
- Claude 3.5 Sonnet
- GPT-4o
- GLM-4
- Minimax
```

### 3. 工作流保存

```bash
# 保存会话
File → Save Conversation

# 导出会话
File → Export → JSON/Markdown

# 导入会话
File → Import → Select File
```

### 4. 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd + Enter` | 发送消息 |
| `Cmd + N` | 新建会话 |
| `Cmd + S` | 保存会话 |
| `Cmd + /` | 显示帮助 |
| `Esc` | 取消操作 |

---

## 🔧 故障排查

### 问题 1：无法启动服务

**错误信息**：
```
Error: Port 3000 already in use
```

**解决方案**：
```bash
# 查看端口占用
lsof -i :3000

# 停止占用进程
kill -9 <PID>

# 或使用其他端口
docker-compose.yml → ports: "3001:3000"
```

### 问题 2：Docker 镜像构建失败

**错误信息**：
```
Error: failed to build Docker image
```

**解决方案**：
```bash
# 清理 Docker 缓存
docker system prune -a

# 重新构建
make build

# 或使用预构建镜像
docker pull openhands/openhands:latest
```

### 问题 3：API Key 无效

**解决方案**：
1. 检查 API Key 格式
2. 确认 API Key 有效
3. 重启服务

### 问题 4：内存不足

**解决方案**：
```bash
# 增加 Docker 内存限制
Docker Desktop → Settings → Resources → Memory: 8GB

# 或在 docker-compose.yml 中配置
services:
  openhands:
    deploy:
      resources:
        limits:
          memory: 8G
```

---

## 📊 性能优化

### 1. 资源配置

```yaml
# docker-compose.yml
services:
  openhands:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          memory: 4G
```

### 2. 缓存配置

```bash
# 启用 Docker 缓存
DOCKER_BUILDKIT=1 docker-compose build

# 清理缓存
docker builder prune
```

### 3. 网络优化

```yaml
# 使用本地网络
networks:
  openhands-network:
    driver: bridge
```

---

## 🎓 最佳实践

### 1. 项目组织

```
workspace/
├── project1/
│   ├── src/
│   ├── tests/
│   └── README.md
├── project2/
└── shared/
```

### 2. 定期备份

```bash
# 导出所有会话
File → Export All → backup.tar.gz

# 恢复备份
File → Import → backup.tar.gz
```

### 3. 监控资源

```bash
# 查看 Docker 资源使用
docker stats

# 查看日志
docker-compose logs -f --tail=100
```

---

## 📚 相关资源

- [CLI 模式指南](cli-quickstart.md)
- [Cloud 模式指南](cloud-quickstart.md)
- [故障排查](../FAQ.md)

---

<div align="center">
  <p>🎉 掌握 GUI 模式，提升开发效率！</p>
</div>
