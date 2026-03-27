# OpenHands 灾难恢复方案

> 从 RTO 4小时 到 RTO 15分钟

---

## 📋 目录

- [灾难恢复策略](#灾难恢复策略)
- [备份方案](#备份方案)
- [故障转移](#故障转移)
- [恢复流程](#恢复流程)

---

## 🚨 灾难恢复策略

### RPO/RTO 目标

```
┌─────────────────────────────────────┐
│         灾难恢复目标                │
├─────────────────────────────────────┤
│ RPO (Recovery Point Objective)     │
│ 目标：< 15 分钟数据丢失             │
├─────────────────────────────────────┤
│ RTO (Recovery Time Objective)      │
│ 目标：< 15 分钟服务恢复             │
└─────────────────────────────────────┘
```

### 多区域架构

```
┌─────────────────────────────────────┐
│      Region A (Primary)             │
│  ┌──────────────────────────────┐   │
│  │   Application (Active)       │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │   Database (Master)          │   │
│  └──────────────────────────────┘   │
└─────────────────┬───────────────────┘
                  │
         Synchronous Replication
                  │
┌─────────────────▼───────────────────┐
│      Region B (Standby)             │
│  ┌──────────────────────────────┐   │
│  │   Application (Standby)      │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │   Database (Replica)         │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

---

## 💾 备份方案

### 1. 数据库备份

```python
# backup/database.py
import boto3
from datetime import datetime
import subprocess

class DatabaseBackup:
    def __init__(self):
        self.rds = boto3.client('rds')
        self.s3 = boto3.client('s3')
        self.db_identifier = 'openhands-db'
        self.bucket = 'openhands-backups'
    
    def create_snapshot(self):
        """创建数据库快照"""
        snapshot_id = f"{self.db_identifier}-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
        
        response = self.rds.create_db_snapshot(
            DBSnapshotIdentifier=snapshot_id,
            DBInstanceIdentifier=self.db_identifier
        )
        
        return response
    
    def export_snapshot(self, snapshot_id: str):
        """导出快照到 S3"""
        response = self.rds.start_export_task(
            ExportTaskIdentifier=f"export-{snapshot_id}",
            SourceArn=f"arn:aws:rds:us-east-1:123456789:snapshot:{snapshot_id}",
            S3BucketName=self.bucket,
            IamRoleArn='arn:aws:iam::123456789:role/rds-export-role',
            KmsKeyId='arn:aws:kms:us-east-1:123456789:key/123456789'
        )
        
        return response
    
    def automated_backup(self):
        """自动化备份（每小时）"""
        # 1. 创建快照
        snapshot = self.create_snapshot()
        
        # 2. 等待完成
        self.wait_for_snapshot(snapshot['DBSnapshotIdentifier'])
        
        # 3. 导出到 S3
        self.export_snapshot(snapshot['DBSnapshotIdentifier'])
        
        # 4. 清理旧备份（保留 7 天）
        self.cleanup_old_backups(7)
        
        return snapshot

# 使用示例
backup = DatabaseBackup()
snapshot = backup.automated_backup()
```

### 2. 应用备份

```python
# backup/application.py
import boto3
from datetime import datetime
import docker

class ApplicationBackup:
    def __init__(self):
        self.ecr = boto3.client('ecr')
        self.s3 = boto3.client('s3')
        self.docker = docker.from_env()
    
    def backup_container_image(self):
        """备份容器镜像"""
        # 1. 获取当前镜像
        image = self.docker.images.get('openhands:latest')
        
        # 2. 推送到 ECR
        repo_name = 'openhands-backup'
        
        # 登录 ECR
        self.ecr.get_authorization_token()
        
        # 标记镜像
        image.tag(f"{repo_name}:latest")
        image.tag(f"{repo_name}:{datetime.now().strftime('%Y%m%d-%H%M%S')}")
        
        # 推送
        self.docker.images.push(repo_name)
        
        return image
    
    def backup_configurations(self):
        """备份配置文件"""
        configs = [
            '/app/config/database.yml',
            '/app/config/secrets.yml',
            '/app/config/environment.yml'
        ]
        
        for config in configs:
            self.s3.upload_file(
                config,
                'openhands-backups',
                f"configs/{datetime.now().strftime('%Y%m%d-%H%M%S')}/{config.split('/')[-1]}"
            )

# 使用示例
backup = ApplicationBackup()
backup.backup_container_image()
backup.backup_configurations()
```

### 3. 增量备份

```python
# backup/incremental.py
import boto3
from datetime import datetime, timedelta
import hashlib
import os

class IncrementalBackup:
    def __init__(self):
        self.s3 = boto3.client('s3')
        self.bucket = 'openhands-incremental-backups'
    
    def calculate_file_hash(self, filepath: str) -> str:
        """计算文件哈希"""
        hasher = hashlib.md5()
        with open(filepath, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b''):
                hasher.update(chunk)
        return hasher.hexdigest()
    
    def get_changed_files(self, directory: str, last_backup_time: datetime):
        """获取变更文件"""
        changed_files = []
        
        for root, dirs, files in os.walk(directory):
            for file in files:
                filepath = os.path.join(root, file)
                
                # 检查修改时间
                mtime = datetime.fromtimestamp(os.path.getmtime(filepath))
                
                if mtime > last_backup_time:
                    changed_files.append(filepath)
        
        return changed_files
    
    def backup_changes(self, directory: str):
        """增量备份"""
        # 1. 获取上次备份时间
        last_backup = self.get_last_backup_time()
        
        # 2. 获取变更文件
        changed_files = self.get_changed_files(directory, last_backup)
        
        # 3. 上传变更文件
        for filepath in changed_files:
            s3_key = f"incremental/{datetime.now().strftime('%Y%m%d-%H%M%S')}/{filepath}"
            
            self.s3.upload_file(filepath, self.bucket, s3_key)
        
        # 4. 更新备份时间
        self.update_last_backup_time()
        
        return {
            "files_backed_up": len(changed_files),
            "backup_time": datetime.now().isoformat()
        }

# 使用示例
backup = IncrementalBackup()
result = backup.backup_changes('/app/data')
```

---

## 🔄 故障转移

### 1. 数据库故障转移

```python
# failover/database.py
import boto3
import time

class DatabaseFailover:
    def __init__(self):
        self.rds = boto3.client('rds')
        self.cluster_identifier = 'openhands-cluster'
    
    def check_primary_health(self):
        """检查主库健康"""
        try:
            response = self.rds.describe_db_clusters(
                DBClusterIdentifier=self.cluster_identifier
            )
            
            cluster = response['DBClusters'][0]
            status = cluster['Status']
            
            return status == 'available'
        except Exception as e:
            print(f"Error checking primary: {e}")
            return False
    
    def trigger_failover(self):
        """触发故障转移"""
        print("🚨 Triggering database failover...")
        
        # 1. 检查主库状态
        if self.check_primary_health():
            print("✅ Primary is healthy, no failover needed")
            return False
        
        # 2. 触发故障转移
        response = self.rds.failover_db_cluster(
            DBClusterIdentifier=self.cluster_identifier
        )
        
        # 3. 等待完成
        self.wait_for_failover()
        
        print("✅ Failover completed")
        return True
    
    def wait_for_failover(self, timeout=300):
        """等待故障转移完成"""
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            response = self.rds.describe_db_clusters(
                DBClusterIdentifier=self.cluster_identifier
            )
            
            status = response['DBClusters'][0]['Status']
            
            if status == 'available':
                return True
            
            time.sleep(10)
        
        raise TimeoutError("Failover timed out")

# 使用示例
failover = DatabaseFailover()
failover.trigger_failover()
```

### 2. 应用故障转移

```python
# failover/application.py
import boto3
from kubernetes import client, config

class ApplicationFailover:
    def __init__(self):
        self.elbv2 = boto3.client('elbv2')
        config.load_kube_config()
        self.k8s = client.AppsV1Api()
    
    def switch_traffic(self, target_region: str):
        """切换流量到目标区域"""
        print(f"🔀 Switching traffic to {target_region}...")
        
        # 1. 获取目标 ALB
        target_alb_arn = self.get_alb_arn(target_region)
        
        # 2. 更新 DNS 记录
        self.update_dns(target_alb_arn)
        
        # 3. 验证流量切换
        self.verify_traffic_switch(target_region)
        
        print(f"✅ Traffic switched to {target_region}")
    
    def get_alb_arn(self, region: str):
        """获取 ALB ARN"""
        response = self.elbv2.describe_load_balancers(
            Names=[f'openhands-alb-{region}']
        )
        
        return response['LoadBalancers'][0]['LoadBalancerArn']
    
    def update_dns(self, alb_arn: str):
        """更新 DNS 记录"""
        route53 = boto3.client('route53')
        
        # 获取 ALB DNS 名称
        alb_dns = self.elbv2.describe_load_balancers(
            LoadBalancerArns=[alb_arn]
        )['LoadBalancers'][0]['DNSName']
        
        # 更新 Route53
        route53.change_resource_record_sets(
            HostedZoneId='Z1234567890ABC',
            ChangeBatch={
                'Changes': [
                    {
                        'Action': 'UPSERT',
                        'ResourceRecordSet': {
                            'Name': 'api.openhands.dev',
                            'Type': 'CNAME',
                            'TTL': 60,
                            'ResourceRecords': [{'Value': alb_dns}]
                        }
                    }
                ]
            }
        )
    
    def scale_standby(self, replicas: int = 3):
        """扩展备用区域"""
        self.k8s.patch_namespaced_deployment_scale(
            name='openhands',
            namespace='default',
            body={
                'spec': {
                    'replicas': replicas
                }
            }
        )

# 使用示例
failover = ApplicationFailover()
failover.switch_traffic('us-west-2')
failover.scale_standby(3)
```

---

## 🔧 恢复流程

### 1. 自动恢复脚本

```python
# recovery/auto_recovery.py
import boto3
import time
from monitoring.health_check import HealthChecker

class AutoRecovery:
    def __init__(self):
        self.health_checker = HealthChecker()
        self.database_failover = DatabaseFailover()
        self.app_failover = ApplicationFailover()
    
    def monitor_and_recover(self):
        """监控并自动恢复"""
        while True:
            # 1. 健康检查
            health = self.health_checker.check_all()
            
            # 2. 检测故障
            if not health['database']:
                print("🚨 Database failure detected")
                self.database_failover.trigger_failover()
            
            if not health['application']:
                print("🚨 Application failure detected")
                self.app_failover.switch_traffic('us-west-2')
            
            # 3. 等待下一次检查
            time.sleep(60)

# 使用示例
recovery = AutoRecovery()
recovery.monitor_and_recover()
```

### 2. 手动恢复流程

```bash
# recovery/manual_recovery.sh

#!/bin/bash

echo "🚨 Starting disaster recovery..."

# 1. 检查主区域状态
echo "📊 Checking primary region status..."
aws rds describe-db-instances --db-instance-identifier openhands-db

# 2. 触发数据库故障转移
echo "🔄 Triggering database failover..."
aws rds failover-db-cluster --db-cluster-identifier openhands-cluster

# 3. 切换应用流量
echo "🔀 Switching application traffic..."
python3 failover/application.py --target us-west-2

# 4. 验证服务恢复
echo "✅ Verifying service recovery..."
curl -f https://api.openhands.dev/health || exit 1

echo "🎉 Recovery completed!"
```

---

## 📊 恢复测试

### 混沌工程测试

```python
# recovery/chaos_test.py
import random
from recovery.auto_recovery import AutoRecovery

class ChaosTest:
    def __init__(self):
        self.recovery = AutoRecovery()
    
    def inject_database_failure(self):
        """注入数据库故障"""
        print("💥 Injecting database failure...")
        
        # 停止数据库
        # 等待自动恢复
        # 验证恢复时间
        
        pass
    
    def inject_network_failure(self):
        """注入网络故障"""
        print("💥 Injecting network failure...")
        
        # 模拟网络分区
        # 等待故障转移
        # 验证服务可用性
        
        pass
    
    def measure_recovery_time(self):
        """测量恢复时间"""
        start_time = time.time()
        
        # 注入故障
        self.inject_database_failure()
        
        # 等待恢复
        self.recovery.monitor_and_recover()
        
        # 计算恢复时间
        recovery_time = time.time() - start_time
        
        print(f"⏱️ Recovery time: {recovery_time} seconds")
        
        # 验证是否满足 RTO
        if recovery_time <= 900:  # 15 分钟
            print("✅ RTO target met!")
        else:
            print("❌ RTO target missed!")

# 使用示例
chaos = ChaosTest()
chaos.measure_recovery_time()
```

---

## 🎯 灾难恢复检查清单

### ✅ 备份
- [ ] 数据库自动备份（每小时）
- [ ] 应用配置备份（每天）
- [ ] 增量备份（实时）
- [ ] 备份验证（每周）

### ✅ 故障转移
- [ ] 数据库故障转移测试（每月）
- [ ] 应用故障转移测试（每月）
- [ ] DNS 切换测试（每季度）
- [ ] 全链路测试（每半年）

### ✅ 恢复流程
- [ ] 文档化恢复流程
- [ ] 自动化恢复脚本
- [ ] 定期演练（每季度）
- [ ] 团队培训

### ✅ 监控告警
- [ ] 健康检查（每分钟）
- [ ] 故障告警（即时）
- [ ] 恢复告警（即时）
- [ ] 演练报告（每次）

---

<div align="center">
  <p>🚨 灾难恢复，有备无患！</p>
</div>
