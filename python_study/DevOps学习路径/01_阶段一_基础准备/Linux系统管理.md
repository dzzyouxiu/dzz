# Linux系统管理

## 一、Linux基础

### 1. Linux简介
- **Linux**：开源的类Unix操作系统
- **发行版**：Ubuntu、CentOS、Debian、Red Hat等
- **文件系统**：ext4、xfs等
- **目录结构**：/bin、/etc、/home、/var等

### 2. 命令行基础
- **终端**：命令行界面
- **shell**：命令解释器（bash、sh、zsh等）
- **命令格式**：command [options] [arguments]
- **快捷键**：Tab补全、Ctrl+C终止、Ctrl+D退出

## 二、常用Linux命令

### 1. 文件管理命令
- **ls**：列出目录内容
  ```bash
  ls -la  # 详细列出所有文件
  ```
- **cd**：切换目录
  ```bash
  cd /home/user  # 切换到用户目录
  ```
- **pwd**：显示当前目录
  ```bash
  pwd  # 显示当前路径
  ```
- **mkdir**：创建目录
  ```bash
  mkdir -p dir1/dir2  # 递归创建目录
  ```
- **rm**：删除文件或目录
  ```bash
  rm -rf dir  # 强制递归删除目录
  ```
- **cp**：复制文件或目录
  ```bash
  cp -r src dest  # 递归复制目录
  ```
- **mv**：移动或重命名文件
  ```bash
  mv file1 file2  # 重命名文件
  ```
- **touch**：创建空文件
  ```bash
  touch file.txt  # 创建空文件
  ```

### 2. 文件查看命令
- **cat**：查看文件内容
  ```bash
  cat file.txt  # 查看文件内容
  ```
- **less**：分页查看文件
  ```bash
  less file.txt  # 分页查看
  ```
- **head**：查看文件开头
  ```bash
  head -n 10 file.txt  # 查看前10行
  ```
- **tail**：查看文件末尾
  ```bash
  tail -n 10 file.txt  # 查看后10行
  tail -f file.txt  # 实时查看文件变化
  ```
- **grep**：搜索文件内容
  ```bash
  grep "pattern" file.txt  # 搜索匹配内容
  grep -r "pattern" dir  # 递归搜索
  ```

### 3. 系统管理命令
- **ps**：查看进程
  ```bash
  ps aux  # 查看所有进程
  ps -ef | grep process  # 过滤进程
  ```
- **top**：实时查看系统状态
  ```bash
  top  # 实时监控
  ```
- **free**：查看内存使用情况
  ```bash
  free -h  # 人类可读格式
  ```
- **df**：查看磁盘使用情况
  ```bash
  df -h  # 人类可读格式
  ```
- **du**：查看目录大小
  ```bash
  du -sh dir  # 查看目录总大小
  ```
- **kill**：终止进程
  ```bash
  kill PID  # 终止进程
  kill -9 PID  # 强制终止
  ```

### 4. 网络命令
- **ifconfig**：查看网络接口
  ```bash
  ifconfig  # 查看网络接口
  ```
- **ip**：网络配置工具
  ```bash
  ip addr  # 查看IP地址
  ip route  # 查看路由表
  ```
- **ping**：测试网络连接
  ```bash
  ping google.com  # 测试连接
  ```
- **netstat**：查看网络状态
  ```bash
  netstat -tuln  # 查看监听端口
  ```
- **curl**：HTTP客户端
  ```bash
  curl http://example.com  # 获取网页内容
  ```
- **wget**：下载文件
  ```bash
  wget http://example.com/file  # 下载文件
  ```

### 5. 用户管理命令
- **useradd**：创建用户
  ```bash
  useradd username  # 创建用户
  ```
- **passwd**：修改密码
  ```bash
  passwd username  # 修改密码
  ```
- **userdel**：删除用户
  ```bash
  userdel -r username  # 删除用户及家目录
  ```
- **groupadd**：创建组
  ```bash
  groupadd groupname  # 创建组
  ```
- **id**：查看用户信息
  ```bash
  id username  # 查看用户ID和组
  ```

### 6. 权限管理命令
- **chmod**：修改文件权限
  ```bash
  chmod 755 file  # 设置权限
  chmod +x file  # 添加执行权限
  ```
- **chown**：修改文件所有者
  ```bash
  chown user:group file  # 修改所有者和组
  ```
- **chgrp**：修改文件组
  ```bash
  chgrp group file  # 修改组
  ```
- **umask**：设置默认权限
  ```bash
  umask 022  # 设置默认权限
  ```

## 三、系统配置

### 1. 网络配置
- **CentOS/RHEL**：
  ```bash
  # 编辑网络配置文件
  vi /etc/sysconfig/network-scripts/ifcfg-eth0
  ```
  ```
  TYPE=Ethernet
  BOOTPROTO=static
  NAME=eth0
  DEVICE=eth0
  ONBOOT=yes
  IPADDR=192.168.1.100
  NETMASK=255.255.255.0
  GATEWAY=192.168.1.1
  DNS1=8.8.8.8
  DNS2=8.8.4.4
  ```
  ```bash
  # 重启网络服务
  systemctl restart network
  ```

- **Ubuntu/Debian**：
  ```bash
  # 编辑网络配置文件
  vi /etc/netplan/00-installer-config.yaml
  ```
  ```yaml
  network:
    ethernets:
      eth0:
        addresses:
        - 192.168.1.100/24
        routes:
        - to: default
          via: 192.168.1.1
        nameservers:
          addresses:
          - 8.8.8.8
          - 8.8.4.4
    version: 2
  ```
  ```bash
  # 应用网络配置
  netplan apply
  ```

### 2. 防火墙配置
- **firewalld**（CentOS 7+）：
  ```bash
  # 查看状态
  systemctl status firewalld
  
  # 启动/停止/重启
  systemctl start firewalld
  systemctl stop firewalld
  systemctl restart firewalld
  
  # 开放端口
  firewall-cmd --permanent --add-port=80/tcp
  firewall-cmd --reload
  
  # 查看开放的端口
  firewall-cmd --list-ports
  ```

- **iptables**：
  ```bash
  # 查看规则
  iptables -L
  
  # 开放端口
  iptables -A INPUT -p tcp --dport 80 -j ACCEPT
  
  # 保存规则
  service iptables save
  ```

### 3. 服务管理
- **systemctl**（systemd）：
  ```bash
  # 查看服务状态
  systemctl status service
  
  # 启动/停止/重启服务
  systemctl start service
  systemctl stop service
  systemctl restart service
  
  # 设置开机自启/禁用
  systemctl enable service
  systemctl disable service
  
  # 查看服务列表
  systemctl list-unit-files --type=service
  ```

### 4. 包管理
- **yum**（CentOS/RHEL）：
  ```bash
  # 安装包
  yum install package
  
  # 卸载包
  yum remove package
  
  # 更新包
  yum update package
  
  # 查看包信息
  yum info package
  
  # 搜索包
  yum search package
  ```

- **apt**（Ubuntu/Debian）：
  ```bash
  # 更新包列表
  apt update
  
  # 安装包
  apt install package
  
  # 卸载包
  apt remove package
  
  # 更新包
  apt upgrade package
  
  # 查看包信息
  apt show package
  
  # 搜索包
  apt search package
  ```

## 四、Shell脚本编写

### 1. 脚本基础
- **脚本格式**：
  ```bash
  #!/bin/bash
  # 脚本注释
  echo "Hello World"
  ```

- **执行脚本**：
  ```bash
  # 赋予执行权限
  chmod +x script.sh
  
  # 执行脚本
  ./script.sh
  bash script.sh
  ```

### 2. 变量
- **定义变量**：
  ```bash
  var=value
  var="value with spaces"
  ```

- **使用变量**：
  ```bash
  echo $var
  echo ${var}suffix
  ```

- **环境变量**：
  ```bash
  export VAR=value
  echo $HOME
  echo $PATH
  ```

### 3. 特殊变量
- **$0**：脚本名称
- **$1-$9**：脚本参数
- **$@**：所有参数
- **$#**：参数个数
- **$?**：上一个命令的返回值
- **$$**：当前进程ID

### 4. 条件判断
- **if语句**：
  ```bash
  if [ condition ]; then
      commands
  elif [ condition ]; then
      commands
  else
      commands
  fi
  ```

- **条件表达式**：
  ```bash
  # 文件测试
  [ -f file ]  # 文件存在且是普通文件
  [ -d dir ]   # 目录存在
  [ -e path ]  # 路径存在
  [ -r file ]  # 文件可读
  [ -w file ]  # 文件可写
  [ -x file ]  # 文件可执行
  
  # 字符串测试
  [ "$a" = "$b" ]  # 字符串相等
  [ "$a" != "$b" ]  # 字符串不等
  [ -z "$a" ]  # 字符串为空
  [ -n "$a" ]  # 字符串不为空
  
  # 数值测试
  [ $a -eq $b ]  # 相等
  [ $a -ne $b ]  # 不等
  [ $a -lt $b ]  # 小于
  [ $a -le $b ]  # 小于等于
  [ $a -gt $b ]  # 大于
  [ $a -ge $b ]  # 大于等于
  ```

### 5. 循环结构
- **for循环**：
  ```bash
  # 遍历列表
  for var in item1 item2 item3; do
      echo $var
  done
  
  # 遍历文件
  for file in *.txt; do
      echo $file
  done
  
  # C风格循环
  for ((i=1; i<=10; i++)); do
      echo $i
  done
  ```

- **while循环**：
  ```bash
  i=1
  while [ $i -le 10 ]; do
      echo $i
      i=$((i+1))
  done
  ```

- **until循环**：
  ```bash
  i=1
  until [ $i -gt 10 ]; do
      echo $i
      i=$((i+1))
  done
  ```

### 6. 函数
- **定义函数**：
  ```bash
  function_name() {
      commands
      return value
  }
  ```

- **调用函数**：
  ```bash
  function_name
  function_name arg1 arg2
  ```

### 7. 脚本示例
- **系统监控脚本**：
  ```bash
  #!/bin/bash
  
  echo "=== System Status ==="
  echo "Date: $(date)"
  echo "Hostname: $(hostname)"
  echo ""
  echo "=== CPU Usage ==="
  top -bn1 | grep "Cpu(s)"
  echo ""
  echo "=== Memory Usage ==="
  free -h
  echo ""
  echo "=== Disk Usage ==="
  df -h
  echo ""
  echo "=== Network Connections ==="
  netstat -tuln | head -20
  ```

- **文件备份脚本**：
  ```bash
  #!/bin/bash
  
  SOURCE_DIR="/home/user/data"
  BACKUP_DIR="/home/user/backup"
  DATE=$(date +%Y%m%d_%H%M%S)
  
  mkdir -p $BACKUP_DIR
  tar -czf $BACKUP_DIR/backup_$DATE.tar.gz $SOURCE_DIR
  
  echo "Backup completed: $BACKUP_DIR/backup_$DATE.tar.gz"
  echo "Backup size: $(du -h $BACKUP_DIR/backup_$DATE.tar.gz)"
  ```

## 五、推荐学习资源

### 书籍
- 《Linux命令行与shell脚本编程大全》
- 《鸟哥的Linux私房菜》
- 《Linux系统管理技术手册》

### 在线资源
- [Linux Documentation Project](https://www.tldp.org/)
- [Linux Command](https://linuxcommand.org/)
- [Ubuntu Documentation](https://help.ubuntu.com/)
- [CentOS Documentation](https://docs.centos.org/)

### 视频教程
- 慕课网：Linux基础课程
- B站：Linux相关视频
- YouTube：Linux教程

## 六、实践建议

1. **搭建实验环境**：使用虚拟机软件安装Linux系统
2. **多做练习**：通过实际操作巩固命令
3. **解决问题**：尝试解决各种Linux问题
4. **参与社区**：加入Linux社区，向他人学习
5. **阅读源码**：查看一些简单的Linux工具源码

通过系统学习和实践，你将逐渐掌握Linux系统管理技能，为后续的DevOps学习打下坚实的基础。