# OpenHands 云原生部署指南

> Kubernetes 和云原生实践

---

## 📋 目录

- [云原生架构](#云原生架构)
- [Kubernetes 部署](#kubernetes-部署)
- [服务网格](#服务网格)
- [云原生最佳实践](#云原生最佳实践)

---

## ☁️ 云原生架构

### 云原生架构图

```
┌─────────────────────────────────────────┐
│           用户层（Users）                │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│      CDN + Load Balancer                │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│      Ingress Controller                 │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│ API    │  │ Web    │  │ Worker │
│Gateway │  │App     │  │Service │
└───┬────┘  └───┬────┘  └───┬────┘
    │           │            │
    └───────────┼────────────┘
                │
        ┌───────▼────────┐
        │ Service Mesh   │
        │ (Istio/Linkerd)│
        └───────┬────────┘
                │
    ┌───────────┼───────────┐
    │           │           │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│Database│  │ Cache  │  │Queue   │
│(RDS)   │  │(Redis) │  │(SQS)   │
└────────┘  └────────┘  └────────┘
```

### 云原生设计原则

```python
# cloud_native/principles.py
from typing import List, Dict

class CloudNativePrinciples:
    """云原生设计原则"""
    
    @staticmethod
    def microservices() -> List[str]:
        """微服务原则"""
        return [
            "单一职责 - 每个服务只负责一个业务功能",
            "独立部署 - 服务可以独立部署和扩展",
            "去中心化 - 避免单点故障",
            "容错设计 - 服务失败时能够优雅降级",
            "松耦合 - 服务之间通过 API 通信"
        ]
    
    @staticmethod
    def containerization() -> List[str]:
        """容器化原则"""
        return [
            "不可变基础设施 - 容器一旦创建不再修改",
            "轻量级 - 容器镜像尽可能小",
            "快速启动 - 容器启动时间 < 5s",
            "资源限制 - 设置 CPU 和内存限制",
            "健康检查 - 实现健康检查端点"
        ]
    
    @staticmethod
    def orchestration() -> List[str]:
        """编排原则"""
        return [
            "声明式配置 - 使用 YAML 描述期望状态",
            "自动调度 - 自动调度 Pod 到节点",
            "自动扩展 - 根据负载自动扩展",
            "滚动更新 - 零停机部署",
            "自愈能力 - 自动重启失败容器"
        ]
    
    @staticmethod
    def devops() -> List[str]:
        """DevOps 原则"""
        return [
            "自动化 - 所有流程自动化",
            "持续集成 - 代码提交自动构建测试",
            "持续部署 - 测试通过自动部署",
            "监控 - 实时监控所有服务",
            "日志 - 集中化日志管理"
        ]

# 使用
principles = CloudNativePrinciples()

print("微服务原则:")
for p in principles.microservices():
    print(f"  - {p}")

print("\n容器化原则:")
for p in principles.containerization():
    print(f"  - {p}")
```

---

## 🚀 Kubernetes 部署

### 完整部署配置

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openhands
  labels:
    app: openhands
    environment: production
```

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openhands-config
  namespace: openhands
data:
  DATABASE_URL: "postgresql://user:pass@postgres:5432/openhands"
  REDIS_URL: "redis://redis:6379"
  LOG_LEVEL: "INFO"
  MAX_WORKERS: "4"
```

```yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: openhands-secrets
  namespace: openhands
type: Opaque
stringData:
  ANTHROPIC_API_KEY: "sk-ant-xxx"
  DATABASE_PASSWORD: "secure_password"
  JWT_SECRET: "jwt_secret_key"
```

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openhands
  namespace: openhands
  labels:
    app: openhands
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openhands
  template:
    metadata:
      labels:
        app: openhands
        version: v1.0.0
    spec:
      containers:
      - name: openhands
        image: openhands/openhands:latest
        ports:
        - containerPort: 8000
          name: http
        
        envFrom:
        - configMapRef:
            name: openhands-config
        - secretRef:
            name: openhands-secrets
        
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        volumeMounts:
        - name: data
          mountPath: /data
      
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: openhands-pvc
      
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - openhands
              topologyKey: kubernetes.io/hostname
```

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: openhands
  namespace: openhands
spec:
  selector:
    app: openhands
  ports:
  - port: 80
    targetPort: 8000
    name: http
  type: ClusterIP
```

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: openhands-ingress
  namespace: openhands
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
spec:
  tls:
  - hosts:
    - api.openhands.dev
    secretName: openhands-tls
  rules:
  - host: api.openhands.dev
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: openhands
            port:
              number: 80
```

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: openhands-hpa
  namespace: openhands
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: openhands
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 部署脚本

```python
# k8s/deploy.py
import subprocess
from typing import Dict, List

class KubernetesDeployer:
    """Kubernetes 部署器"""
    
    def __init__(self, namespace: str = "openhands"):
        self.namespace = namespace
        self.kubectl = "kubectl"
    
    def deploy(self, environment: str):
        """部署"""
        print(f"🚀 Deploying to {environment}")
        
        # 1. 创建命名空间
        self._create_namespace()
        
        # 2. 应用配置
        self._apply_configs()
        
        # 3. 部署应用
        self._deploy_app()
        
        # 4. 等待就绪
        self._wait_for_ready()
        
        # 5. 健康检查
        self._health_check()
        
        print(f"✅ Deployment successful")
    
    def _create_namespace(self):
        """创建命名空间"""
        cmd = [
            self.kubectl, "apply",
            "-f", "k8s/namespace.yaml"
        ]
        
        subprocess.run(cmd, check=True)
    
    def _apply_configs(self):
        """应用配置"""
        configs = [
            "k8s/configmap.yaml",
            "k8s/secret.yaml",
            "k8s/pvc.yaml"
        ]
        
        for config in configs:
            cmd = [
                self.kubectl, "apply",
                "-f", config,
                "-n", self.namespace
            ]
            
            subprocess.run(cmd, check=True)
    
    def _deploy_app(self):
        """部署应用"""
        resources = [
            "k8s/deployment.yaml",
            "k8s/service.yaml",
            "k8s/ingress.yaml",
            "k8s/hpa.yaml"
        ]
        
        for resource in resources:
            cmd = [
                self.kubectl, "apply",
                "-f", resource,
                "-n", self.namespace
            ]
            
            subprocess.run(cmd, check=True)
    
    def _wait_for_ready(self):
        """等待就绪"""
        cmd = [
            self.kubectl, "rollout", "status",
            "deployment/openhands",
            "-n", self.namespace,
            "--timeout=300s"
        ]
        
        subprocess.run(cmd, check=True)
    
    def _health_check(self):
        """健康检查"""
        # 获取服务 URL
        cmd = [
            self.kubectl, "get", "ingress",
            "openhands-ingress",
            "-n", self.namespace,
            "-o", "jsonpath={.spec.rules[0].host}"
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        host = result.stdout
        
        # 健康检查
        import requests
        response = requests.get(f"https://{host}/health")
        
        if response.status_code != 200:
            raise Exception("Health check failed")
    
    def rollback(self):
        """回滚"""
        cmd = [
            self.kubectl, "rollout", "undo",
            "deployment/openhands",
            "-n", self.namespace
        ]
        
        subprocess.run(cmd, check=True)
        print("✅ Rollback successful")
    
    def scale(self, replicas: int):
        """扩展"""
        cmd = [
            self.kubectl, "scale",
            "deployment/openhands",
            f"--replicas={replicas}",
            "-n", self.namespace
        ]
        
        subprocess.run(cmd, check=True)
        print(f"✅ Scaled to {replicas} replicas")

# 使用
deployer = KubernetesDeployer(namespace="openhands")
deployer.deploy(environment="production")

# 回滚
deployer.rollback()

# 扩展
deployer.scale(replicas=5)
```

---

## 🔗 服务网格

### Istio 配置

```yaml
# istio/virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: openhands
  namespace: openhands
spec:
  hosts:
  - api.openhands.dev
  gateways:
  - openhands-gateway
  http:
  - route:
    - destination:
        host: openhands
        port:
          number: 80
      weight: 90
    - destination:
        host: openhands-canary
        port:
          number: 80
      weight: 10
```

```yaml
# istio/destination-rule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: openhands
  namespace: openhands
spec:
  host: openhands
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: UPGRADE
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
    tls:
      mode: ISTIO_MUTUAL
  subsets:
  - name: v1
    labels:
      version: v1.0.0
  - name: v2
    labels:
      version: v2.0.0
```

---

## 🎯 云原生最佳实践

### 1. 容器最佳实践

```dockerfile
# Dockerfile 最佳实践
# 使用多阶段构建
FROM python:3.11-slim as builder

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 生产镜像
FROM python:3.11-slim

WORKDIR /app

# 复制依赖
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages

# 复制应用
COPY . .

# 非 root 用户
RUN useradd -m -u 1000 appuser
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# 启动应用
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 2. Kubernetes 最佳实践

- **资源限制** - 设置合理的 CPU 和内存限制
- **健康检查** - 实现 liveness 和 readiness 探针
- **自动扩展** - 使用 HPA 自动扩展
- **滚动更新** - 使用滚动更新零停机部署
- **配置分离** - 使用 ConfigMap 和 Secret

### 3. 监控最佳实践

- **四大黄金信号** - 延迟、流量、错误、饱和度
- **分布式追踪** - 使用 Jaeger 或 Zipkin
- **日志聚合** - 使用 ELK 或 Loki
- **告警** - 设置合理的告警规则

---

<div align="center">
  <p>☁️ 云原生，现代化部署！</p>
</div>
