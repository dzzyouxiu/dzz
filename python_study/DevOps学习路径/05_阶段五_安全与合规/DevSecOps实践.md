# DevSecOps实践

## DevSecOps简介

### 什么是DevSecOps
- 将安全集成到DevOps流程中
- 安全左移（Shift Left）
- 自动化安全检查
- 持续安全监控

### 安全集成点

```
开发 → 代码提交 → 构建 → 测试 → 部署 → 运行
 ↓        ↓        ↓      ↓       ↓       ↓
SAST    密钥检测   镜像扫描 DAST   配置检查  运行时安全
```

## 安全基础

### CIA三元组

| 要素 | 说明 | 措施 |
|------|------|------|
| 机密性 | 防止未授权访问 | 加密、访问控制 |
| 完整性 | 防止未授权修改 | 哈希、签名 |
| 可用性 | 确保服务可用 | 冗余、备份 |

### 身份认证

```bash
# SSH密钥认证
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id user@server

# 禁用密码登录
# /etc/ssh/sshd_config
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# 重启服务
systemctl restart sshd
```

### 防火墙配置

```bash
# UFW (Ubuntu)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status

# Firewalld (CentOS)
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --zone=public --add-service=ssh --permanent
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --reload
```

### SSL/TLS证书

```bash
# 使用Let's Encrypt
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com

# 自动续期
sudo certbot renew --dry-run

# 生成自签名证书
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/nginx-selfsigned.key \
  -out /etc/ssl/certs/nginx-selfsigned.crt
```

## 静态应用安全测试(SAST)

### SonarQube安装

```yaml
# docker-compose.yml
version: '3.8'

services:
  sonarqube:
    image: sonarqube:community
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    depends_on:
      - db

  db:
    image: postgres:12
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```

### SonarQube扫描

```bash
# 安装SonarScanner
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip
unzip sonar-scanner-cli-4.8.0.2856-linux.zip
export PATH=$PATH:/path/to/sonar-scanner/bin

# 配置sonar-project.properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0
sonar.sources=.
sonar.language=py
sonar.sourceEncoding=UTF-8
sonar.host.url=http://localhost:9000
sonar.login=admin
sonar.password=admin

# 执行扫描
sonar-scanner
```

### GitLab CI集成SonarQube

```yaml
sonarqube-check:
  image: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
    GIT_DEPTH: "0"
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - sonar-scanner
        -Dsonar.projectKey=$SONAR_PROJECT_KEY
        -Dsonar.sources=.
        -Dsonar.host.url=$SONAR_HOST_URL
        -Dsonar.login=$SONAR_TOKEN
  only:
    - main
```

## 依赖漏洞扫描

### 使用Snyk

```bash
# 安装Snyk CLI
npm install -g snyk
# 或
pip install snyk

# 认证
snyk auth

# 扫描npm项目
snyk test

# 扫描Python项目
snyk test --file=requirements.txt

# 监控项目
snyk monitor

# 修复漏洞
snyk wizard
```

### 使用Trivy

```bash
# 安装
wget https://github.com/aquasecurity/trivy/releases/download/v0.48.0/trivy_0.48.0_Linux-64bit.tar.gz
tar zxf trivy_0.48.0_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/

# 扫描文件系统
trivy fs /path/to/project

# 扫描特定文件
trivy fs --scanners vuln,config,secret requirements.txt

# 扫描Git仓库
trivy repo https://github.com/org/repo

# 输出JSON格式
trivy fs --format json --output results.json /path
```

### 检查Python依赖

```bash
# 使用safety
pip install safety
safety check --full-report

# 使用bandit（Python代码扫描）
pip install bandit
bandit -r myapp/ -f json -o bandit-report.json

# 使用pip-audit
pip install pip-audit
pip-audit
```

### 检查npm依赖

```bash
# 内置审计
npm audit
npm audit fix

# 使用depcheck检查未使用的依赖
npx depcheck

# 使用npm-check
npx npm-check
```

## 密钥检测

### GitGuardian

```bash
# 安装ggshield
pip install ggshield

# 认证
ggshield auth login

# 扫描本地仓库
ggshield secret scan repo /path/to/repo

# 扫描PR/MR
ggshield secret scan range --rev HEAD~10..HEAD

# 安装pre-commit钩子
ggshield install --mode local
```

### TruffleHog

```bash
# 安装
pip install truffleHog

# 扫描仓库
trufflehog --regex --entropy=False https://github.com/user/repo

# 扫描本地仓库
trufflehog file:///path/to/repo --json
```

### 清理Git历史

```bash
# 使用BFG Repo-Cleaner
java -jar bfg.jar --delete-files id_rsa my-repo.git
java -jar bfg.jar --replace-text passwords.txt my-repo.git

# 使用git-filter-repo
pip install git-filter-repo
git filter-repo --invert-paths --path id_rsa

# 创建密码替换文件
# passwords.txt
PASSWORD1:***REMOVED***
API_KEY:***REMOVED***
```

### .gitignore最佳实践

```
# 敏感文件
*.pem
*.key
*.p12
*.pfx
.secret
.env
.env.*
!.env.example

# IDE
.vscode/
.idea/
*.swp
*.swo

# 依赖
node_modules/
vendor/
__pycache__/

# 构建输出
dist/
build/
*.log
```

## 容器镜像安全

### Trivy扫描镜像

```bash
# 扫描镜像
trivy image nginx:latest

# 只显示高危和严重
trivy image --severity HIGH,CRITICAL nginx:latest

# 忽略特定漏洞
trivy image --ignorefile .trivyignore nginx:latest

# 输出HTML报告
trivy image --format template --template "@html.tpl" -o report.html nginx:latest

# .trivyignore
# 忽略的CVE
CVE-2023-1234
```

### Dockerfile最佳实践

```dockerfile
# 使用官方基础镜像
FROM python:3.11-slim-bookworm

# 更新系统包
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 创建非root用户
RUN useradd -m appuser
USER appuser

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY --chown=appuser:appuser requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY --chown=appuser:appuser . .

# 不要在镜像中存储密钥
# ENV API_KEY=xxx # ❌ 不要这样做

EXPOSE 8000

CMD ["python", "app.py"]
```

### 多阶段构建

```dockerfile
# 构建阶段
FROM golang:1.21 AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server

# 运行阶段（最小镜像）
FROM alpine:latest

RUN apk --no-cache add ca-certificates
COPY --from=builder /app/server /server

# 创建非root用户
RUN adduser -D appuser
USER appuser

CMD ["/server"]
```

### Anchore Grype

```bash
# 安装
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh

# 扫描镜像
grype nginx:latest

# 扫描目录
grype dir:/path/to/project

# 扫描SBOM
grype sbom:./sbom.json
```

## CI/CD中的安全集成

### GitLab CI完整安全流水线

```yaml
stages:
  - test
  - build
  - security

# SAST扫描
sast:
  stage: test
  image: sonarsource/sonar-scanner-cli:latest
  script:
    - sonar-scanner -Dsonar.projectKey=$PROJECT_KEY -Dsonar.host.url=$SONAR_URL -Dsonar.login=$SONAR_TOKEN

# 密钥检测
secrets:
  stage: test
  image: gitguardian/ggshield:latest
  script:
    - ggshield secret scan ci

# 依赖扫描
dependency_scan:
  stage: test
  image: python:3.11
  script:
    - pip install safety
    - safety check --full-report --output json > safety.json
  artifacts:
    paths:
      - safety.json

# 镜像扫描
image_scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --severity HIGH,CRITICAL --exit-code 1 $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

# DAST扫描（需要部署测试环境）
dast:
  stage: security
  image: owasp/zap2docker-stable
  script:
    - zap-baseline.py -t https://test.example.com -r zap-report.html
  artifacts:
    paths:
      - zap-report.html
```

### GitHub Actions安全工作流

```yaml
# .github/workflows/security.yml
name: Security Checks

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

      - name: Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Snyk Scan
        uses: snyk/actions/python@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      - name: GitGuardian scan
        uses: GitGuardian/ggshield-action@master
        env:
          GITHUB_PUSH_BEFORE_SHA: ${{ github.event.before }}
          GITHUB_PUSH_BASE_SHA: ${{ github.event.base }}
          GITHUB_PULL_BASE_SHA: ${{ github.event.pull_request.base.sha }}
          GITHUB_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}
          GG_API_KEY: ${{ secrets.GG_API_KEY }}
```

## 运行时安全

### Falco（Kubernetes运行时安全）

```yaml
# 安装Falco
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco

# 规则示例
# /etc/falco/falco_rules.local.yaml
- rule: Write below binary dir
  desc: an attempt to write to any file below a set of binary directories
  condition: >
    bin_dir and evt.dir = < and open_write
    and not package_mgmt_procs
    and not exe_running_docker_save
    and not python_running_get_pip
    and not python_running_ms_oms
    and not user_known_write_below_binary_dir_activities
  output: >
    File below a known binary directory opened for writing (user=%user.name
    command=%proc.cmdline file=%fd.name parent=%proc.pname pcmdline=%proc.pcmdline
    gparent=%proc.aname[2] ggparent=%proc.aname[3] gggparent=%proc.aname[4]
    container_id=%container.id image=%container.image.repository)
  priority: ERROR
  tags: [filesystem, mitre_persistence]
```

### OPA Gatekeeper

```yaml
# 策略示例：禁止使用latest标签
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sallowedrepos
spec:
  crd:
    spec:
      names:
        kind: K8sAllowedRepos
      validation:
        openAPIV3Schema:
          type: object
          properties:
            repos:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sallowedrepos
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          satisfied := [good | repo = input.parameters.repos[_] ; good = contains(container.image, repo)]
          not any(satisfied)
          msg := sprintf("container <%v> has an invalid image repo <%v>, allowed repos are %v", [input.review.object.metadata.name, container.image, input.parameters.repos])
        }
```

## 日志审计

### 安全事件日志

```bash
# 监控SSH登录
grep "Failed password" /var/log/auth.log
grep "Accepted" /var/log/auth.log

# 监控sudo使用
grep sudo /var/log/auth.log

# 使用auditd监控文件变化
sudo apt install auditd
sudo auditctl -w /etc/passwd -p rwxa -k passwd_changes

# 查看审计日志
sudo ausearch -k passwd_changes
```

### Fail2Ban

```bash
# 安装
sudo apt install fail2ban

# 配置
sudo cp /etc/fail2ban/jail.{conf,local}

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 22
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3
bantime = 3600

# 启动
sudo systemctl enable --now fail2ban

# 查看状态
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## 学习检查清单

- [ ] 理解DevSecOps概念
- [ ] 能够配置SSH和防火墙安全
- [ ] 能够使用SonarQube扫描代码
- [ ] 能够扫描依赖漏洞
- [ ] 能够检测Git中的密钥
- [ ] 理解Dockerfile安全最佳实践
- [ ] 能够扫描容器镜像漏洞
- [ ] 能够在CI/CD中集成安全检查
- [ ] 了解运行时安全工具
- [ ] 理解日志审计的重要性
