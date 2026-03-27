# OpenHands 工具集成指南

> 集成常用开发工具

---

## 📋 目录

- [Git 集成](#git-集成)
- [Docker 集成](#docker-集成)
- [CI/CD 集成](#cicd-集成)
- [IDE 集成](#ide-集成)

---

## 🔧 Git 集成

### 自动提交

```python
# integration/git.py
from openhands import Agent
import subprocess

class GitIntegration:
    def __init__(self):
        self.agent = Agent(model="claude-3.5-sonnet")
    
    def auto_commit(self, message: str):
        """自动提交"""
        # 1. 执行任务
        result = self.agent.run(message)
        
        # 2. 检查变更
        changes = subprocess.check_output(
            ["git", "status", "--short"]
        ).decode()
        
        if changes:
            # 3. 提交变更
            subprocess.run(["git", "add", "-A"])
            subprocess.run(["git", "commit", "-m", message])
            
            return True
        
        return False
    
    def create_pr(self, title: str, body: str):
        """创建 Pull Request"""
        # 1. 创建分支
        branch_name = f"feature/{title.lower().replace(' ', '-')}"
        subprocess.run(["git", "checkout", "-b", branch_name])
        
        # 2. 提交变更
        subprocess.run(["git", "add", "-A"])
        subprocess.run(["git", "commit", "-m", title])
        
        # 3. 推送分支
        subprocess.run(["git", "push", "-u", "origin", branch_name])
        
        # 4. 创建 PR
        subprocess.run([
            "gh", "pr", "create",
            "--title", title,
            "--body", body
        ])

# 使用示例
git = GitIntegration()

# 自动提交
git.auto_commit("feat: 添加新功能")

# 创建 PR
git.create_pr(
    title="添加用户认证功能",
    body="实现基于 JWT 的用户认证系统"
)
```

---

## 🐳 Docker 集成

### 自动构建

```python
# integration/docker.py
from openhands import Agent
import subprocess

class DockerIntegration:
    def __init__(self):
        self.agent = Agent(model="claude-3.5-sonnet")
    
    def build_image(self, name: str, tag: str = "latest"):
        """构建 Docker 镜像"""
        # 1. 生成 Dockerfile
        dockerfile = self.agent.run(f"生成 {name} 的 Dockerfile")
        
        # 2. 保存 Dockerfile
        with open("Dockerfile", "w") as f:
            f.write(dockerfile.output)
        
        # 3. 构建镜像
        image_name = f"{name}:{tag}"
        subprocess.run([
            "docker", "build",
            "-t", image_name,
            "."
        ])
        
        return image_name
    
    def run_container(self, image: str, ports: dict):
        """运行容器"""
        port_args = []
        for host_port, container_port in ports.items():
            port_args.extend(["-p", f"{host_port}:{container_port}"])
        
        subprocess.run([
            "docker", "run",
            "-d",
            *port_args,
            image
        ])
    
    def push_image(self, image: str, registry: str):
        """推送镜像"""
        # 1. 标记镜像
        full_image = f"{registry}/{image}"
        subprocess.run(["docker", "tag", image, full_image])
        
        # 2. 推送镜像
        subprocess.run(["docker", "push", full_image])

# 使用示例
docker = DockerIntegration()

# 构建镜像
image = docker.build_image("my-app", "v1.0")

# 运行容器
docker.run_container(image, {8000: 8000})

# 推送镜像
docker.push_image(image, "registry.example.com")
```

---

## 🔄 CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/openhands.yml
name: OpenHands CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup OpenHands
        run: pip install openhands
      
      - name: Run tests
        run: openhands run "运行单元测试"
      
      - name: Generate coverage
        run: openhands run "生成测试覆盖率报告"
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - pip install openhands
    - openhands run "运行单元测试"
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

build:
  stage: build
  script:
    - openhands run "构建 Docker 镜像"
  only:
    - main

deploy:
  stage: deploy
  script:
    - openhands run "部署到生产环境"
  only:
    - main
  when: manual
```

---

## 💻 IDE 集成

### VS Code

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "OpenHands: Run Task",
      "type": "shell",
      "command": "openhands",
      "args": ["run", "${input:task}"],
      "problemMatcher": []
    },
    {
      "label": "OpenHands: Generate Code",
      "type": "shell",
      "command": "openhands",
      "args": ["run", "生成 ${input:component} 组件"],
      "problemMatcher": []
    }
  ],
  "inputs": [
    {
      "id": "task",
      "type": "promptString",
      "description": "Enter task description"
    },
    {
      "id": "component",
      "type": "pickString",
      "options": ["Button", "Form", "Table", "Modal"],
      "description": "Select component type"
    }
  ]
}
```

### PyCharm

```xml
<!-- .idea/runConfigurations/OpenHands.xml -->
<component name="ProjectRunConfigurationManager">
  <configuration default="false" name="OpenHands" type="ShConfigurationType">
    <option name="SCRIPT_TEXT" value="openhands run &quot;$Prompt$&quot;" />
    <option name="INDEPENDENT_SCRIPT_PATH" value="true" />
    <envs>
      <env name="ANTHROPIC_API_KEY" value="sk-ant-xxx" />
    </envs>
  </configuration>
</component>
```

---

## 🔗 完整示例

### 项目自动化

```python
# integration/project_automation.py
from openhands import Agent
import subprocess

class ProjectAutomation:
    def __init__(self):
        self.agent = Agent(model="claude-3.5-sonnet")
    
    def create_feature(self, name: str):
        """创建功能分支并实现"""
        # 1. 创建分支
        subprocess.run(["git", "checkout", "-b", f"feature/{name}"])
        
        # 2. 实现功能
        result = self.agent.run(f"实现 {name} 功能")
        
        # 3. 提交代码
        subprocess.run(["git", "add", "-A"])
        subprocess.run(["git", "commit", "-m", f"feat: {name}"])
        
        # 4. 推送分支
        subprocess.run(["git", "push", "-u", "origin", f"feature/{name}"])
        
        # 5. 创建 PR
        subprocess.run([
            "gh", "pr", "create",
            "--title", f"feat: {name}",
            "--body", result.output
        ])
    
    def fix_bug(self, issue_id: str):
        """修复 Bug"""
        # 1. 获取 Issue 详情
        issue = subprocess.check_output([
            "gh", "issue", "view", issue_id
        ]).decode()
        
        # 2. 修复 Bug
        result = self.agent.run(f"修复 Bug: {issue}")
        
        # 3. 提交代码
        subprocess.run(["git", "add", "-A"])
        subprocess.run(["git", "commit", "-m", f"fix: #{issue_id}"])
        
        # 4. 推送
        subprocess.run(["git", "push"])
        
        # 5. 关闭 Issue
        subprocess.run([
            "gh", "issue", "close", issue_id,
            "--comment", "已修复"
        ])

# 使用示例
automation = ProjectAutomation()

# 创建功能
automation.create_feature("用户认证")

# 修复 Bug
automation.fix_bug("123")
```

---

<div align="center">
  <p>🔗 工具集成，提升效率！</p>
</div>
