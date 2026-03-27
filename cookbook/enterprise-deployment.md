# 企业部署指南

> 在企业环境中部署和管理 OpenHands

---

## 📋 目录

- [架构设计](#架构设计)
- [Kubernetes 部署](#kubernetes-部署)
- [Docker 部署](#docker-部署)
- [安全配置](#安全配置)
- [监控和维护](#监控和维护)

---

## 🏗️ 架构设计

### 系统架构

```
┌─────────────────────────────────────────┐
│           Load Balancer (Nginx)         │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
┌───▼────┐              ┌────▼───┐
│ OpenHands API 1 │              │ OpenHands API 2 │
└───┬────┘              └────┬───┘
    │                         │
    └────────────┬────────────┘
                 │
         ┌───────▼────────┐
         │   PostgreSQL   │
         │    Database    │
         └────────────────┘
                 │
         ┌───────▼────────┐
         │  Redis Cache   │
         └────────────────┘
```

### 组件说明

| 组件 | 说明 | 推荐配置 |
|------|------|---------|
| **Load Balancer** | 负载均衡 | Nginx / HAProxy |
| **OpenHands API** | 核心服务 | 2+ 实例 |
| **PostgreSQL** | 数据库 | 主从复制 |
| **Redis** | 缓存 | 集群模式 |

---

## ☸️ Kubernetes 部署

### 1. 创建命名空间

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openhands
```

### 2. 创建 Secret

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: openhands-secrets
  namespace: openhands
type: Opaque
stringData:
  ANTHROPIC_API_KEY: "sk-ant-xxx"
  DATABASE_URL: "postgresql://user:pass@postgres:5432/openhands"
  REDIS_URL: "redis://redis:6379"
```

### 3. 部署 PostgreSQL

```yaml
# postgres.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: openhands
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: openhands
        - name: POSTGRES_USER
          value: openhands
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: openhands-secrets
              key: DATABASE_PASSWORD
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: openhands
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
```

### 4. 部署 Redis

```yaml
# redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: openhands
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: openhands
spec:
  selector:
    app: redis
  ports:
  - port: 6379
```

### 5. 部署 OpenHands API

```yaml
# openhands.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openhands-api
  namespace: openhands
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openhands-api
  template:
    metadata:
      labels:
        app: openhands-api
    spec:
      containers:
      - name: openhands
        image: openhands/openhands:latest
        ports:
        - containerPort: 3000
        env:
        - name: ANTHROPIC_API_KEY
          valueFrom:
            secretKeyRef:
              name: openhands-secrets
              key: ANTHROPIC_API_KEY
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: openhands-secrets
              key: DATABASE_URL
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: openhands-secrets
              key: REDIS_URL
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: openhands-api
  namespace: openhands
spec:
  selector:
    app: openhands-api
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

### 6. 配置 Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: openhands-ingress
  namespace: openhands
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - openhands.yourcompany.com
    secretName: openhands-tls
  rules:
  - host: openhands.yourcompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: openhands-api
            port:
              number: 80
```

### 7. 部署命令

```bash
# 创建命名空间
kubectl apply -f namespace.yaml

# 创建 Secret
kubectl apply -f secret.yaml

# 部署数据库
kubectl apply -f postgres.yaml

# 部署缓存
kubectl apply -f redis.yaml

# 部署 OpenHands
kubectl apply -f openhands.yaml

# 配置 Ingress
kubectl apply -f ingress.yaml

# 查看状态
kubectl get pods -n openhands
kubectl get services -n openhands
```

---

## 🐳 Docker 部署

### 1. Docker Compose 配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: openhands
      POSTGRES_USER: openhands
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - openhands-network

  redis:
    image: redis:7
    volumes:
      - redis_data:/data
    networks:
      - openhands-network

  openhands-api:
    image: openhands/openhands:latest
    ports:
      - "3000:3000"
    environment:
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      DATABASE_URL: postgresql://openhands:${DATABASE_PASSWORD}@postgres:5432/openhands
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    volumes:
      - ./data:/app/data
    networks:
      - openhands-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - openhands-api
    networks:
      - openhands-network

volumes:
  postgres_data:
  redis_data:

networks:
  openhands-network:
    driver: bridge
```

### 2. Nginx 配置

```nginx
# nginx.conf
upstream openhands_backend {
    server openhands-api:3000;
}

server {
    listen 80;
    server_name openhands.yourcompany.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name openhands.yourcompany.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    location / {
        proxy_pass http://openhands_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3. 启动命令

```bash
# 创建环境变量文件
cat > .env << EOF
ANTHROPIC_API_KEY=sk-ant-xxx
DATABASE_PASSWORD=secure_password
EOF

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

---

## 🔒 安全配置

### 1. RBAC 权限控制

```yaml
# rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: openhands-role
  namespace: openhands
rules:
- apiGroups: [""]
  resources: ["pods", "services", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update"]
```

### 2. 网络策略

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: openhands-network-policy
  namespace: openhands
spec:
  podSelector:
    matchLabels:
      app: openhands-api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: openhands
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    - podSelector:
        matchLabels:
          app: redis
```

### 3. Pod 安全策略

```yaml
# pod-security-policy.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: openhands-psp
spec:
  privileged: false
  runAsUser:
    rule: MustRunAsNonRoot
  seLinux:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
```

---

## 📊 监控和维护

### 1. Prometheus 监控

```yaml
# prometheus.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: openhands-monitor
  namespace: openhands
spec:
  selector:
    matchLabels:
      app: openhands-api
  endpoints:
  - port: web
    path: /metrics
    interval: 30s
```

### 2. Grafana 仪表板

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
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      }
    ]
  }
}
```

### 3. 日志收集

```yaml
# fluent-bit.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: openhands
data:
  fluent-bit.conf: |
    [INPUT]
      Name              tail
      Tag               openhands.*
      Path              /var/log/containers/openhands-*.log
      Parser            docker
      DB                /var/log/flb_openhands.db
      Mem_Buf_Limit     5MB

    [OUTPUT]
      Name              elasticsearch
      Match             openhands.*
      Host              elasticsearch
      Port              9200
      Index             openhands-logs
      Type              _doc
```

---

## 🔧 维护操作

### 1. 数据备份

```bash
#!/bin/bash
# backup.sh

# 备份 PostgreSQL
kubectl exec -n openhands postgres-pod -- pg_dump -U openhands openhands > backup_$(date +%Y%m%d).sql

# 备份 Redis
kubectl exec -n openhands redis-pod -- redis-cli BGSAVE
kubectl cp openhands/redis-pod:/data/dump.rdb redis_backup_$(date +%Y%m%d).rdb

# 上传到 S3
aws s3 cp backup_$(date +%Y%m%d).sql s3://your-bucket/backups/
aws s3 cp redis_backup_$(date +%Y%m%d).rdb s3://your-bucket/backups/
```

### 2. 滚动更新

```bash
# 更新镜像
kubectl set image deployment/openhands-api openhands=openhands/openhands:v2.0.0 -n openhands

# 查看更新状态
kubectl rollout status deployment/openhands-api -n openhands

# 回滚
kubectl rollout undo deployment/openhands-api -n openhands
```

### 3. 扩容缩容

```bash
# 扩容到 5 个副本
kubectl scale deployment openhands-api --replicas=5 -n openhands

# 自动扩容
kubectl autoscale deployment openhands-api --cpu-percent=70 --min=2 --max=10 -n openhands
```

---

## 📚 相关资源

- [Kubernetes 官方文档](https://kubernetes.io/docs)
- [Docker 官方文档](https://docs.docker.com)
- [OpenHands 企业文档](https://docs.openhands.dev/enterprise)

---

<div align="center">
  <p>🎉 在企业环境中部署 OpenHands！</p>
</div>
