# 项目1：Web应用完整部署

## 项目概述

构建一个完整的Web应用DevOps流水线，从代码提交到生产部署全自动化。

## 技术栈

```
应用层：Python Flask / Node.js Express
数据层：PostgreSQL / Redis
代理层：Nginx
缓存：Redis
消息：RabbitMQ（可选）
```

## 基础设施

```
Git → CI/CD → Docker Build → Registry → K8s Deploy
                              ↓
                    Prometheus + Grafana
                              ↓
                         ELK / Loki
```

## 第一步：创建应用代码

### Flask应用示例

```python
# app.py
from flask import Flask, jsonify, request
import redis
import os
import time

app = Flask(__name__)

redis_host = os.getenv('REDIS_HOST', 'localhost')
r = redis.Redis(host=redis_host, port=6379)

@app.route('/')
def index():
    return jsonify({
        'message': 'Hello from Flask!',
        'version': '1.0.0',
        'timestamp': time.time()
    })

@app.route('/health')
def health():
    try:
        r.ping()
        return jsonify({'status': 'healthy'}), 200
    except Exception as e:
        return jsonify({'status': 'unhealthy', 'error': str(e)}), 500

@app.route('/api/users')
def get_users():
    return jsonify([
        {'id': 1, 'name': 'Alice'},
        {'id': 2, 'name': 'Bob'}
    ])

@app.route('/metrics')
def metrics():
    from prometheus_client import generate_latest, CONTENT_TYPE_LATEST
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```python
# requirements.txt
Flask==3.0.0
redis==5.0.1
prometheus-client==0.19.0
gunicorn==21.2.0
```

### 测试代码

```python
# test_app.py
import pytest
from app import app as flask_app

@pytest.fixture
def app():
    yield flask_app

@pytest.fixture
def client(app):
    return app.test_client()

def test_index(client):
    response = client.get('/')
    assert response.status_code == 200
    data = response.get_json()
    assert 'message' in data

def test_health(client):
    response = client.get('/health')
    # 可能返回200或500（取决于Redis）
    assert response.status_code in [200, 500]
```

## 第二步：Docker化

### Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim-bookworm

WORKDIR /app

RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m appuser
USER appuser

COPY --chown=appuser:appuser requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser . .

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:5000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

### Docker Compose（开发环境）

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - web

volumes:
  redis_data:
```

### Nginx配置

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://web:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        access_log off;
        proxy_pass http://web:5000;
    }
}
```

## 第三步：CI/CD配置

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - scan
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_NAME: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

cache:
  paths:
    - .venv/

test:
  stage: test
  image: python:3.11
  before_script:
    - pip install poetry
    - poetry install
  script:
    - poetry run pytest --cov=app --cov-report=xml
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

lint:
  stage: test
  image: python:3.11
  script:
    - pip install flake8
    - flake8 app.py --max-line-length=120

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_NAME .
    - docker push $IMAGE_NAME

security_scan:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy image --severity HIGH,CRITICAL --exit-code 1 $IMAGE_NAME

deploy_staging:
  stage: deploy
  image: alpine/helm:3.13.0
  script:
    - helm upgrade --install myapp ./helm \
        --namespace staging \
        --create-namespace \
        --set image.tag=$CI_COMMIT_SHORT_SHA
  only:
    - main

deploy_prod:
  stage: deploy
  image: alpine/helm:3.13.0
  script:
    - helm upgrade --install myapp ./helm \
        --namespace production \
        --create-namespace \
        --set image.tag=$CI_COMMIT_SHORT_SHA
  when: manual
  only:
    - main
```

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: pytest --cov=app --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: user/myapp:${{ github.sha }}

  security:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: user/myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
```

## 第四步：Kubernetes部署

### Helm Chart结构

```
helm/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── hpa.yaml
```

### Deployment

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-web
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "5000"
    spec:
      containers:
        - name: web
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 5000
          env:
            - name: REDIS_HOST
              value: {{ .Release.Name }}-redis
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

### Service

```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-web
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 5000
  selector:
    app: web
```

### Ingress

```yaml
# templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-web
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
    - hosts:
        - {{ .Values.ingress.host }}
      secretName: {{ .Release.Name }}-tls
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Release.Name }}-web
                port:
                  number: 80
```

### HPA（自动扩缩）

```yaml
# templates/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Release.Name }}-web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Release.Name }}-web
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### values.yaml

```yaml
# values.yaml
replicaCount: 2

image:
  repository: myregistry/myapp
  tag: latest

ingress:
  host: myapp.example.com

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

## 第五步：监控配置

### ServiceMonitor

```yaml
# servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-web
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: web
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

### 告警规则

```yaml
# alerts.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
spec:
  groups:
    - name: myapp
      rules:
        - alert: MyAppDown
          expr: up{job="myapp"} == 0
          for: 1m
          labels:
            severity: critical
          annotations:
            summary: "MyApp instance {{ $labels.instance }} is down"

        - alert: MyAppHighCPU
          expr: rate(container_cpu_usage_seconds_total{pod=~"myapp-.*"}[5m]) * 100 > 80
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "MyApp CPU usage high"
```

## 第六步：日志配置

### Filebeat Sidecar

```yaml
# 添加到deployment
containers:
  - name: filebeat
    image: elastic/filebeat:8.11.0
    volumeMounts:
      - name: logs
        mountPath: /var/log/myapp
      - name: filebeat-config
        mountPath: /usr/share/filebeat/filebeat.yml
        subPath: filebeat.yml
volumes:
  - name: logs
    emptyDir: {}
  - name: filebeat-config
    configMap:
      name: filebeat-config
```

### Grafana Dashboard

导入Dashboard ID：12900（Flask应用监控）或创建自定义Dashboard

## 项目交付清单

- [x] 应用代码（含测试）
- [x] Dockerfile
- [x] Docker Compose（开发环境）
- [x] CI/CD配置
- [x] Helm Chart
- [x] 监控配置（Prometheus规则）
- [x] 日志配置
- [x] README.md
- [x] 架构图
- [x] 运行手册

## 运行验证

```bash
# 本地运行
docker-compose up -d

# 访问应用
curl http://localhost

# 运行测试
pytest

# 部署到K8s
helm install myapp ./helm

# 查看Pod状态
kubectl get pods

# 查看日志
kubectl logs -l app=web

# 访问监控
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000
```
