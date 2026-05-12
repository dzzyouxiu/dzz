# CI/CD实践

## 一、CI/CD基础

### 1. 基本概念

- **CI (持续集成)**：频繁地将代码集成到主干分支，每次集成都通过自动化测试
- **CD (持续交付/部署)**：将集成后的代码自动部署到测试环境或生产环境

### 2. CI/CD的价值

- **减少集成风险**：频繁集成，早期发现问题
- **提高开发效率**：自动化测试和部署
- **加速交付速度**：快速迭代，持续部署
- **改善代码质量**：自动化测试确保代码质量

### 3. CI/CD流程

1. 代码提交
2. 自动构建
3. 自动化测试
4. 代码审查
5. 自动部署
6. 监控和反馈

## 二、Jenkins

### 1. Jenkins简介

- **Jenkins**：开源的持续集成/持续部署工具
- **特点**：
  - 可扩展的插件系统
  - 支持分布式构建
  - 丰富的集成能力
  - 开源免费

## **一、最快速的安装方式**

<br />

```
docker pull jenkins/jenkins:lts
docker run -d --name my-jenkins -p 8080:8080 jenkins/jenkins:lts

```

<br />

两行命令完成：

<br />

- **第一行**：拉取 Jenkins 官方 LTS（长期支持版）镜像
- **第二行**：后台启动容器，命名为 `my-jenkins`，映射 8080 端口

<br />

首次访问 `http://localhost:8080` 会提示安装向导。

<br />

## **二、生产级部署（持久化数据 + 权限优化）**

<br />

生产环境必须持久化数据，否则容器重启会丢失所有配置和任务：

<br />

```
# 创建数据目录
mkdir -p ~/jenkins/home

# 设置正确的 UID（Jenkins 容器默认使用 jenkins 用户，UID 为 1000）
sudo chown -R 1000:1000 ~/jenkins/home

# 启动容器（挂载数据卷）
docker run -d \
  --name my-jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v ~/jenkins/home:/var/jenkins_home \
  --restart=always \
  jenkins/jenkins:lts

```

<br />

**关键参数说明**：

<br />

- `-v ~/jenkins/home:/var/jenkins_home`：持久化 Jenkins 主目录
- `-p 50000:50000`：Jenkins agent 连接端口（分布式构建需要）
- `chown 1000:1000`：避免权限问题（容器内 Jenkins 用户 UID 为 1000）
- `--restart=always`：异常退出自动重启

<br />

## **三、获取初始管理员密码**

<br />

首次启动后，需要获取初始密码进行解锁：

<br />

```
# 方式 1：查看容器日志
docker logs my-jenkins

# 方式 2：直接读取文件
docker exec my-jenkins cat /var/jenkins_home/secrets/initialAdminPassword

```

<br />

访问 `http://localhost:8080`，粘贴密码完成初始化设置。

<br />

## **四、常用管理命令**

<br />

```
# 查看容器状态
docker ps -a | grep jenkins

```

### 2. Jenkins安装

#### 2.1 Docker安装

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

#### 2.2 本地安装

- **CentOS**：
  ```bash
  # 安装Java
  yum install -y java-11-openjdk

  # 添加Jenkins仓库
  wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

  # 安装Jenkins
  yum install -y jenkins

  # 启动Jenkins
  systemctl start jenkins
  systemctl enable jenkins
  ```
- **Ubuntu**：
  ```bash
  # 安装Java
  apt update
  apt install -y openjdk-11-jdk

  # 添加Jenkins仓库
  wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | apt-key add -
  sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

  # 安装Jenkins
  apt update
  apt install -y jenkins

  # 启动Jenkins
  systemctl start jenkins
  systemctl enable jenkins
  ```

### 3. Jenkins初始化

1. **访问Jenkins**：<http://localhost:8080>
2. **获取初始密码**：
   ```bash
   # Docker安装
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

   # 本地安装
   cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
3. **安装推荐插件**
4. **创建管理员用户**
5. **配置实例**

### 4. Jenkins Pipeline

#### 4.1 Pipeline基础

- **Pipeline**：Jenkins的工作流定义方式
- **Jenkinsfile**：定义Pipeline的文件，存储在代码仓库中

#### 4.2 Jenkinsfile示例

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'mvn test'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                sh 'scp target/app.war user@server:/path/to/deploy'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            mail to: 'admin@example.com',
                 subject: 'Pipeline failed',
                 body: 'The pipeline for ${env.JOB_NAME} failed!'
        }
    }
}
```

#### 4.3 多分支Pipeline

- **功能**：为每个分支自动创建Pipeline
- **配置**：
  1. 新建项目 → 多分支Pipeline
  2. 配置源码管理
  3. 配置分支源
  4. 保存并应用

### 5. Jenkins插件

#### 5.1 常用插件

- **Git**：Git版本控制
- **Pipeline**：Pipeline支持
- **Docker**：Docker集成
- **Kubernetes**：Kubernetes集成
- **Blue Ocean**：现代化UI
- **Credentials Binding**：凭证管理
- **Email Extension**：邮件通知
- **Slack Notification**：Slack通知

#### 5.2 安装插件

1. 管理Jenkins → 插件管理 → 可用插件
2. 搜索并安装所需插件
3. 重启Jenkins

### 6. Jenkins与Git集成

#### 6.1 配置Git凭证

1. 管理Jenkins → 凭证 → 系统 → 全局凭证 → 添加凭证
2. 选择SSH Username with private key或Username with password
3. 填写凭证信息

#### 6.2 配置Git仓库

1. 新建项目 → 自由风格的项目
2. 源码管理 → Git
3. 填写仓库URL和凭证
4. 配置分支构建

## 三、GitLab CI

### 1. GitLab CI简介

- **GitLab CI**：GitLab内置的CI/CD工具
- **特点**：
  - 与GitLab无缝集成
  - 基于YAML配置
  - 支持并行执行
  - 强大的缓存机制

### 2. GitLab CI配置

#### 2.1 .gitlab-ci.yml文件

- **位置**：存储在代码仓库根目录
- **语法**：YAML格式

#### 2.2 基本配置示例

```yaml
stages:
  - build
  - test
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

cache:
  paths:
    - .m2/repository/
    - target/

build:
  stage: build
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.war

 test:
  stage: test
  script:
    - mvn test

 deploy:
  stage: deploy
  script:
    - scp target/*.war user@server:/path/to/deploy
  environment:
    name: production
  only:
    - master
```

### 3. GitLab Runner

#### 3.1 Runner简介

- **Runner**：执行CI/CD作业的代理
- **类型**：
  - 共享Runner
  - 群组Runner
  - 项目Runner

#### 3.2 安装Runner

- **Linux**：
  ```bash
  # 添加GitLab Runner仓库
  curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash

  # 安装Runner
  sudo yum install gitlab-runner

  # 注册Runner
  sudo gitlab-runner register
  ```
- **Docker**：
  ```bash
  docker run -d --name gitlab-runner --restart always \
    -v /srv/gitlab-runner/config:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:latest

  # 注册Runner
  docker exec -it gitlab-runner gitlab-runner register
  ```

#### 3.3 注册Runner

1. 访问GitLab项目 → 设置 → CI/CD → Runners
2. 复制注册令牌
3. 执行注册命令
4. 填写相关信息

## 四、GitHub Actions

### 1. GitHub Actions简介

- **GitHub Actions**：GitHub的CI/CD工具
- **特点**：
  - 与GitHub无缝集成
  - 基于YAML配置
  - 丰富的市场动作
  - 支持矩阵构建

### 2. GitHub Actions配置

#### 2.1 工作流文件

- **位置**：`.github/workflows/`目录
- **命名**：`*.yml`或`*.yaml`文件

#### 2.2 基本配置示例

```yaml
name: CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
        cache: maven
    
    - name: Build with Maven
      run: mvn clean package
    
    - name: Test with Maven
      run: mvn test
    
    - name: Deploy to Server
      if: github.ref == 'refs/heads/master'
      run: |
        scp target/*.war user@server:/path/to/deploy
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### 3. GitHub Actions市场

- **位置**：[GitHub Marketplace](https://github.com/marketplace?type=actions)
- **常用动作**：
  - `actions/checkout`：检出代码
  - `actions/setup-node`：设置Node.js环境
  - `actions/setup-python`：设置Python环境
  - `actions/setup-java`：设置Java环境
  - `actions/upload-artifact`：上传构建产物
  - `actions/download-artifact`：下载构建产物

### 4.  secrets管理

- **位置**：GitHub项目 → Settings → Secrets and variables → Actions
- **用途**：存储敏感信息，如API密钥、SSH私钥等
- **使用**：在工作流中通过`${{ secrets.SECRET_NAME }}`使用

## 五、CI/CD最佳实践

### 1. 代码质量

- **静态代码分析**：SonarQube集成
- **代码覆盖率**：Jacoco集成
- **依赖检查**：OWASP Dependency Check

### 2. 安全扫描

- **SAST**：静态应用安全测试
- **DAST**：动态应用安全测试
- **容器扫描**：Trivy集成

### 3. 部署策略

- **蓝绿部署**：最小化 downtime
- **金丝雀部署**：逐步发布
- **滚动部署**：逐步更新

### 4. 监控和反馈

- **构建状态**：集成到代码仓库
- **部署通知**：Slack、Email通知
- **监控集成**：Prometheus、Grafana

### 5. 性能优化

- **并行执行**：同时运行多个作业
- **缓存**：缓存依赖和构建产物
- **矩阵构建**：同时测试多个版本

## 六、推荐学习资源

### 书籍

- 《持续交付》
- 《DevOps实践指南》
- 《Jenkins权威指南》

### 在线资源

- [Jenkins官方文档](https://www.jenkins.io/doc/)
- [GitLab CI/CD文档](https://docs.gitlab.com/ee/ci/)
- [GitHub Actions文档](https://docs.github.com/en/actions)

### 视频教程

- [Jenkins入门教程](https://www.youtube.com/results?search_query=jenkins+tutorial)
- [GitLab CI/CD教程](https://www.youtube.com/results?search_query=gitlab+ci+tutorial)
- [GitHub Actions教程](https://www.youtube.com/results?search_query=github+actions+tutorial)

### 实践项目

- [Jenkins Pipeline示例](https://github.com/jenkinsci/pipeline-examples)
- [GitLab CI示例](https://gitlab.com/gitlab-org/gitlab-ci.yml)
- [GitHub Actions示例](https://github.com/actions/starter-workflows)

## 七、总结

通过本章节的学习，你应该已经掌握了以下CI/CD技能：

1. **Jenkins**：
   - 安装配置
   - Pipeline编写
   - 插件管理
   - 多分支Pipeline
2. **GitLab CI**：
   - .gitlab-ci.yml配置
   - Runner安装注册
   - 环境变量和缓存
3. **GitHub Actions**：
   - 工作流配置
   - 市场动作使用
   - Secrets管理
4. **CI/CD最佳实践**：
   - 代码质量
   - 安全扫描
   - 部署策略
   - 监控和反馈

CI/CD是DevOps的核心实践之一，掌握CI/CD工具将大大提高你的开发和部署效率。
