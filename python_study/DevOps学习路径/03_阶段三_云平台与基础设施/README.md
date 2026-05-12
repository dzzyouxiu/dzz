# 阶段三：云平台与基础设施

## 学习目标
- 掌握主流云平台（AWS/阿里云/腾讯云）基础服务
- 理解基础设施即代码（IaC）概念
- 熟练使用Terraform和Ansible进行自动化管理
- 掌握网络架构和负载均衡配置

## 主要内容
1. **云平台基础**：AWS/阿里云/腾讯云核心服务
2. **基础设施即代码**：Terraform配置管理
3. **配置管理**：Ansible自动化配置
4. **网络架构**：VPC、子网、路由、安全组
5. **负载均衡**：ALB/NLB/CLB配置

## 学习路径

### 第1-2周：云平台基础

#### 知识点
- 云计算概念和服务模型（IaaS/PaaS/SaaS）
- 计算服务：EC2/ECS/CVM
- 存储服务：S3/OSS/COS、EBS/云盘
- 网络服务：VPC、子网、弹性IP
- 数据库服务：RDS/云数据库

#### 实操环境搭建
1. 注册云平台免费套餐
   - AWS Free Tier：https://aws.amazon.com/free
   - 阿里云免费套餐：https://www.aliyun.com/free
   - 腾讯云免费套餐：https://cloud.tencent.com/free
2. 创建账户并配置访问密钥
3. 安装云平台CLI工具
   - AWS CLI
   - 阿里云CLI
   - 腾讯云CLI

#### 实操练习
1. 创建一台云服务器（EC2/ECS/CVM）
2. 配置安全组开放端口
3. 创建云存储桶并上传文件
4. 创建VPC和子网

#### 免费资源
- **AWS**：
  - 官方文档：https://docs.aws.amazon.com
  - AWS Skill Builder：https://explore.skillbuilder.aws（免费课程）
  - YouTube：AWS Online Tech Talks
- **阿里云**：
  - 官方文档：https://help.aliyun.com
  - 阿里云大学：https://edu.aliyun.com（免费课程）
- **腾讯云**：
  - 官方文档：https://cloud.tencent.com/document
  - 腾讯云大学：https://cloud.tencent.com/edu

### 第3-4周：Terraform基础设施即代码

#### 知识点
- IaC概念和优势
- Terraform工作原理
- HCL配置语言
- Provider、Resource、Module概念
- Terraform状态管理
- Terraform工作流（init/plan/apply/destroy）

#### 实操环境搭建
1. 安装Terraform
   - 下载：https://developer.hashicorp.com/terraform/downloads
   - 配置环境变量
2. 配置云平台凭证
3. 安装VS Code Terraform插件

#### 实操练习
1. 使用Terraform创建VPC
2. 使用Terraform创建ECS实例
3. 使用Terraform创建安全组
4. 创建模块化的Terraform配置
5. 使用Terraform Workspace管理多环境

#### 示例代码
```hcl
# main.tf
terraform {
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = ">= 1.0.0"
    }
  }
}

provider "alicloud" {
  region = "cn-hangzhou"
}

# 创建VPC
resource "alicloud_vpc" "main" {
  vpc_name   = "my-vpc"
  cidr_block = "10.0.0.0/16"
}

# 创建子网
resource "alicloud_vswitch" "main" {
  vswitch_name = "my-vswitch"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = "10.0.1.0/24"
  zone_id      = "cn-hangzhou-b"
}
```

#### 免费资源
- 官方文档：https://developer.hashicorp.com/terraform/docs
- Terraform入门：https://developer.hashicorp.com/terraform/tutorials
- YouTube：HashiCorp官方频道
- 书籍：《Terraform: Up and Running》

### 第5-6周：Ansible配置管理

#### 知识点
- Ansible架构和工作原理
- Inventory管理
- Playbook编写
- Module使用
- Role和Galaxy
- 变量管理和模板

#### 实操环境搭建
1. 安装Ansible（控制节点使用Linux/Mac/WSL）
   ```bash
   pip install ansible
   ```
2. 配置SSH免密登录
3. 准备被管理节点（可以使用虚拟机）

#### 实操练习
1. 编写Inventory文件
2. 使用Ad-hoc命令执行操作
3. 编写Playbook部署Nginx
4. 创建Role部署应用
5. 使用变量和模板配置服务

#### 示例代码
```yaml
# inventory.ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20

# nginx.yml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy config file
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

#### 免费资源
- 官方文档：https://docs.ansible.com
- Ansible Galaxy：https://galaxy.ansible.com
- YouTube：Ansible官方教程
- 书籍：《Ansible: Up and Running》

### 第7-8周：网络架构与负载均衡

#### 知识点
- VPC网络设计
- 子网划分和路由表
- 安全组和网络ACL
- NAT网关配置
- 负载均衡原理和类型
- 七层和四层负载均衡

#### 实操练习
1. 设计多可用区VPC架构
2. 配置公网和私网子网
3. 配置NAT网关
4. 创建负载均衡器
5. 配置SSL证书
6. 实现自动扩缩容

#### 免费资源
- AWS VPC文档：https://docs.aws.amazon.com/vpc
- 阿里云VPC文档：https://help.aliyun.com/vpc
- 负载均衡最佳实践

## 学习方法
1. **动手实践**：云平台提供免费套餐，多动手操作
2. **架构设计**：学习参考架构，设计自己的VPC
3. **成本控制**：注意删除不使用的资源
4. **最佳实践**：学习云平台Well-Architected Framework

## 时间规划
- 总时长：8周
- 每天学习时间：2-3小时
- 周末可以安排较长时间的实验操作

## 预期成果
- 能够在主流云平台上创建基础架构
- 能够使用Terraform编写基础设施代码
- 能够使用Ansible进行自动化配置
- 能够设计基础的网络架构和负载均衡

## 成本提示
- 使用免费套餐进行学习
- 实验完成后及时删除资源
- 设置账单告警避免意外费用

## 后续阶段
完成本阶段学习后，将进入**阶段四：监控与可观测性**，学习Prometheus、Grafana、ELK等监控工具。
