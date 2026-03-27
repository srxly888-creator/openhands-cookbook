# OpenHands 企业级部署案例

> 2026 年最新企业部署实践

---

## 📋 目录

- [案例 1：Kubernetes 部署](#案例-1kubernetes-部署)
- [案例 2：Docker Swarm 部署](#案例-2docker-swarm-部署)
- [案例 3：混合云部署](#案例-3混合云部署)

---

## 🏗️ 案例 1：Kubernetes 部署

### 架构图

```
┌─────────────────────────────────────────┐
│           Load Balancer (Nginx)         │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
┌───▼────┐              ┌────▼───┐
│ OpenHands-1 │              │ OpenHands-2 │
└───┬────┘              └────┬───┘
    │                         │
    └────────────┬────────────┘
                 │
         ┌───────▼────────┐
         │   PostgreSQL   │
         └────────────────┘
```

### 部署配置

**1. Namespace**
```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openhands
```

**2. ConfigMap**
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openhands-config
  namespace: openhands
data:
  DATABASE_URL: "postgresql://user:pass@postgres:5432/openhands"
  REDIS_URL: "redis://redis:6379"
  LOG_LEVEL: "INFO"
```

**3. Secret**
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
  DATABASE_PASSWORD: "secure_password"
```

**4. Deployment**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openhands
  namespace: openhands
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openhands
  template:
    metadata:
      labels:
        app: openhands
    spec:
      containers:
      - name: openhands
        image: openhands/openhands:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: openhands-config
        - secretRef:
            name: openhands-secrets
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
```

**5. Service**
```yaml
# service.yaml
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
    targetPort: 3000
  type: LoadBalancer
```

**6. Ingress**
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
            name: openhands
            port:
              number: 80
```

### 部署命令

```bash
# 创建命名空间
kubectl apply -f namespace.yaml

# 创建配置
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml

# 部署应用
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml

# 查看状态
kubectl get pods -n openhands
kubectl get services -n openhands
```

---

## 🐳 案例 2：Docker Swarm 部署

### docker-compose.yml

```yaml
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
    deploy:
      placement:
        constraints:
          - node.role == manager

  redis:
    image: redis:7
    volumes:
      - redis_data:/data
    networks:
      - openhands-network
    deploy:
      replicas: 1

  openhands:
    image: openhands/openhands:latest
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://openhands:${DATABASE_PASSWORD}@postgres:5432/openhands
      REDIS_URL: redis://redis:6379
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
    depends_on:
      - postgres
      - redis
    networks:
      - openhands-network
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        max_attempts: 3

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - openhands
    networks:
      - openhands-network
    deploy:
      placement:
        constraints:
          - node.role == manager

volumes:
  postgres_data:
  redis_data:

networks:
  openhands-network:
    driver: overlay
```

### 部署命令

```bash
# 初始化 Swarm
docker swarm init

# 部署服务
docker stack deploy -c docker-compose.yml openhands

# 查看服务
docker service ls

# 查看日志
docker service logs openhands_openhands
```

---

## ☁️ 案例 3：混合云部署

### 架构

```
┌─────────────────────────────────────────┐
│            AWS Cloud                    │
│  ┌─────────────────────────────────┐   │
│  │   OpenHands Production         │   │
│  │   - EKS Cluster                │   │
│  │   - RDS PostgreSQL             │   │
│  │   - ElastiCache Redis          │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
                 │
          VPN Connection
                 │
┌─────────────────────────────────────────┐
│          On-Premise                     │
│  ┌─────────────────────────────────┐   │
│  │   OpenHands Development        │   │
│  │   - Docker Compose             │   │
│  │   - Local PostgreSQL           │   │
│  │   - Local Redis                │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### AWS 配置

**EKS 集群**:
```bash
# 创建 EKS 集群
eksctl create cluster \
  --name openhands-cluster \
  --region us-east-1 \
  --nodes 3 \
  --node-type t3.large

# 配置 kubectl
aws eks update-kubeconfig \
  --name openhands-cluster \
  --region us-east-1
```

**RDS PostgreSQL**:
```bash
# 创建 RDS 实例
aws rds create-db-instance \
  --db-instance-identifier openhands-db \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --master-username openhands \
  --master-user-password ${DATABASE_PASSWORD} \
  --allocated-storage 100
```

**ElastiCache Redis**:
```bash
# 创建 Redis 集群
aws elasticache create-replication-group \
  --replication-group-id openhands-redis \
  --replication-group-description "OpenHands Redis" \
  --engine redis \
  --cache-node-type cache.t3.medium \
  --num-cache-clusters 2
```

---

## 📊 成本分析

### AWS 月度成本

| 资源 | 配置 | 月度成本 |
|------|------|---------|
| **EKS 集群** | 3x t3.large | $210 |
| **RDS PostgreSQL** | db.t3.medium | $75 |
| **ElastiCache Redis** | cache.t3.medium | $60 |
| **Load Balancer** | ALB | $25 |
| **存储** | 100GB EBS | $10 |
| **网络** | 数据传输 | $50 |
| **总计** | - | **$430/月** |

---

## 🔒 安全最佳实践

### 1. 网络隔离

```yaml
# Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: openhands-network-policy
  namespace: openhands
spec:
  podSelector:
    matchLabels:
      app: openhands
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

### 2. RBAC

```yaml
# RBAC 配置
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

### 3. Secret 管理

```bash
# 使用 AWS Secrets Manager
aws secretsmanager create-secret \
  --name openhands/api-key \
  --secret-string "sk-ant-xxx"

# Kubernetes 集成
kubectl create secret generic openhands-secrets \
  --from-literal=ANTHROPIC_API_KEY=$(aws secretsmanager get-secret-value \
    --secret-id openhands/api-key \
    --query SecretString \
    --output text)
```

---

## 📈 监控和日志

### Prometheus 监控

```yaml
# prometheus.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    
    scrape_configs:
      - job_name: 'openhands'
        kubernetes_sd_configs:
          - role: pod
            namespaces:
              names:
                - openhands
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_label_app]
            action: keep
            regex: openhands
```

### ELK 日志

```yaml
# filebeat.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: logging
data:
  filebeat.yml: |
    filebeat.inputs:
    - type: container
      paths:
        - /var/log/containers/*openhands*.log
    
    output.elasticsearch:
      hosts: ["elasticsearch:9200"]
      index: "openhands-%{[kubernetes][namespace]}-%{+yyyy.MM.dd}"
```

---

<div align="center">
  <p>🏢 企业级部署，生产就绪！</p>
</div>
