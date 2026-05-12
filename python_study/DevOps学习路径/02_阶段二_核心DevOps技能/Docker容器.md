# Docker容器

容器的概念：轻量级的虚拟化，目的和虚拟机一样，创造“隔离环境”。

虚拟机是操作系统级别的隔离，容器是进程级别的隔离

虚拟机占用空间大，启动与初始化慢

容器轻量化，资源占用少，启动快，性能好

创建敏捷，版本控制，

缺点：复杂性增加，原生Linux支持，

## 一、Docker基础

### 1. Docker简介

- **Docker**：开源的容器化平台
- **特点**：
  - 轻量级：共享主机内核
  - 标准化：统一的容器格式
  - 可移植：一次构建，到处运行
  - 隔离性：资源隔离和安全
  - 自动化：简化部署和扩展

### 2. Docker核心概念

- **镜像（Image）**：只读模板，包含运行应用所需的所有内容
- **容器（Container）**：镜像的运行实例
- **仓库（Repository）**：存储镜像的地方
- **Dockerfile**：定义如何构建镜像的文本文件
- **Docker Compose**：定义和运行多容器应用的工具

### 3. Docker架构

- **Docker Engine**：Docker的核心组件
  - 服务器（dockerd）：守护进程，管理容器
  - REST API：与守护进程通信的接口
  - 命令行工具（docker）：用户交互界面
- **Registry**：存储和分发镜像的服务
  - Docker Hub：公共镜像仓库
  - 私有Registry：企业内部使用

## 二、Docker安装

### 1. Linux安装

#### 1.1 Ubuntu/Debian

```bash
# 1. 更新包列表并安装依赖
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common

# 2. 添加Docker GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 3. 添加Docker仓库
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# 4. 修改仓库地址文件（使用国内镜像源，可选）
sudo tee /etc/apt/sources.list.d/docker.list << 'EOF'
deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable
EOF

# 5. 安装Docker
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io

# 6. 启动Docker服务并设置开机自启
sudo systemctl start docker && sudo systemctl enable docker

# 7. 添加用户到docker组
sudo usermod -aG docker $USER && newgrp docker
```

#### 1.2 CentOS/RHEL

```bash
# 1. 安装依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 2. 添加Docker仓库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 3. 修改仓库地址文件（使用国内镜像源，可选）
sudo sed -i 's|download.docker.com|mirrors.aliyun.com/docker-ce|g' /etc/yum.repos.d/docker-ce.repo

# 4. 安装Docker
sudo yum install -y docker-ce docker-ce-cli containerd.io

# 5. 启动Docker服务并设置开机自启
sudo systemctl start docker && sudo systemctl enable docker

# 6. 添加用户到docker组
sudo usermod -aG docker $USER && newgrp docker
```

### 2. Windows安装

- 下载Docker Desktop for Windows
- 安装并启动Docker Desktop
- 启用WSL 2或Hyper-V
- 验证安装：`docker --version`

### 3. macOS安装

- 下载Docker Desktop for Mac
- 安装并启动Docker Desktop
- 验证安装：`docker --version`

### 4. 验证安装

```bash
# 查看Docker版本
docker --version

# 运行Hello World容器
docker run hello-world

# 查看Docker信息
docker info
```

## 三、Docker基本操作

### 1. 镜像操作

#### 1.1 搜索镜像

docker配置文件修改

sudo vim /etc/docker/daemon.json

```bash
docker search ubuntu
```

#### 1.2 拉取镜像

```bash
docker pull ubuntu:20.04
```

#### 1.3 查看镜像

```bash
docker images
docker image ls
```

#### 1.4 删除镜像

```bash
# 删除指定镜像
docker rmi ubuntu:20.04

# 强制删除镜像
docker rmi -f ubuntu:20.04

# 删除所有未使用的镜像
docker image prune
```

### 2. 容器操作

#### 2.1 运行容器

```bash
# 运行交互式容器
docker run -it ubuntu:20.04 /bin/bash

# 运行后台容器
docker run -d --name my-container ubuntu:20.04 sleep 3600

# 映射端口
docker run -d -p 8080:80 --name web nginx

# 挂载卷
docker run -d -v /host/path:/container/path --name data-container ubuntu:20.04

# 设置环境变量
docker run -d -e "ENV_VAR=value" --name env-container ubuntu:20.04
```

#### 2.2 查看容器

```bash
# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 查看容器详情
docker inspect my-container

# 查看容器日志
docker logs my-container

# 实时查看日志
docker logs -f my-container
```

#### 2.3 操作容器

```bash
# 启动容器
docker start my-container

# 停止容器
docker stop my-container

# 重启容器
docker restart my-container

# 进入容器
docker exec -it my-container /bin/bash

# 复制文件
docker cp my-container:/path/to/file /host/path

docker cp /host/path my-container:/path/to/directory

# 删除容器
docker rm my-container

# 强制删除运行中的容器
docker rm -f my-container

# 删除所有停止的容器
docker container prune
```

### 3. 网络操作

#### 3.1 查看网络

```bash
docker network ls
```

#### 3.2 创建网络

```bash
# 创建bridge网络
docker network create my-network

# 创建overlay网络（用于Swarm）
docker network create -d overlay swarm-network
```

#### 3.3 连接容器到网络

```bash
docker run -d --name container1 --network my-network nginx

# 连接已运行的容器
docker network connect my-network container2
```

#### 3.4 网络类型

- **bridge**：默认网络，容器间通过IP通信
- **host**：共享主机网络命名空间
- **none**：无网络
- **overlay**：跨主机网络，用于Docker Swarm
- **macvlan**：为容器分配MAC地址

### 4. 存储操作

#### 4.1 卷管理

```bash
# 创建卷
docker volume create my-volume

# 查看卷
docker volume ls

# 查看卷详情
docker volume inspect my-volume

# 删除卷
docker volume rm my-volume

# 删除未使用的卷
docker volume prune
```

#### 4.2 绑定挂载

```bash
docker run -d -v /host/path:/container/path:ro nginx
```

#### 4.3 tmpfs挂载

```bash
docker run -d --tmpfs /tmp:rw,noexec,nosuid,size=64m nginx
```

## 四、Dockerfile

### 1. Dockerfile基础

- **位置**：存储在项目根目录
- **语法**：指令+参数
- **指令执行**：从上到下执行
- **构建上下文**：Dockerfile所在目录的所有文件

### 2. 常用指令

#### 2.1 基础指令

- **FROM**：指定基础镜像
  ```dockerfile
  FROM ubuntu:20.04
  ```
- **MAINTAINER**：维护者信息
  ```dockerfile
  MAINTAINER Your Name <your.email@example.com>
  ```
- **LABEL**：添加元数据
  ```dockerfile
  LABEL version="1.0"
  LABEL description="My Application"
  ```

#### 2.2 构建指令

- **RUN**：执行命令
  ```dockerfile
  RUN apt update && apt install -y nginx
  ```
- **COPY**：复制文件
  ```dockerfile
  COPY . /app
  ```
- **ADD**：复制文件（支持URL和压缩文件）
  ```dockerfile
  ADD https://example.com/file.tar.gz /app
  ```
- **WORKDIR**：设置工作目录
  ```dockerfile
  WORKDIR /app
  ```
- **ARG**：定义构建参数
  ```dockerfile
  ARG VERSION=1.0
  RUN echo "Building version $VERSION"
  ```
- **ENV**：设置环境变量
  ```dockerfile
  ENV NODE_ENV=production
  ```

#### 2.3 运行时指令

- **EXPOSE**：暴露端口
  ```dockerfile
  EXPOSE 80 443
  ```
- **CMD**：容器启动命令
  ```dockerfile
  CMD ["nginx", "-g", "daemon off;"]
  ```
- **ENTRYPOINT**：容器入口点
  ```dockerfile
  ENTRYPOINT ["java", "-jar"]
  CMD ["app.jar"]
  ```
- **VOLUME**：声明卷
  ```dockerfile
  VOLUME ["/data"]
  ```
- **USER**：设置用户
  ```dockerfile
  USER appuser
  ```

### 3. Dockerfile示例

#### 3.1 简单Web应用

```dockerfile
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

#### 3.2 Java应用

```dockerfile
FROM openjdk:11-jdk-slim

WORKDIR /app

COPY target/myapp.jar /app/app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 3.3 Python应用

```dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000
CMD ["python", "app.py"]
```

### 4. 构建镜像

```bash
# 构建镜像
docker build -t myapp:1.0 .

# 指定Dockerfile路径
docker build -t myapp:1.0 -f Dockerfile.prod .

# 使用构建参数
docker build --build-arg VERSION=1.1 -t myapp:1.1 .

# 查看构建历史
docker history myapp:1.0
```

## 五、Docker Compose

### 1. Docker Compose简介

- **Docker Compose**：定义和运行多容器Docker应用的工具
- **配置文件**：`docker-compose.yml`
- **特点**：
  - 定义多容器应用
  - 配置网络和存储
  - 一键启动和停止
  - 环境变量管理

### 2. 安装Docker Compose

#### 2.1 Linux安装

```bash
# 下载Docker Compose（使用国内镜像加速）
curl -L "https://ghproxy.com/https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 或使用其他国内镜像
# curl -L "https://hub.fastgit.xyz/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 赋予执行权限
chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

#### 2.2 Windows/macOS

- Docker Desktop已包含Docker Compose
- 验证安装：`docker-compose --version`

### 3. docker-compose.yml配置

#### 3.1 基本结构

```yaml
version: '3'

services:
  web:
    build: .
    ports:
      - "80:80"
    volumes:
      - ./web:/app
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_PORT=5432

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb

volumes:
  postgres_data:

networks:
  default:
    driver: bridge
```

#### 3.2 常用配置项

- **build**：构建镜像
- **image**：使用现有镜像
- **ports**：端口映射
- **volumes**：卷挂载
- **environment**：环境变量
- **depends\_on**：服务依赖
- **restart**：重启策略
- **networks**：网络配置
- **command**：覆盖默认命令

### 4. Docker Compose命令

#### 4.1 基本命令

```bash
# 启动服务
docker-compose up

# 后台启动
docker-compose up -d

# 构建并启动
docker-compose up --build

# 停止服务
docker-compose down

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs

# 查看单个服务日志
docker-compose logs web

# 实时查看日志
docker-compose logs -f

# 进入容器
docker-compose exec web /bin/bash

# 查看服务依赖
docker-compose top

# 验证配置
docker-compose config
```

#### 4.2 其他命令

```bash
# 构建服务
docker-compose build

# 拉取镜像
docker-compose pull

# 推送镜像
docker-compose push

# 运行命令
docker-compose run --rm web python manage.py migrate

# 停止服务
docker-compose stop

# 启动服务
docker-compose start

# 重启服务
docker-compose restart

# 删除停止的服务容器
docker-compose rm
```

## 六、Docker最佳实践

### 1. 镜像优化

- **使用轻量级基础镜像**：alpine、slim版本
- **最小化镜像层数**：合并RUN指令
- **使用多阶段构建**：减小最终镜像大小
- **清理临时文件**：在同一RUN指令中清理
- **使用.dockerignore**：排除不必要的文件

### 2. 安全最佳实践

- **使用官方镜像**：避免使用未知来源的镜像
- **定期更新镜像**：修复安全漏洞
- **最小化容器权限**：使用非root用户
- **限制容器资源**：CPU和内存限制
- **使用只读文件系统**：`--read-only`
- **扫描镜像漏洞**：使用Trivy、Clair等工具

### 3. 开发最佳实践

- **使用Docker Compose**：简化开发环境
- **挂载源码**：便于开发和调试
- **使用环境变量**：分离配置和代码
- **保持容器无状态**：使用卷存储数据
- **编写清晰的Dockerfile**：添加注释和版本控制

### 4. 生产最佳实践

- **使用固定版本标签**：避免使用latest标签
- **配置健康检查**：确保服务正常运行
- **使用编排工具**：Kubernetes、Docker Swarm
- **实现日志管理**：集中化日志收集
- **监控容器**：Prometheus、Grafana
- **备份数据**：定期备份卷数据

## 七、推荐学习资源

### 书籍

- 《Docker实战》
- 《Docker容器与容器云》
- 《Docker技术入门与实践》

### 在线资源

- [Docker官方文档](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Blog](https://www.docker.com/blog/)

### 视频教程

- [Docker入门教程](https://www.docker.com/get-started)
- [Docker Mastery](https://www.udemy.com/course/docker-mastery/)
- [Docker for Beginners](https://youtube.com/docker)

### 实践项目

- [Docker Samples](https://github.com/docker/labs)
- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)

## 八、实践项目

### 项目一：Docker化Web应用

- 创建Dockerfile
- 构建镜像
- 运行容器
- 配置网络和存储

### 项目二：多容器应用

- 使用Docker Compose
- 配置服务依赖
- 实现数据持久化
- 构建开发环境

### 项目三：微服务架构

- 拆分应用为多个服务
- 使用Docker Compose编排
- 实现服务间通信
- 配置负载均衡

## 九、总结

通过本章节的学习，你应该已经掌握了以下Docker技能：

1. **Docker基础**：
   - 镜像和容器管理
   - 网络配置
   - 存储管理
2. **Dockerfile**：
   - 编写Dockerfile
   - 构建优化
   - 多阶段构建
3. **Docker Compose**：
   - 配置多容器应用
   - 管理服务依赖
   - 环境配置
4. **最佳实践**：
   - 镜像优化
   - 安全实践
   - 生产部署

Docker已经成为现代应用部署的标准工具，掌握Docker将大大简化你的开发和部署流程。
