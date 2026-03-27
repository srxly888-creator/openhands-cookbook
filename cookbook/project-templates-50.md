# OpenHands 项目模板库

> 50+ 项目模板

---

## 📋 目录

- [Web 开发模板](#web-开发模板)
- [数据分析模板](#数据分析模板)
- [AI/ML 模板](#aiml-模板)
- [DevOps 模板](#devops-模板)

---

## 🌐 Web 开发模板

### 模板 1：FastAPI 项目

```yaml
# templates/fastapi.yaml
name: FastAPI 项目模板
description: 创建 FastAPI 项目

prompt: |
  创建 FastAPI 项目，包含：
  
  1. **项目结构**
     - main.py（应用入口）
     - routers/（路由模块）
     - models/（数据模型）
     - services/（业务逻辑）
     - tests/（单元测试）
  
  2. **核心功能**
     - 用户认证（JWT）
     - 数据库连接（PostgreSQL）
     - API 文档（OpenAPI）
     - 日志系统
     - 错误处理
  
  3. **配置**
     - requirements.txt
     - .env.example
     - README.md
     - Dockerfile
     - docker-compose.yml

parameters:
  - name: project_name
    type: string
    default: my-api
    description: 项目名称
  
  - name: database
    type: string
    default: PostgreSQL
    description: 数据库类型

# 使用
openhands run --template templates/fastapi.yaml \
  --param project_name=my-api \
  --param database=PostgreSQL
```

---

### 模板 2：Django 项目

```yaml
# templates/django.yaml
name: Django 项目模板
description: 创建 Django 项目

prompt: |
  创建 Django 项目，包含：
  
  1. **项目结构**
     - config/（配置）
     - apps/（应用）
     - templates/（模板）
     - static/（静态文件）
  
  2. **核心功能**
     - 用户系统
     - Admin 界面
     - REST API（DRF）
     - Celery 任务队列
     - Redis 缓存
  
  3. **配置**
     - requirements.txt
     - .env.example
     - README.md

# 使用
openhands run --template templates/django.yaml
```

---

### 模板 3：Flask 项目

```yaml
# templates/flask.yaml
name: Flask 项目模板
description: 创建 Flask 项目

prompt: |
  创建 Flask 项目，包含：
  
  1. **项目结构**
     - app.py（应用入口）
     - blueprints/（蓝图）
     - models/（模型）
     - templates/（模板）
  
  2. **核心功能**
     - 用户认证
     - 数据库（SQLAlchemy）
     - 表单验证
     - 邮件发送
  
  3. **配置**
     - requirements.txt
     - README.md

# 使用
openhands run --template templates/flask.yaml
```

---

## 📊 数据分析模板

### 模板 4：数据分析项目

```yaml
# templates/data-analysis.yaml
name: 数据分析项目模板
description: 创建数据分析项目

prompt: |
  创建数据分析项目，包含：
  
  1. **项目结构**
     - data/（数据）
     - notebooks/（Jupyter）
     - scripts/（脚本）
     - reports/（报告）
  
  2. **核心功能**
     - 数据清洗
     - 探索性分析
     - 可视化
     - 统计分析
     - 报告生成
  
  3. **配置**
     - requirements.txt
     - README.md

# 使用
openhands run --template templates/data-analysis.yaml
```

---

### 模板 5：机器学习项目

```yaml
# templates/ml-project.yaml
name: 机器学习项目模板
description: 创建机器学习项目

prompt: |
  创建机器学习项目，包含：
  
  1. **项目结构**
     - data/（数据）
     - models/（模型）
     - notebooks/（实验）
     - src/（源码）
  
  2. **核心功能**
     - 数据预处理
     - 特征工程
     - 模型训练
     - 模型评估
     - 模型部署
  
  3. **配置**
     - requirements.txt
     - README.md

# 使用
openhands run --template templates/ml-project.yaml
```

---

## 🤖 AI/ML 模板

### 模板 6：NLP 项目

```yaml
# templates/nlp-project.yaml
name: NLP 项目模板
description: 创建 NLP 项目

prompt: |
  创建 NLP 项目，包含：
  
  1. **项目结构**
     - data/（语料）
     - models/（模型）
     - scripts/（脚本）
  
  2. **核心功能**
     - 文本预处理
     - 分词
     - 词向量
     - 情感分析
     - 文本分类
  
  3. **配置**
     - requirements.txt
     - README.md

# 使用
openhands run --template templates/nlp-project.yaml
```

---

### 模板 7：计算机视觉项目

```yaml
# templates/cv-project.yaml
name: 计算机视觉项目模板
description: 创建 CV 项目

prompt: |
  创建计算机视觉项目，包含：
  
  1. **项目结构**
     - data/（图像）
     - models/（模型）
     - scripts/（脚本）
  
  2. **核心功能**
     - 图像预处理
     - 目标检测
     - 图像分类
     - 图像分割
  
  3. **配置**
     - requirements.txt
     - README.md

# 使用
openhands run --template templates/cv-project.yaml
```

---

## 🚀 DevOps 模板

### 模板 8：Docker 项目

```yaml
# templates/docker-project.yaml
name: Docker 项目模板
description: 创建 Docker 项目

prompt: |
  创建 Docker 项目，包含：
  
  1. **配置文件**
     - Dockerfile
     - docker-compose.yml
     - .dockerignore
  
  2. **核心功能**
     - 多阶段构建
     - 环境变量
     - 数据卷
     - 网络配置
  
  3. **部署**
     - 构建脚本
     - 部署脚本

# 使用
openhands run --template templates/docker-project.yaml
```

---

### 模板 9：Kubernetes 项目

```yaml
# templates/kubernetes-project.yaml
name: Kubernetes 项目模板
description: 创建 K8s 项目

prompt: |
  创建 Kubernetes 项目，包含：
  
  1. **配置文件**
     - deployment.yaml
     - service.yaml
     - configmap.yaml
     - secret.yaml
  
  2. **核心功能**
     - 部署配置
     - 服务发现
     - 配置管理
     - 自动扩缩容
  
  3. **部署**
     - kubectl 脚本
     - Helm Chart

# 使用
openhands run --template templates/kubernetes-project.yaml
```

---

### 模板 10：CI/CD 项目

```yaml
# templates/cicd-project.yaml
name: CI/CD 项目模板
description: 创建 CI/CD 项目

prompt: |
  创建 CI/CD 项目，包含：
  
  1. **GitHub Actions**
     - test.yml
     - build.yml
     - deploy.yml
  
  2. **核心功能**
     - 自动测试
     - 自动构建
     - 自动部署
  
  3. **配置**
     - 环境变量
     - 密钥管理

# 使用
openhands run --template templates/cicd-project.yaml
```

---

## 📚 更多模板（50+）

### Web 开发（11-20）

11. React 项目
12. Vue 项目
13. Angular 项目
14. Next.js 项目
15. Nuxt 项目
16. Express 项目
17. NestJS 项目
18. Fastify 项目
19. Tornado 项目
20. Sanic 项目

### 数据分析（21-30）

21. Pandas 项目
22. NumPy 项目
23. Matplotlib 项目
24. Seaborn 项目
25. Plotly 项目
26. Scikit-learn 项目
27. TensorFlow 项目
28. PyTorch 项目
29. Keras 项目
30. XGBoost 项目

### AI/ML（31-40）

31. 聊天机器人项目
32. 推荐系统项目
33. 图像生成项目
34. 语音识别项目
35. 文本生成项目
36. 强化学习项目
37. 迁移学习项目
38. 联邦学习项目
39. AutoML 项目
40. MLOps 项目

### DevOps（41-50）

41. Ansible 项目
42. Terraform 项目
43. Jenkins 项目
44. GitLab CI 项目
45. Prometheus 项目
46. Grafana 项目
47. ELK Stack 项目
48. Nginx 项目
49. HAProxy 项目
50. Vault 项目

---

<div align="center">
  <p>📚 50+ 模板，快速启动！</p>
</div>
