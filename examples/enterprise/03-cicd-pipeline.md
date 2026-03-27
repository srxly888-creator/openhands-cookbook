# OpenHands CI/CD 流水线案例

> 从代码提交到生产部署的完整自动化流程

---

## 📋 目录

- [CI/CD 架构设计](#cicd-架构设计)
- [GitHub Actions 配置](#github-actions-配置)
- [GitLab CI 配置](#gitlab-ci-配置)
- [Jenkins Pipeline](#jenkins-pipeline)

---

## 🏗️ CI/CD 架构设计

### 完整流水线

```
┌─────────────┐
│  代码提交   │
└──────┬──────┘
       │
┌──────▼──────┐
│  代码检查   │ (Lint + Format)
└──────┬──────┘
       │
┌──────▼──────┐
│  单元测试   │ (pytest + coverage)
└──────┬──────┘
       │
┌──────▼──────┐
│  集成测试   │ (API + DB)
└──────┬──────┘
       │
┌──────▼──────┐
│  构建镜像   │ (Docker Build)
└──────┬──────┘
       │
┌──────▼──────┐
│  安全扫描   │ (Trivy + Snyk)
└──────┬──────┘
       │
┌──────▼──────┐
│  部署测试   │ (Staging)
└──────┬──────┘
       │
┌──────▼──────┐
│  部署生产   │ (Production)
└──────┬──────┘
       │
┌──────▼──────┐
│  监控告警   │ (Prometheus)
└─────────────┘
```

---

## 🐙 GitHub Actions 配置

### 完整工作流

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ========== 代码检查 ==========
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install black flake8 mypy isort
          pip install -r requirements.txt

      - name: Run Black
        run: black --check .

      - name: Run isort
        run: isort --check-only .

      - name: Run Flake8
        run: flake8 src tests

      - name: Run MyPy
        run: mypy src

  # ========== 单元测试 ==========
  test-unit:
    runs-on: ubuntu-latest
    needs: lint

    steps:
      - uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-asyncio

      - name: Run unit tests
        run: pytest tests/unit -v --cov=src --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml

  # ========== 集成测试 ==========
  test-integration:
    runs-on: ubuntu-latest
    needs: lint

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-asyncio

      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
          REDIS_URL: redis://localhost:6379
        run: pytest tests/integration -v

  # ========== 构建镜像 ==========
  build:
    runs-on: ubuntu-latest
    needs: [test-unit, test-integration]
    if: github.event_name == 'push'

    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v3

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ========== 安全扫描 ==========
  security:
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'push'

    steps:
      - uses: actions/checkout@v3

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # ========== 部署测试环境 ==========
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.ref == 'refs/heads/develop'

    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to staging
        run: |
          # 使用 Kubernetes 或 Docker Swarm
          kubectl set image deployment/openhands \
            openhands=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n staging

      - name: Wait for deployment
        run: |
          kubectl rollout status deployment/openhands -n staging --timeout=300s

      - name: Run smoke tests
        run: |
          pytest tests/smoke --env=staging

  # ========== 部署生产环境 ==========
  deploy-production:
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.ref == 'refs/heads/main'

    environment:
      name: production
      url: https://example.com

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to production
        run: |
          # 使用蓝绿部署或金丝雀发布
          kubectl set image deployment/openhands \
            openhands=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production

      - name: Wait for deployment
        run: |
          kubectl rollout status deployment/openhands -n production --timeout=300s

      - name: Run health check
        run: |
          curl -f https://example.com/health || exit 1

      - name: Notify deployment
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text":"✅ OpenHands deployed to production"}'

  # ========== 回滚机制 ==========
  rollback:
    runs-on: ubuntu-latest
    if: failure()

    steps:
      - name: Rollback deployment
        run: |
          kubectl rollout undo deployment/openhands -n production

      - name: Notify rollback
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text":"❌ OpenHands deployment rolled back"}'
```

---

## 🦊 GitLab CI 配置

### .gitlab-ci.yml

```yaml
stages:
  - lint
  - test
  - build
  - security
  - deploy

variables:
  REGISTRY: registry.gitlab.com
  IMAGE_NAME: $CI_REGISTRY_IMAGE

# ========== 代码检查 ==========
lint:
  stage: lint
  image: python:3.11
  script:
    - pip install black flake8 mypy isort
    - black --check .
    - isort --check-only .
    - flake8 src tests
    - mypy src

# ========== 单元测试 ==========
test-unit:
  stage: test
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pip install pytest pytest-cov
    - pytest tests/unit -v --cov=src --cov-report=xml
  coverage: '/TOTAL.+?(\d+%)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

# ========== 集成测试 ==========
test-integration:
  stage: test
  image: python:3.11
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_PASSWORD: postgres
    DATABASE_URL: postgresql://postgres:postgres@postgres:5432/test
    REDIS_URL: redis://redis:6379
  script:
    - pip install -r requirements.txt
    - pip install pytest pytest-asyncio
    - pytest tests/integration -v

# ========== 构建镜像 ==========
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - |
      if [ "$CI_COMMIT_BRANCH" == "main" ]; then
        docker tag $IMAGE_NAME:$CI_COMMIT_SHA $IMAGE_NAME:latest
        docker push $IMAGE_NAME:latest
      fi
  only:
    - main
    - develop

# ========== 安全扫描 ==========
security:
  stage: security
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL \
        $IMAGE_NAME:$CI_COMMIT_SHA
  allow_failure: true
  only:
    - main
    - develop

# ========== 部署测试环境 ==========
deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/openhands openhands=$IMAGE_NAME:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/openhands -n staging --timeout=300s
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# ========== 部署生产环境 ==========
deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/openhands openhands=$IMAGE_NAME:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/openhands -n production --timeout=300s
    - curl -f https://example.com/health || exit 1
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

---

## 🔧 Jenkins Pipeline

### Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        REGISTRY = 'registry.hub.docker.com'
        IMAGE_NAME = 'openhands/openhands'
        KUBECONFIG = credentials('kubeconfig')
    }

    stages {
        stage('Lint') {
            steps {
                sh '''
                    pip install black flake8 mypy isort
                    black --check .
                    isort --check-only .
                    flake8 src tests
                    mypy src
                '''
            }
        }

        stage('Test Unit') {
            steps {
                sh '''
                    pip install -r requirements.txt
                    pip install pytest pytest-cov
                    pytest tests/unit -v --cov=src --cov-report=xml
                '''
            }
            post {
                always {
                    junit 'test-results.xml'
                    cobertura coberturaReportFile: 'coverage.xml'
                }
            }
        }

        stage('Test Integration') {
            steps {
                sh '''
                    docker-compose -f docker-compose.test.yml up -d
                    pytest tests/integration -v
                    docker-compose -f docker-compose.test.yml down
                '''
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'docker-credentials') {
                        def image = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                        image.push()
                        if (env.BRANCH_NAME == 'main') {
                            image.push('latest')
                        }
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:${env.BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                    kubectl set image deployment/openhands openhands=${IMAGE_NAME}:${env.BUILD_NUMBER} -n staging
                    kubectl rollout status deployment/openhands -n staging --timeout=300s
                '''
            }
        }

        stage('Deploy Production') {
            when {
                branch 'main'
            }
            steps {
                input 'Deploy to production?'
                sh '''
                    kubectl set image deployment/openhands openhands=${IMAGE_NAME}:${env.BUILD_NUMBER} -n production
                    kubectl rollout status deployment/openhands -n production --timeout=300s
                    curl -f https://example.com/health || exit 1
                '''
            }
        }
    }

    post {
        failure {
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Pipeline failed: ${env.BUILD_URL}"
        }
        success {
            mail to: 'team@example.com',
                 subject: "Success Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Pipeline succeeded: ${env.BUILD_URL}"
        }
    }
}
```

---

## 📊 监控和告警

### Prometheus 监控

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'openhands'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        action: keep
        regex: openhands

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'
```

### 告警规则

```yaml
# alerts.yml
groups:
  - name: openhands
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: High error rate detected
          description: Error rate is {{ $value }} requests/s

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: High latency detected
          description: P95 latency is {{ $value }}s

      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Pod is crash looping
          description: Pod {{ $labels.pod }} is restarting
```

---

## 🎯 最佳实践

### 1. 分支策略
- `main` - 生产环境
- `develop` - 测试环境
- `feature/*` - 功能分支
- `hotfix/*` - 紧急修复

### 2. 部署策略
- **蓝绿部署** - 零停机部署
- **金丝雀发布** - 逐步滚动
- **滚动更新** - 逐个替换

### 3. 回滚策略
- 自动回滚（健康检查失败）
- 手动回滚（紧急情况）
- 回滚通知（Slack/邮件）

### 4. 安全策略
- 镜像扫描（Trivy）
- 依赖检查（Snyk）
- 密钥管理（Vault）
- 最小权限（RBAC）

---

<div align="center">
  <p>🚀 CI/CD 流水线，自动化部署！</p>
</div>
