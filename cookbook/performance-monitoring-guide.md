# OpenHands 性能监控完全指南

> 实时监控和性能分析

---

## 📋 目录

- [监控架构](#监控架构)
- [指标收集](#指标收集)
- [日志管理](#日志管理)
- [告警配置](#告警配置)

---

## 🏗️ 监控架构

### 完整监控栈

```
┌─────────────────────────────────────┐
│       Grafana（可视化）             │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│    Prometheus（指标存储）            │
└────────────────┬────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│ App    │  │ System │  │ Custom │
│Metrics │  │Metrics │  │Metrics │
└────────┘  └────────┘  └────────┘
```

### 监控组件

```python
# monitoring/monitor.py
from prometheus_client import Counter, Histogram, Gauge
import time
from functools import wraps

class PerformanceMonitor:
    """性能监控器"""
    
    def __init__(self):
        # 定义指标
        self.request_count = Counter(
            'http_requests_total',
            'Total HTTP requests',
            ['method', 'endpoint', 'status']
        )
        
        self.request_latency = Histogram(
            'http_request_duration_seconds',
            'HTTP request latency',
            ['method', 'endpoint']
        )
        
        self.active_requests = Gauge(
            'http_requests_active',
            'Active HTTP requests'
        )
        
        self.task_count = Counter(
            'tasks_total',
            'Total tasks',
            ['status', 'agent']
        )
        
        self.task_duration = Histogram(
            'task_duration_seconds',
            'Task execution duration',
            ['agent']
        )
        
        self.memory_usage = Gauge(
            'memory_usage_bytes',
            'Memory usage in bytes'
        )
        
        self.cpu_usage = Gauge(
            'cpu_usage_percent',
            'CPU usage percentage'
        )
    
    def monitor_request(self, func):
        """监控请求装饰器"""
        @wraps(func)
        async def wrapper(*args, **kwargs):
            self.active_requests.inc()
            
            start_time = time.time()
            
            try:
                response = await func(*args, **kwargs)
                
                # 记录成功请求
                self.request_count.labels(
                    method="POST",
                    endpoint=func.__name__,
                    status=200
                ).inc()
                
                return response
            
            except Exception as e:
                # 记录失败请求
                self.request_count.labels(
                    method="POST",
                    endpoint=func.__name__,
                    status=500
                ).inc()
                
                raise
            
            finally:
                # 记录延迟
                duration = time.time() - start_time
                self.request_latency.labels(
                    method="POST",
                    endpoint=func.__name__
                ).observe(duration)
                
                self.active_requests.dec()
        
        return wrapper
    
    def monitor_task(self, func):
        """监控任务装饰器"""
        @wraps(func)
        async def wrapper(*args, **kwargs):
            start_time = time.time()
            
            try:
                result = await func(*args, **kwargs)
                
                # 记录成功任务
                self.task_count.labels(
                    status="success",
                    agent="openhands"
                ).inc()
                
                return result
            
            except Exception as e:
                # 记录失败任务
                self.task_count.labels(
                    status="failed",
                    agent="openhands"
                ).inc()
                
                raise
            
            finally:
                # 记录任务时长
                duration = time.time() - start_time
                self.task_duration.labels(
                    agent="openhands"
                ).observe(duration)
        
        return wrapper
    
    def update_system_metrics(self):
        """更新系统指标"""
        import psutil
        
        # 内存使用
        memory = psutil.virtual_memory()
        self.memory_usage.set(memory.used)
        
        # CPU 使用
        cpu = psutil.cpu_percent(interval=1)
        self.cpu_usage.set(cpu)

# 使用
monitor = PerformanceMonitor()

@monitor.monitor_request
async def create_task(task: str):
    """创建任务"""
    # 执行任务
    pass

@monitor.monitor_task
async def execute_task(task: str):
    """执行任务"""
    # 执行任务
    pass
```

---

## 📊 指标收集

### 自定义指标

```python
# monitoring/custom_metrics.py
from prometheus_client import Counter, Histogram, Gauge, Summary

class CustomMetrics:
    """自定义指标"""
    
    def __init__(self):
        # 业务指标
        self.user_registrations = Counter(
            'user_registrations_total',
            'Total user registrations',
            ['source']
        )
        
        self.order_value = Histogram(
            'order_value_dollars',
            'Order value in dollars',
            ['category'],
            buckets=[10, 50, 100, 500, 1000, 5000]
        )
        
        self.active_users = Gauge(
            'active_users',
            'Currently active users'
        )
        
        # 性能指标
        self.db_query_duration = Summary(
            'db_query_duration_seconds',
            'Database query duration',
            ['query_type']
        )
        
        self.cache_hits = Counter(
            'cache_hits_total',
            'Total cache hits',
            ['cache_type']
        )
        
        self.cache_misses = Counter(
            'cache_misses_total',
            'Total cache misses',
            ['cache_type']
        )
    
    def record_user_registration(self, source: str):
        """记录用户注册"""
        self.user_registrations.labels(source=source).inc()
    
    def record_order(self, value: float, category: str):
        """记录订单"""
        self.order_value.labels(category=category).observe(value)
    
    def set_active_users(self, count: int):
        """设置活跃用户数"""
        self.active_users.set(count)
    
    def record_db_query(self, duration: float, query_type: str):
        """记录数据库查询"""
        self.db_query_duration.labels(query_type=query_type).observe(duration)
    
    def record_cache_hit(self, cache_type: str):
        """记录缓存命中"""
        self.cache_hits.labels(cache_type=cache_type).inc()
    
    def record_cache_miss(self, cache_type: str):
        """记录缓存未命中"""
        self.cache_misses.labels(cache_type=cache_type).inc()

# 使用
metrics = CustomMetrics()

# 记录业务指标
metrics.record_user_registration("web")
metrics.record_order(99.99, "electronics")
metrics.set_active_users(150)

# 记录性能指标
metrics.record_db_query(0.05, "select")
metrics.record_cache_hit("redis")
```

### 指标导出

```python
# monitoring/exporter.py
from prometheus_client import start_http_server, generate_latest
from fastapi import Response

class MetricsExporter:
    """指标导出器"""
    
    def __init__(self, port: int = 9090):
        self.port = port
    
    def start(self):
        """启动指标服务器"""
        start_http_server(self.port)
        print(f"Metrics server started on port {self.port}")
    
    def get_metrics(self) -> str:
        """获取指标"""
        return generate_latest().decode('utf-8')
    
    async def metrics_endpoint(self):
        """FastAPI 端点"""
        from fastapi import Response
        
        metrics = self.get_metrics()
        return Response(
            content=metrics,
            media_type="text/plain"
        )

# FastAPI 集成
from fastapi import FastAPI

app = FastAPI()
exporter = MetricsExporter()

@app.get("/metrics")
async def metrics():
    """指标端点"""
    return await exporter.metrics_endpoint()

# 启动
exporter.start()
```

---

## 📝 日志管理

### 结构化日志

```python
# monitoring/logging_config.py
import logging
import json
from datetime import datetime

class StructuredFormatter(logging.Formatter):
    """结构化日志格式化器"""
    
    def format(self, record):
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno
        }
        
        # 添加额外字段
        if hasattr(record, 'user_id'):
            log_entry['user_id'] = record.user_id
        
        if hasattr(record, 'request_id'):
            log_entry['request_id'] = record.request_id
        
        if hasattr(record, 'duration'):
            log_entry['duration'] = record.duration
        
        # 添加异常信息
        if record.exc_info:
            log_entry['exception'] = self.formatException(record.exc_info)
        
        return json.dumps(log_entry)

# 配置日志
def setup_logging():
    """设置日志"""
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    
    # 文件处理器
    file_handler = logging.FileHandler('/var/log/openhands/app.log')
    file_handler.setFormatter(StructuredFormatter())
    logger.addHandler(file_handler)
    
    # 控制台处理器
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(StructuredFormatter())
    logger.addHandler(console_handler)

# 使用
setup_logging()
logger = logging.getLogger(__name__)

# 记录日志
logger.info(
    "User registered",
    extra={
        'user_id': 'user-123',
        'request_id': 'req-456'
    }
)

logger.error(
    "Database connection failed",
    extra={
        'duration': 5.2,
        'error': 'Connection timeout'
    }
)
```

### 日志聚合

```python
# monitoring/log_aggregator.py
from typing import List, Dict
import json

class LogAggregator:
    """日志聚合器"""
    
    def __init__(self):
        self.logs: List[Dict] = []
        self.max_logs = 10000
    
    def add_log(self, log: Dict):
        """添加日志"""
        self.logs.append(log)
        
        # 限制日志数量
        if len(self.logs) > self.max_logs:
            self.logs = self.logs[-self.max_logs:]
    
    def search(self, query: str, level: str = None) -> List[Dict]:
        """搜索日志"""
        results = []
        
        for log in self.logs:
            # 搜索消息
            if query.lower() in log.get('message', '').lower():
                # 过滤级别
                if level and log.get('level') != level:
                    continue
                
                results.append(log)
        
        return results
    
    def get_stats(self) -> Dict:
        """获取统计"""
        stats = {
            'total_logs': len(self.logs),
            'by_level': {},
            'by_module': {}
        }
        
        for log in self.logs:
            # 按级别统计
            level = log.get('level', 'UNKNOWN')
            stats['by_level'][level] = stats['by_level'].get(level, 0) + 1
            
            # 按模块统计
            module = log.get('module', 'unknown')
            stats['by_module'][module] = stats['by_module'].get(module, 0) + 1
        
        return stats

# 使用
aggregator = LogAggregator()

# 添加日志
aggregator.add_log({
    "timestamp": "2026-03-27T16:00:00Z",
    "level": "INFO",
    "message": "User registered",
    "module": "auth"
})

# 搜索日志
results = aggregator.search("user")

# 获取统计
stats = aggregator.get_stats()
```

---

## 🚨 告警配置

### 告警规则

```yaml
# monitoring/alerts.yml
groups:
  - name: openhands
    rules:
      # 应用告警
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests/s"
      
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "P95 latency is {{ $value }}s"
      
      # 资源告警
      - alert: HighCPUUsage
        expr: cpu_usage_percent > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value }}%"
      
      - alert: HighMemoryUsage
        expr: memory_usage_bytes / (1024 * 1024 * 1024) > 8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}GB"
```

### 告警管理器

```python
# monitoring/alert_manager.py
from typing import List, Dict
from datetime import datetime
import smtplib
from email.mime.text import MIMEText

class AlertManager:
    """告警管理器"""
    
    def __init__(self, config: Dict):
        self.config = config
        self.alerts: List[Dict] = []
    
    def send_alert(self, alert: Dict):
        """发送告警"""
        # 添加时间戳
        alert['timestamp'] = datetime.utcnow().isoformat()
        
        # 保存告警
        self.alerts.append(alert)
        
        # 发送通知
        self._send_notification(alert)
    
    def _send_notification(self, alert: Dict):
        """发送通知"""
        severity = alert.get('severity', 'info')
        
        if severity == 'critical':
            # 发送邮件
            self._send_email(alert)
            
            # 发送 Slack
            self._send_slack(alert)
        
        elif severity == 'warning':
            # 发送 Slack
            self._send_slack(alert)
    
    def _send_email(self, alert: Dict):
        """发送邮件"""
        msg = MIMEText(alert.get('description', ''))
        msg['Subject'] = f"[{alert.get('severity')}] {alert.get('summary')}"
        msg['From'] = self.config['email']['from']
        msg['To'] = self.config['email']['to']
        
        with smtplib.SMTP(
            self.config['email']['host'],
            self.config['email']['port']
        ) as server:
            server.send_message(msg)
    
    def _send_slack(self, alert: Dict):
        """发送 Slack"""
        import requests
        
        webhook_url = self.config['slack']['webhook_url']
        
        payload = {
            "text": f"[{alert.get('severity')}] {alert.get('summary')}",
            "attachments": [
                {
                    "text": alert.get('description'),
                    "color": "danger" if alert.get('severity') == 'critical' else "warning"
                }
            ]
        }
        
        requests.post(webhook_url, json=payload)
    
    def get_alerts(self, severity: str = None) -> List[Dict]:
        """获取告警"""
        if severity:
            return [a for a in self.alerts if a.get('severity') == severity]
        
        return self.alerts

# 使用
config = {
    'email': {
        'host': 'smtp.gmail.com',
        'port': 587,
        'from': 'alerts@example.com',
        'to': 'admin@example.com'
    },
    'slack': {
        'webhook_url': 'https://hooks.slack.com/services/...'
    }
}

alert_manager = AlertManager(config)

# 发送告警
alert_manager.send_alert({
    'severity': 'critical',
    'summary': 'High error rate detected',
    'description': 'Error rate is 0.1 requests/s'
})
```

---

## 📊 Grafana 仪表板

### 仪表板配置

```json
{
  "dashboard": {
    "title": "OpenHands Monitoring",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "{{status}}"
          }
        ]
      },
      {
        "title": "Latency (P95)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "P95"
          }
        ]
      },
      {
        "title": "CPU Usage",
        "type": "gauge",
        "targets": [
          {
            "expr": "cpu_usage_percent"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "gauge",
        "targets": [
          {
            "expr": "memory_usage_bytes / (1024 * 1024 * 1024)"
          }
        ]
      }
    ]
  }
}
```

---

## 🎯 监控最佳实践

### 1. 四大黄金信号

- **延迟**（Latency）- 请求响应时间
- **流量**（Traffic）- 请求速率
- **错误**（Errors）- 错误率
- **饱和度**（Saturation）- 资源使用率

### 2. 监控原则

- **可操作性** - 告警应该能被解决
- **避免噪音** - 减少误报
- **分级告警** - Critical/Warning/Info
- **聚合通知** - 避免告警风暴

### 3. 性能优化

- **采样率** - 合理设置采样率
- **聚合** - 使用聚合减少数据量
- **保留策略** - 设置合理的数据保留时间

---

<div align="center">
  <p>📊 实时监控，性能无忧！</p>
</div>
