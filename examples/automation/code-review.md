# 自动化案例集

> 使用 OpenHands 实现自动化工作流

---

## 📋 案例列表

- [案例 1：自动化代码审查](#案例-1自动化代码审查)
- [案例 2：自动化测试生成](#案例-2自动化测试生成)
- [案例 3：自动化部署](#案例-3自动化部署)
- [案例 4：自动化文档生成](#案例-4自动化文档生成)
- [案例 5：自动化依赖更新](#案例-5自动化依赖更新)

---

## 案例 1：自动化代码审查

### 任务描述

自动审查 Pull Request 中的代码，提供改进建议。

### 实现方案

**Python 脚本**：
```python
import os
from openhands import Agent
from github import Github

def auto_code_review(repo_name, pr_number):
    """自动化代码审查"""
    
    # 初始化 GitHub
    g = Github(os.getenv("GITHUB_TOKEN"))
    repo = g.get_repo(repo_name)
    pr = repo.get_pull(pr_number)
    
    # 获取变更文件
    files = pr.get_files()
    
    # 初始化 Agent
    agent = Agent(model="claude-3.5-sonnet")
    
    reviews = []
    for file in files:
        if file.filename.endswith('.py'):
            # 审查每个文件
            result = agent.run(f"""
            审查以下代码并提供改进建议：

            文件：{file.filename}
            变更：
            {file.patch}

            审查要点：
            1. 代码质量
            2. 性能问题
            3. 安全风险
            4. 最佳实践
            """)
            
            reviews.append({
                'file': file.filename,
                'review': result.output
            })
    
    # 发布审查评论
    comment = "## 🤖 自动代码审查\n\n"
    for review in reviews:
        comment += f"### {review['file']}\n\n{review['review']}\n\n"
    
    pr.create_issue_comment(comment)
    
    return reviews

# 使用
auto_code_review("owner/repo", 123)
```

**GitHub Actions 配置**：
```yaml
name: Auto Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Code Review
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENHANDS_API_KEY: ${{ secrets.OPENHANDS_API_KEY }}
        run: |
          pip install openhands PyGithub
          python scripts/auto_review.py
```

### 学习要点

- ✅ GitHub API 使用
- ✅ 自动化工作流
- ✅ 代码质量检查
- ✅ CI/CD 集成

---

## 案例 2：自动化测试生成

### 任务描述

为现有代码自动生成单元测试。

### 实现方案

**Python 脚本**：
```python
import os
from openhands import Agent

def generate_tests(source_dir, test_dir):
    """自动生成测试"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    # 获取所有 Python 文件
    for root, dirs, files in os.walk(source_dir):
        for file in files:
            if file.endswith('.py') and not file.startswith('__'):
                source_file = os.path.join(root, file)
                
                # 读取源代码
                with open(source_file, 'r') as f:
                    code = f.read()
                
                # 生成测试
                result = agent.run(f"""
                为以下代码生成完整的单元测试：

                文件：{file}
                代码：
                {code}

                要求：
                1. 测试覆盖率 > 90%
                2. 包含正常情况测试
                3. 包含边界条件测试
                4. 包含异常情况测试
                5. 使用 pytest
                """)
                
                if result.success:
                    # 保存测试文件
                    test_file = os.path.join(
                        test_dir,
                        file.replace('.py', '_test.py')
                    )
                    
                    with open(test_file, 'w') as f:
                        f.write(result.output)
                    
                    print(f"✅ 生成测试: {test_file}")

# 使用
generate_tests("./src", "./tests")
```

**配置文件**：
```python
# test_config.yaml
source_dir: "./src"
test_dir: "./tests"
model: "claude-3.5-sonnet"
coverage_threshold: 90
```

### 学习要点

- ✅ 测试驱动开发
- ✅ 自动化测试生成
- ✅ 代码覆盖率
- ✅ pytest 使用

---

## 案例 3：自动化部署

### 任务描述

自动化部署应用到生产环境。

### 实现方案

**部署脚本**：
```python
from openhands import Agent
import subprocess

def auto_deploy(app_name, environment):
    """自动化部署"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 拉取最新代码
    agent.execute_command("git pull origin main")
    
    # 2. 安装依赖
    agent.execute_command("pip install -r requirements.txt")
    
    # 3. 运行测试
    test_result = agent.execute_command("pytest tests/")
    
    if "passed" not in test_result:
        print("❌ 测试失败，停止部署")
        return False
    
    # 4. 构建镜像
    agent.execute_command(f"docker build -t {app_name}:{environment} .")
    
    # 5. 推送镜像
    agent.execute_command(f"docker push registry/{app_name}:{environment}")
    
    # 6. 更新 Kubernetes
    agent.execute_command(f"""
    kubectl set image deployment/{app_name} \\
        {app_name}=registry/{app_name}:{environment} \\
        --namespace {environment}
    """)
    
    # 7. 验证部署
    result = agent.execute_command(f"""
    kubectl rollout status deployment/{app_name} \\
        --namespace {environment}
    """)
    
    if "successfully" in result:
        print(f"✅ {app_name} 部署成功！")
        return True
    else:
        print(f"❌ {app_name} 部署失败")
        return False

# 使用
auto_deploy("my-api", "production")
```

**CI/CD 配置**：
```yaml
# .github/workflows/deploy.yml
name: Auto Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Production
        env:
          OPENHANDS_API_KEY: ${{ secrets.OPENHANDS_API_KEY }}
        run: |
          pip install openhands
          python scripts/deploy.py
```

### 学习要点

- ✅ CI/CD 流水线
- ✅ Docker 部署
- ✅ Kubernetes 更新
- ✅ 自动化验证

---

## 案例 4：自动化文档生成

### 任务描述

自动生成 API 文档和 README。

### 实现方案

**文档生成脚本**：
```python
from openhands import Agent
import os

def generate_docs(project_dir):
    """自动生成文档"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 生成 API 文档
    result = agent.run(f"""
    为项目生成 API 文档：

    项目目录：{project_dir}

    要求：
    1. 分析所有 API 路由
    2. 生成 OpenAPI 格式文档
    3. 包含请求/响应示例
    4. 包含错误码说明
    """)
    
    if result.success:
        with open(f"{project_dir}/docs/api.md", 'w') as f:
            f.write(result.output)
        print("✅ API 文档已生成")
    
    # 2. 生成 README
    result = agent.run(f"""
    为项目生成 README.md：

    项目目录：{project_dir}

    包含：
    1. 项目简介
    2. 安装说明
    3. 使用示例
    4. API 文档链接
    5. 贡献指南
    """)
    
    if result.success:
        with open(f"{project_dir}/README.md", 'w') as f:
            f.write(result.output)
        print("✅ README 已生成")

# 使用
generate_docs("./my-project")
```

### 学习要点

- ✅ 自动文档生成
- ✅ OpenAPI 规范
- ✅ Markdown 编写
- ✅ 项目文档管理

---

## 案例 5：自动化依赖更新

### 任务描述

自动检查并更新项目依赖。

### 实现方案

**依赖更新脚本**：
```python
from openhands import Agent
import subprocess

def auto_update_dependencies():
    """自动化依赖更新"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 检查过时的依赖
    result = agent.execute_command("pip list --outdated")
    
    # 2. 分析每个依赖
    lines = result.split('\n')[2:]  # 跳过表头
    updates = []
    
    for line in lines:
        if line.strip():
            parts = line.split()
            package = parts[0]
            current = parts[1]
            latest = parts[2]
            
            # 检查兼容性
            check_result = agent.run(f"""
            检查依赖更新兼容性：

            包：{package}
            当前版本：{current}
            最新版本：{latest}

            分析：
            1. 是否有破坏性变更
            2. 是否需要代码修改
            3. 推荐的更新方式
            """)
            
            updates.append({
                'package': package,
                'current': current,
                'latest': latest,
                'analysis': check_result.output
            })
    
    # 3. 生成报告
    report = "# 依赖更新报告\n\n"
    for update in updates:
        report += f"## {update['package']}\n"
        report += f"- 当前版本：{update['current']}\n"
        report += f"- 最新版本：{update['latest']}\n"
        report += f"- 分析：\n{update['analysis']}\n\n"
    
    with open("dependency_update_report.md", 'w') as f:
        f.write(report)
    
    print("✅ 依赖更新报告已生成")

# 使用
auto_update_dependencies()
```

### 学习要点

- ✅ 依赖管理
- ✅ 版本兼容性
- ✅ 自动化检查
- ✅ 风险评估

---

## 📚 相关资源

- [CI/CD 最佳实践](https://docs.openhands.dev/best-practices/ci-cd)
- [自动化工作流](https://docs.openhands.dev/workflows/automation)
- [GitHub Actions 集成](https://docs.openhands.dev/integrations/github-actions)

---

<div align="center">
  <p>🎉 自动化你的工作流，提升效率！</p>
</div>
