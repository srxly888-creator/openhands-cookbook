# OpenHands 自动化运维指南

> DevOps 自动化实践

---

## 📋 目录

- [自动化部署](#自动化部署)
- [自动化测试](#自动化测试)
- [自动化监控](#自动化监控)
- [自动化备份](#自动化备份)

---

## 🚀 自动化部署

### CI/CD 流水线

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: pytest tests/ -v --cov=src
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t openhands:${{ github.sha }} .
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push openhands:${{ github.sha }}

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production
        run: |
          kubectl set image deployment/openhands \
            openhands=openhands:${{ github.sha }} \
            -n production
      
      - name: Wait for deployment
        run: |
          kubectl rollout status deployment/openhands -n production --timeout=300s
      
      - name: Run health check
        run: |
          curl -f https://api.openhands.dev/health || exit 1
```

### 自动化部署脚本

```python
# deploy/deploy.py
import subprocess
import time
from typing import Dict

class AutoDeployer:
    """自动部署器"""
    
    def __init__(self, config: Dict):
        self.config = config
        self.kubectl = "kubectl"
    
    def deploy(self, image: str, environment: str):
        """部署"""
        print(f"🚀 Deploying {image} to {environment}")
        
        # 1. 更新镜像
        self._update_image(image, environment)
        
        # 2. 等待部署
        self._wait_for_deployment(environment)
        
        # 3. 健康检查
        self._health_check(environment)
        
        print(f"✅ Deployment successful")
    
    def _update_image(self, image: str, environment: str):
        """更新镜像"""
        cmd = [
            self.kubectl, "set", "image",
            f"deployment/{self.config['deployment_name']}",
            f"{self.config['container_name']}={image}",
            "-n", environment
        ]
        
        subprocess.run(cmd, check=True)
    
    def _wait_for_deployment(self, environment: str):
        """等待部署完成"""
        cmd = [
            self.kubectl, "rollout", "status",
            f"deployment/{self.config['deployment_name']}",
            "-n", environment,
            "--timeout=300s"
        ]
        
        subprocess.run(cmd, check=True)
    
    def _health_check(self, environment: str):
        """健康检查"""
        url = self.config['health_urls'][environment]
        
        for i in range(10):
            try:
                subprocess.run(
                    ["curl", "-f", url],
                    check=True,
                    capture_output=True
                )
                return
            except subprocess.CalledProcessError:
                time.sleep(10)
        
        raise Exception("Health check failed")
    
    def rollback(self, environment: str):
        """回滚"""
        print(f"🔄 Rolling back {environment}")
        
        cmd = [
            self.kubectl, "rollout", "undo",
            f"deployment/{self.config['deployment_name']}",
            "-n", environment
        ]
        
        subprocess.run(cmd, check=True)
        print(f"✅ Rollback successful")

# 使用
config = {
    'deployment_name': 'openhands',
    'container_name': 'openhands',
    'health_urls': {
        'production': 'https://api.openhands.dev/health',
        'staging': 'https://staging.openhands.dev/health'
    }
}

deployer = AutoDeployer(config)
deployer.deploy('openhands:v1.0.0', 'production')
```

---

## 🧪 自动化测试

### 测试自动化框架

```python
# test/automation.py
import pytest
import subprocess
from typing import List

class TestAutomation:
    """测试自动化"""
    
    @staticmethod
    def run_unit_tests():
        """运行单元测试"""
        result = subprocess.run(
            ["pytest", "tests/unit", "-v", "--cov=src"],
            capture_output=True,
            text=True
        )
        
        return result.returncode == 0
    
    @staticmethod
    def run_integration_tests():
        """运行集成测试"""
        result = subprocess.run(
            ["pytest", "tests/integration", "-v"],
            capture_output=True,
            text=True
        )
        
        return result.returncode == 0
    
    @staticmethod
    def run_e2e_tests():
        """运行 E2E 测试"""
        result = subprocess.run(
            ["pytest", "tests/e2e", "-v"],
            capture_output=True,
            text=True
        )
        
        return result.returncode == 0
    
    @staticmethod
    def run_performance_tests():
        """运行性能测试"""
        result = subprocess.run(
            ["locust", "-f", "tests/performance/locustfile.py"],
            capture_output=True,
            text=True
        )
        
        return result.returncode == 0
    
    @staticmethod
    def run_security_tests():
        """运行安全测试"""
        result = subprocess.run(
            ["bandit", "-r", "src"],
            capture_output=True,
            text=True
        )
        
        return result.returncode == 0

# 使用
automation = TestAutomation()

# 运行所有测试
tests = [
    ("Unit", automation.run_unit_tests),
    ("Integration", automation.run_integration_tests),
    ("E2E", automation.run_e2e_tests),
    ("Performance", automation.run_performance_tests),
    ("Security", automation.run_security_tests)
]

results = {}
for name, test_func in tests:
    print(f"Running {name} tests...")
    results[name] = test_func()
    print(f"{'✅' if results[name] else '❌'} {name} tests")

# 检查结果
if all(results.values()):
    print("✅ All tests passed!")
else:
    print("❌ Some tests failed")
    for name, passed in results.items():
        if not passed:
            print(f"  - {name}")
```

---

## 📊 自动化监控

### 监控自动化

```python
# monitor/automation.py
import asyncio
from typing import Dict, List
import aiohttp

class MonitoringAutomation:
    """监控自动化"""
    
    def __init__(self, config: Dict):
        self.config = config
        self.alerts: List[Dict] = []
    
    async def start_monitoring(self):
        """启动监控"""
        while True:
            # 检查健康
            await self._check_health()
            
            # 检查性能
            await self._check_performance()
            
            # 检查资源
            await self._check_resources()
            
            # 等待
            await asyncio.sleep(60)
    
    async def _check_health(self):
        """检查健康"""
        for service in self.config['services']:
            try:
                async with aiohttp.ClientSession() as session:
                    async with session.get(
                        f"{service['url']}/health",
                        timeout=aiohttp.ClientTimeout(total=5)
                    ) as response:
                        if response.status != 200:
                            self._send_alert(
                                severity="critical",
                                message=f"{service['name']} is unhealthy"
                            )
            except Exception as e:
                self._send_alert(
                    severity="critical",
                    message=f"{service['name']} health check failed: {e}"
                )
    
    async def _check_performance(self):
        """检查性能"""
        for service in self.config['services']:
            try:
                async with aiohttp.ClientSession() as session:
                    async with session.get(
                        f"{service['url']}/metrics",
                        timeout=aiohttp.ClientTimeout(total=5)
                    ) as response:
                        metrics = await response.json()
                        
                        # 检查延迟
                        if metrics.get('latency_p95', 0) > 1.0:
                            self._send_alert(
                                severity="warning",
                                message=f"{service['name']} high latency: {metrics['latency_p95']}s"
                            )
                        
                        # 检查错误率
                        if metrics.get('error_rate', 0) > 0.05:
                            self._send_alert(
                                severity="warning",
                                message=f"{service['name']} high error rate: {metrics['error_rate']}"
                            )
            except Exception as e:
                self._send_alert(
                    severity="warning",
                    message=f"{service['name']} performance check failed: {e}"
                )
    
    async def _check_resources(self):
        """检查资源"""
        for service in self.config['services']:
            try:
                async with aiohttp.ClientSession() as session:
                    async with session.get(
                        f"{service['url']}/metrics",
                        timeout=aiohttp.ClientTimeout(total=5)
                    ) as response:
                        metrics = await response.json()
                        
                        # 检查 CPU
                        if metrics.get('cpu_usage', 0) > 80:
                            self._send_alert(
                                severity="warning",
                                message=f"{service['name']} high CPU: {metrics['cpu_usage']}%"
                            )
                        
                        # 检查内存
                        if metrics.get('memory_usage', 0) > 80:
                            self._send_alert(
                                severity="warning",
                                message=f"{service['name']} high memory: {metrics['memory_usage']}%"
                            )
            except Exception as e:
                self._send_alert(
                    severity="warning",
                    message=f"{service['name']} resource check failed: {e}"
                )
    
    def _send_alert(self, severity: str, message: str):
        """发送告警"""
        alert = {
            'severity': severity,
            'message': message,
            'timestamp': datetime.utcnow().isoformat()
        }
        
        self.alerts.append(alert)
        
        # 发送通知
        if severity == 'critical':
            self._send_email(alert)
            self._send_slack(alert)
        elif severity == 'warning':
            self._send_slack(alert)
    
    def _send_email(self, alert: Dict):
        """发送邮件"""
        # 实现邮件发送
        pass
    
    def _send_slack(self, alert: Dict):
        """发送 Slack"""
        # 实现 Slack 发送
        pass

# 使用
config = {
    'services': [
        {
            'name': 'API',
            'url': 'https://api.openhands.dev'
        },
        {
            'name': 'Web',
            'url': 'https://openhands.dev'
        }
    ]
}

monitor = MonitoringAutomation(config)
asyncio.run(monitor.start_monitoring())
```

---

## 💾 自动化备份

### 备份自动化

```python
# backup/automation.py
import subprocess
from datetime import datetime
from typing import Dict
import os

class BackupAutomation:
    """备份自动化"""
    
    def __init__(self, config: Dict):
        self.config = config
    
    def backup_database(self):
        """备份数据库"""
        timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
        backup_file = f"{self.config['backup_dir']}/db-{timestamp}.sql"
        
        # 执行备份
        cmd = [
            "pg_dump",
            "-h", self.config['db_host'],
            "-U", self.config['db_user'],
            "-d", self.config['db_name'],
            "-f", backup_file
        ]
        
        subprocess.run(cmd, check=True, env={
            'PGPASSWORD': self.config['db_password']
        })
        
        # 压缩
        subprocess.run(["gzip", backup_file], check=True)
        
        # 上传到 S3
        self._upload_to_s3(f"{backup_file}.gz")
        
        # 清理旧备份
        self._cleanup_old_backups()
        
        print(f"✅ Database backed up: {backup_file}.gz")
    
    def backup_files(self):
        """备份文件"""
        timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
        backup_file = f"{self.config['backup_dir']}/files-{timestamp}.tar.gz"
        
        # 打包
        cmd = [
            "tar", "-czf", backup_file,
            "-C", self.config['files_dir'], "."
        ]
        
        subprocess.run(cmd, check=True)
        
        # 上传到 S3
        self._upload_to_s3(backup_file)
        
        # 清理旧备份
        self._cleanup_old_backups()
        
        print(f"✅ Files backed up: {backup_file}")
    
    def _upload_to_s3(self, file_path: str):
        """上传到 S3"""
        cmd = [
            "aws", "s3", "cp",
            file_path,
            f"s3://{self.config['s3_bucket']}/backups/"
        ]
        
        subprocess.run(cmd, check=True)
    
    def _cleanup_old_backups(self):
        """清理旧备份"""
        # 删除本地旧备份
        cmd = [
            "find", self.config['backup_dir'],
            "-name", "*.gz",
            "-mtime", "+7",
            "-delete"
        ]
        
        subprocess.run(cmd, check=True)
        
        # 删除 S3 旧备份
        cmd = [
            "aws", "s3", "ls",
            f"s3://{self.config['s3_bucket']}/backups/"
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        # 解析并删除旧备份
        # ...

# 使用
config = {
    'backup_dir': '/var/backups/openhands',
    'db_host': 'localhost',
    'db_user': 'openhands',
    'db_password': 'password',
    'db_name': 'openhands',
    'files_dir': '/var/lib/openhands',
    's3_bucket': 'openhands-backups'
}

backup = BackupAutomation(config)
backup.backup_database()
backup.backup_files()
```

### 自动化恢复

```python
# backup/restore.py
import subprocess
from typing import Dict

class RestoreAutomation:
    """恢复自动化"""
    
    def __init__(self, config: Dict):
        self.config = config
    
    def restore_database(self, backup_file: str):
        """恢复数据库"""
        print(f"🔄 Restoring database from {backup_file}")
        
        # 下载备份
        local_file = self._download_from_s3(backup_file)
        
        # 解压
        subprocess.run(["gunzip", local_file], check=True)
        
        # 恢复
        sql_file = local_file.replace('.gz', '')
        cmd = [
            "psql",
            "-h", self.config['db_host'],
            "-U", self.config['db_user'],
            "-d", self.config['db_name'],
            "-f", sql_file
        ]
        
        subprocess.run(cmd, check=True, env={
            'PGPASSWORD': self.config['db_password']
        })
        
        print(f"✅ Database restored")
    
    def restore_files(self, backup_file: str):
        """恢复文件"""
        print(f"🔄 Restoring files from {backup_file}")
        
        # 下载备份
        local_file = self._download_from_s3(backup_file)
        
        # 解压
        cmd = [
            "tar", "-xzf", local_file,
            "-C", self.config['files_dir']
        ]
        
        subprocess.run(cmd, check=True)
        
        print(f"✅ Files restored")
    
    def _download_from_s3(self, backup_file: str) -> str:
        """从 S3 下载"""
        local_path = f"/tmp/{os.path.basename(backup_file)}"
        
        cmd = [
            "aws", "s3", "cp",
            f"s3://{self.config['s3_bucket']}/backups/{backup_file}",
            local_path
        ]
        
        subprocess.run(cmd, check=True)
        
        return local_path

# 使用
restore = RestoreAutomation(config)
restore.restore_database('db-20260327-120000.sql.gz')
restore.restore_files('files-20260327-120000.tar.gz')
```

---

## 🎯 自动化最佳实践

### 1. 自动化原则

- **可重复** - 自动化流程可重复执行
- **可回滚** - 支持快速回滚
- **可监控** - 实时监控执行状态
- **可审计** - 记录所有操作

### 2. 部署原则

- **蓝绿部署** - 零停机部署
- **金丝雀发布** - 逐步滚动
- **自动回滚** - 失败自动回滚

### 3. 测试原则

- **自动化** - 所有测试自动化
- **快速** - 测试快速执行
- **可靠** - 测试结果可靠

### 4. 监控原则

- **实时** - 实时监控
- **告警** - 及时告警
- **可视化** - 数据可视化

---

<div align="center">
  <p>🤖 自动化运维，效率提升！</p>
</div>
