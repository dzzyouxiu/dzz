# Ansible实战

## Ansible简介

### 什么是Ansible
- 开源的配置管理和自动化工具
- 基于Python开发
- 无Agent架构（使用SSH）
- 声明式语言（YAML）
- 幂等性保证

### 核心概念

| 概念 | 说明 |
|------|------|
| Control Node | 控制节点（安装Ansible） |
| Managed Node | 被管理节点 |
| Inventory | 主机清单文件 |
| Module | 执行模块 |
| Playbook | 自动化剧本 |
| Role | 可复用的角色 |
| Task | 单个任务 |
| Handler | 触发任务 |

## 安装Ansible

### 控制节点安装

```bash
# 使用pip安装（推荐）
pip install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# CentOS/RHEL
sudo yum install epel-release
sudo yum install ansible

# Mac (Homebrew)
brew install ansible

# 验证安装
ansible --version
```

### 被管理节点要求

```bash
# Linux要求
- Python 2.7 或 Python 3.5+
- SSH服务开启
- 有sudo权限的用户

# 安装Python（被管理节点）
# Ubuntu/Debian
sudo apt install python3

# CentOS/RHEL
sudo yum install python3
```

## 配置SSH免密登录

```bash
# 生成密钥（如果没有）
ssh-keygen -t rsa -b 2048

# 复制公钥到被管理节点
ssh-copy-id user@192.168.1.10

# 测试免密登录
ssh user@192.168.1.10 hostname
```

## Inventory主机清单

### 基础INI格式

```ini
# inventory.ini

[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu

[dbservers]
db1 ansible_host=192.168.1.20 ansible_user=centos ansible_python_interpreter=/usr/bin/python3

[all:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### YAML格式（推荐）

```yaml
# inventory.yml
all:
  vars:
    ansible_ssh_private_key_file: ~/.ssh/id_rsa

  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu
        web2:
          ansible_host: 192.168.1.11
          ansible_user: ubuntu

    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
          ansible_user: centos
```

### 动态Inventory

```bash
# 使用动态Inventory脚本（云平台）
ansible-inventory -i aws_ec2.yml --list

# 阿里云动态Inventory
# 下载：https://github.com/alibaba/ansible-provider
```

## Ad-hoc命令

### 基本语法

```bash
ansible <pattern> -m <module> -a "<arguments>"
```

### 常用命令

```bash
# 测试连通性
ansible all -m ping

# 执行Shell命令
ansible webservers -m shell -a "uptime"

# 查看磁盘空间
ansible all -m command -a "df -h"

# 复制文件
ansible webservers -m copy -a "src=./app.conf dest=/etc/app.conf mode=0644"

# 安装软件包
ansible webservers -m apt -a "name=nginx state=present" -b

# 启动服务
ansible webservers -m service -a "name=nginx state=started enabled=yes"
```

### 常用模块速查

| 模块 | 用途 | 示例 |
|------|------|------|
| ping | 测试连通性 | ansible all -m ping |
| command | 执行命令 | -a "date" |
| shell | 执行Shell | -a "ls -l \| grep txt" |
| copy | 复制文件 | -a "src=a dest=b" |
| file | 文件操作 | -a "path=/tmp mode=0755" |
| apt/yum | 包管理 | -a "name=nginx state=present" |
| service | 服务管理 | -a "name=nginx state=started" |
| user | 用户管理 | -a "name=john state=present" |
| git | Git操作 | -a "repo=xxx dest=/app" |

## Playbook基础

### 第一个Playbook

```yaml
# nginx_install.yml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes
  gather_facts: yes

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
```

### 执行Playbook

```bash
# 执行
ansible-playbook -i inventory.ini nginx_install.yml

# 检查模式（不实际执行）
ansible-playbook -C nginx_install.yml

# 详细输出
ansible-playbook -v nginx_install.yml

# 限制执行主机
ansible-playbook -l web1 nginx_install.yml
```

## Playbook进阶

### 变量使用

```yaml
---
- name: Deploy application
  hosts: webservers
  become: yes
  vars:
    app_name: myapp
    app_port: 8080
    packages:
      - nginx
      - git
      - python3

  tasks:
    - name: Install required packages
      apt:
        name: "{{ packages }}"
        state: present

    - name: Create app directory
      file:
        path: "/opt/{{ app_name }}"
        state: directory
        mode: '0755'
```

### 条件判断

```yaml
---
- name: Configure based on OS
  hosts: all
  become: yes

  tasks:
    - name: Install on Ubuntu
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install on CentOS
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"
```

### 循环

```yaml
---
- name: Create multiple users
  hosts: all
  become: yes

  tasks:
    - name: Create users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop:
        - { name: 'alice', groups: 'sudo,dev' }
        - { name: 'bob', groups: 'dev' }
        - { name: 'charlie', groups: 'sudo' }
```

### 模板（Jinja2）

```yaml
# Playbook
---
- name: Configure Nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Copy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        mode: '0644'
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

```jinja2
{# templates/nginx.conf.j2 #}
user {{ nginx_user }};
worker_processes {{ ansible_processor_vcpus }};

http {
    server {
        listen {{ app_port }};
        server_name {{ server_name }};

        location / {
            proxy_pass http://localhost:{{ backend_port }};
        }
    }
}
```

### Handler触发

```yaml
---
- name: Deploy app
  hosts: webservers
  become: yes

  tasks:
    - name: Copy app config
      copy:
        src: app.conf
        dest: /etc/app.conf
      notify:
        - Restart app
        - Check app status

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

  handlers:
    - name: Restart app
      service:
        name: myapp
        state: restarted

    - name: Check app status
      command: systemctl status myapp

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

## Role角色

### Role目录结构

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    │   └── index.html
    ├── vars/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    └── README.md
```

### 创建Nginx Role

```yaml
# roles/nginx/tasks/main.yml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Copy config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

- name: Start service
  service:
    name: nginx
    state: started
    enabled: yes

# roles/nginx/handlers/main.yml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted

# roles/nginx/defaults/main.yml
---
nginx_port: 80
nginx_worker_processes: auto
```

### 使用Role

```yaml
# site.yml
---
- name: Deploy web servers
  hosts: webservers
  become: yes

  roles:
    - role: nginx
      nginx_port: 8080  # 覆盖默认变量

    - role: common
      tags: common
```

### 从Ansible Galaxy安装Role

```bash
# 搜索Role
ansible-galaxy search nginx

# 安装Role
ansible-galaxy install geerlingguy.nginx

# 列出已安装的Role
ansible-galaxy list

# 使用requirements.yml
ansible-galaxy install -r requirements.yml
```

```yaml
# requirements.yml
---
roles:
  - name: geerlingguy.nginx
    version: 3.0.0

  - name: geerlingguy.docker
    src: geerlingguy.docker
```

## 实战：部署Web应用

```yaml
# deploy_webapp.yml
---
- name: Deploy Web Application
  hosts: webservers
  become: yes
  vars:
    app_repo: "https://github.com/user/myapp.git"
    app_dir: "/opt/myapp"
    app_user: "www-data"

  tasks:
    - name: Install required packages
      apt:
        name:
          - git
          - python3
          - python3-pip
          - nginx
        state: present
        update_cache: yes

    - name: Clone application code
      git:
        repo: "{{ app_repo }}"
        dest: "{{ app_dir }}"
        version: main
        force: yes
      notify: Restart app

    - name: Install Python dependencies
      pip:
        requirements: "{{ app_dir }}/requirements.txt"
        virtualenv: "{{ app_dir }}/venv"
        virtualenv_python: python3

    - name: Set ownership
      file:
        path: "{{ app_dir }}"
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        recurse: yes

    - name: Copy systemd service file
      template:
        src: templates/myapp.service.j2
        dest: /etc/systemd/system/myapp.service
        mode: '0644'
      notify:
        - Reload systemd
        - Restart app

    - name: Copy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/myapp
        mode: '0644'
      notify: Restart Nginx

    - name: Enable Nginx site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: Restart Nginx

  handlers:
    - name: Reload systemd
      systemd:
        daemon_reload: yes

    - name: Restart app
      service:
        name: myapp
        state: restarted
        enabled: yes

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

## 实战：配置数据库服务器

```yaml
# deploy_db.yml
---
- name: Deploy MySQL Server
  hosts: dbservers
  become: yes
  vars:
    mysql_root_password: "secure_password_here"
    mysql_db: "myappdb"
    mysql_user: "myappuser"
    mysql_password: "db_password_here"

  tasks:
    - name: Install MySQL
      apt:
        name:
          - mysql-server
          - python3-pymysql
        state: present

    - name: Start MySQL
      service:
        name: mysql
        state: started
        enabled: yes

    - name: Set root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
        host_all: yes

    - name: Create database
      mysql_db:
        name: "{{ mysql_db }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Create database user
      mysql_user:
        name: "{{ mysql_user }}"
        password: "{{ mysql_password }}"
        priv: "{{ mysql_db }}.*:ALL"
        host: "192.168.1.%"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
```

## 项目结构

```
ansible-project/
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   └── staging/
│       ├── hosts
│       └── group_vars/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── dbservers.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── mysql/
├── templates/
├── files/
├── ansible.cfg
└── README.md
```

## ansible.cfg配置

```ini
[defaults]
inventory = ./inventory/production/hosts
remote_user = ubuntu
become = True
become_method = sudo
host_key_checking = False
retry_files_enabled = False
callback_whitelist = profile_tasks
forks = 10

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

## 最佳实践

1. **版本控制**：所有Playbook提交到Git
2. **变量分离**：敏感变量使用Ansible Vault加密
3. **使用Role**：复用代码，保持整洁
4. **幂等性**：所有操作都应该幂等
5. **测试**：使用--check模式测试
6. **标签**：使用标签选择性执行任务
7. **文档**：为Playbook和Role添加README
8. **小步快跑**：小的Playbook比大的好

## Ansible Vault

```bash
# 加密文件
ansible-vault encrypt secrets.yml

# 解密文件
ansible-vault decrypt secrets.yml

# 查看加密文件
ansible-vault view secrets.yml

# 编辑加密文件
ansible-vault edit secrets.yml

# 使用加密文件执行
ansible-playbook -i hosts site.yml --ask-vault-pass
```

## 学习检查清单

- [ ] 理解Ansible架构和概念
- [ ] 能够编写Inventory文件
- [ ] 能够使用Ad-hoc命令
- [ ] 能够编写基本Playbook
- [ ] 理解变量、条件、循环
- [ ] 能够使用Jinja2模板
- [ ] 理解Handler的使用
- [ ] 能够创建简单Role
- [ ] 能够使用Ansible Galaxy
- [ ] 了解Ansible Vault
