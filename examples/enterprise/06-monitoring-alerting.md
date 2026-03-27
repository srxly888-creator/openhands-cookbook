# OpenHands 监控告警系统

> 全方位监控和智能告警

---

## 📋 目录

- [监控架构](#监控架构)
- [指标收集](#指标收集)
- [日志管理](#日志管理)
- [告警配置](#告警配置)

---

## 🏗️ 监控架构

### 监控栈

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
│ Node   │  │ App    │  │ Custom │
│Exporter│  │Metrics │  │Metrics │
└────────┘  └────────┘  └────────┘
```

---

## 📊 指标收集

### 1. Prometheus 配置

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'

scrape_configs:
  # 应用监控
  - job_name: 'openhands'
    static_configs:
      - targets: ['openhands:8000']
    metrics_path: '/metrics'

  # Node Exporter
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  # PostgreSQL
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  # Redis
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

  # Kubernetes
  - job_name: 'kubernetes'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### 2. 应用指标

```python
# monitoring/metrics.py
from prometheus_client import Counter, Histogram, Gauge
from fastapi import FastAPI, Request
import time

app = FastAPI()

# 定义指标
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint']
)

ACTIVE_REQUESTS = Gauge(
    'http_requests_active',
    'Active HTTP requests'
)

TASK_COUNT = Counter(
    'tasks_total',
    'Total tasks',
    ['status', 'agent']
)

TASK_DURATION = Histogram(
    'task_duration_seconds',
    'Task execution duration',
    ['agent']
)

# 中间件
@app.middleware("http")
async def metrics_middleware(request: Request, call_next):
    """指标收集中间件"""
    ACTIVE_REQUESTS.inc()
    
    start_time = time.time()
    
    try:
        response = await call_next(request)
        
        # 记录请求计数
        REQUEST_COUNT.labels(
            method=request.method,
            endpoint=request.url.path,
            status=response.status_code
        ).inc()
        
        # 记录请求延迟
        duration = time.time() - start_time
        REQUEST_LATENCY.labels(
            method=request.method,
            endpoint=request.url.path
        ).observe(duration)
        
        return response
    
    finally:
        ACTIVE_REQUESTS.dec()

# 自定义业务指标
@app.post("/api/v1/tasks")
async def create_task(task: str):
    """创建任务（记录业务指标）"""
    start_time = time.time()
    
    # 执行任务
    result = await execute_task(task)
    
    # 记录任务指标
    TASK_COUNT.labels(status="success", agent="openhands").inc()
    
    duration = time.time() - start_time
    TASK_DURATION.labels(agent="openhands").observe(duration)
    
    return result

# 指标端点
@app.get("/metrics")
async def metrics():
    """Prometheus 指标端点"""
    from prometheus_client import generate_latest
    return Response(content=generate_latest(), media_type="text/plain")
```

---

## 📝 日志管理

### 1. ELK Stack 配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.8.0
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml
      - /var/log:/var/log:ro
    depends_on:
      - logstash

volumes:
  elasticsearch_data:
```

### 2. Logstash 配置

```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "openhands" {
    grok {
      match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:log_message}" }
    }
    
    date {
      match => [ "timestamp", "ISO8601" ]
    }
    
    # 解析 JSON 日志
    if [log_message] =~ /^\{/ {
      json {
        source => "log_message"
        target => "json_data"
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "openhands-%{+YYYY.MM.dd}"
  }
}
```

### 3. Filebeat 配置

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/openhands/*.log
    fields:
      log_type: openhands
    fields_under_root: true

output.logstash:
  hosts: ["logstash:5044"]

setup.kibana:
  host: "kibana:5601"
```

### 4. Python 日志配置

```python
# monitoring/logging_config.py
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """JSON 格式化器"""
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
        
        if record.exc_info:
            log_entry["exception"] = self.formatException(record.exc_info)
        
        return json.dumps(log_entry)

# 配置日志
def setup_logging():
    """设置日志"""
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    
    # 文件处理器
    file_handler = logging.FileHandler("/var/log/openhands/app.log")
    file_handler.setFormatter(JSONFormatter())
    logger.addHandler(file_handler)
    
    # 控制台处理器
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(JSONFormatter())
    logger.addHandler(console_handler)

# 使用示例
setup_logging()
logger = logging.getLogger(__name__)

logger.info("Application started", extra={"version": "1.0.0"})
logger.error("Task failed", extra={"task_id": "task-123", "error": "Timeout"})
```

---

## 🚨 告警配置

### 1. AlertManager 配置

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

route:
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'slack-notifications'
  routes:
    - match:
        severity: critical
      receiver: 'slack-critical'
    - match:
        severity: warning
      receiver: 'slack-warning'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#openhands-alerts'
        send_resolved: true
        title: '{{ .Status | toUpper }}: {{ .CommonAnnotations.summary }}'
        text: '{{ .CommonAnnotations.description }}'

  - name: 'slack-critical'
    slack_configs:
      - channel: '#openhands-critical'
        send_resolved: true
        title: '🚨 CRITICAL: {{ .CommonAnnotations.summary }}'
        text: '{{ .CommonAnnotations.description }}'

  - name: 'slack-warning'
    slack_configs:
      - channel: '#openhands-warnings'
        send_resolved: true
        title: '⚠️ WARNING: {{ .CommonAnnotations.summary }}'
        text: '{{ .CommonAnnotations.description }}'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

### 2. 告警规则

```yaml
# alerts.yml
groups:
  - name: openhands
    rules:
      # ========== 应用告警 ==========
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests/s (threshold: 0.05)"

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "P95 latency is {{ $value }}s (threshold: 1s)"

      - alert: LowThroughput
        expr: rate(http_requests_total[5m]) < 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low throughput detected"
          description: "Throughput is {{ $value }} requests/s (threshold: 10)"

      # ========== 资源告警 ==========
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value }}% (threshold: 80%)"

      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}% (threshold: 85%)"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space low"
          description: "Available disk space is {{ $value }}% (threshold: 10%)"

      # ========== 容器告警 ==========
      - alert: ContainerRestarting
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Container is restarting frequently"
          description: "Container {{ $labels.pod }} has restarted {{ $value }} times in the last hour"

      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Pod is crash looping"
          description: "Pod {{ $labels.pod }} is restarting"

      - alert: PodNotReady
        expr: kube_pod_status_phase{phase=~"Failed|Unknown"} > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Pod is not ready"
          description: "Pod {{ $labels.pod }} is in {{ $labels.phase }} state"

      # ========== 数据库告警 ==========
      - alert: PostgreSQLDown
        expr: pg_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PostgreSQL is down"
          description: "PostgreSQL instance {{ $labels.instance }} is down"

      - alert: PostgreSQLTooManyConnections
        expr: pg_stat_activity_count > 100
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Too many PostgreSQL connections"
          description: "PostgreSQL has {{ $value }} active connections (threshold: 100)"

      # ========== Redis 告警 ==========
      - alert: RedisDown
        expr: redis_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Redis is down"
          description: "Redis instance {{ $labels.instance }} is down"

      - alert: RedisMemoryHigh
        expr: redis_memory_used_bytes / redis_memory_max_bytes * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis memory usage high"
          description: "Redis memory usage is {{ $value }}% (threshold: 85%)"
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
            "expr": "100 - (avg by(instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "gauge",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100"
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

### 2. 告警原则
- **可操作性** - 告警应该能被解决
- **避免噪音** - 减少误报
- **分级告警** - Critical/Warning/Info
- **聚合通知** - 避免告警风暴

### 3. 日志规范
- **结构化** - JSON 格式
- **上下文丰富** - 包含 trace_id、user_id 等
- **敏感信息脱敏** - 不记录密码等
- **分级存储** - 按重要性和时效性

---

<div align="center">
  <p>📊 全方位监控，智能告警！</p>
</div>
