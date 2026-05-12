# Terraform实战

## Terraform简介

### 什么是Terraform
- HashiCorp开发的基础设施即代码工具
- 使用声明式配置语言
- 支持多云平台
- 基础设施版本化、可重复、可审计

### 核心概念

| 概念 | 说明 |
|------|------|
| Provider | 云平台插件（AWS、阿里云等） |
| Resource | 基础设施资源（实例、网络等） |
| Module | 可复用的配置模块 |
| State | 基础设施状态文件 |
| Plan | 执行计划预览 |
| Apply | 应用配置变更 |

## 安装和配置

### 安装Terraform

```bash
# Linux/Mac
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Mac (Homebrew)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Windows (Chocolatey)
choco install terraform

# 验证安装
terraform version
```

### 配置阿里云Provider

```hcl
# provider.tf
terraform {
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = ">= 1.212.0"
    }
  }
}

provider "alicloud" {
  region = "cn-hangzhou"
}
```

### 配置环境变量

```bash
# 阿里云凭证
export ALICLOUD_ACCESS_KEY="your-access-key"
export ALICLOUD_SECRET_KEY="your-secret-key"
export ALICLOUD_REGION="cn-hangzhou"

# AWS凭证
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_REGION="us-east-1"
```

## HCL语法基础

### 变量定义

```hcl
# variables.tf
variable "region" {
  description = "The region to deploy resources"
  type        = string
  default     = "cn-hangzhou"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_types" {
  description = "ECS instance types"
  type        = list(string)
  default     = ["ecs.g6.large", "ecs.g6.xlarge"]
}

variable "tags" {
  description = "Tags for resources"
  type        = map(string)
  default = {
    Project = "DevOps"
    Env     = "Test"
  }
}
```

### 输出值

```hcl
# outputs.tf
output "vpc_id" {
  description = "The ID of the VPC"
  value       = alicloud_vpc.main.id
}

output "public_ips" {
  description = "Public IPs of ECS instances"
  value       = alicloud_instance.web[*].public_ip
}
```

### 数据源

```hcl
# 获取最新的CentOS镜像
data "alicloud_images" "centos" {
  name_regex  = "^centos_7"
  most_recent = true
  owners      = "system"
}

# 获取可用区列表
data "alicloud_zones" "available" {
  available_resource_creation = "Instance"
}
```

## 实战：创建VPC

```hcl
# vpc.tf

# 创建VPC
resource "alicloud_vpc" "main" {
  vpc_name   = "terraform-vpc"
  cidr_block = var.vpc_cidr
}

# 创建公网子网（可用区A）
resource "alicloud_vswitch" "public_a" {
  vswitch_name = "public-subnet-a"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = "10.0.1.0/24"
  zone_id      = data.alicloud_zones.available.zones[0].id
}

# 创建公网子网（可用区B）
resource "alicloud_vswitch" "public_b" {
  vswitch_name = "public-subnet-b"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = "10.0.2.0/24"
  zone_id      = data.alicloud_zones.available.zones[1].id
}

# 创建私网子网（可用区A）
resource "alicloud_vswitch" "private_a" {
  vswitch_name = "private-subnet-a"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = "10.0.10.0/24"
  zone_id      = data.alicloud_zones.available.zones[0].id
}

# 创建私网子网（可用区B）
resource "alicloud_vswitch" "private_b" {
  vswitch_name = "private-subnet-b"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = "10.0.20.0/24"
  zone_id      = data.alicloud_zones.available.zones[1].id
}
```

## 实战：创建安全组

```hcl
# security.tf

# 创建安全组
resource "alicloud_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = alicloud_vpc.main.id
}

# 允许SSH（建议限制IP范围）
resource "alicloud_security_group_rule" "ssh" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "22/22"
  priority          = 1
  security_group_id = alicloud_security_group.web.id
  cidr_ip           = "0.0.0.0/0"
}

# 允许HTTP
resource "alicloud_security_group_rule" "http" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "80/80"
  priority          = 1
  security_group_id = alicloud_security_group.web.id
  cidr_ip           = "0.0.0.0/0"
}

# 允许HTTPS
resource "alicloud_security_group_rule" "https" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "443/443"
  priority          = 1
  security_group_id = alicloud_security_group.web.id
  cidr_ip           = "0.0.0.0/0"
}

# 允许所有出站流量
resource "alicloud_security_group_rule" "all_egress" {
  type              = "egress"
  ip_protocol       = "all"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "-1/-1"
  priority          = 1
  security_group_id = alicloud_security_group.web.id
  cidr_ip           = "0.0.0.0/0"
}
```

## 实战：创建ECS实例

```hcl
# ecs.tf

# 创建SSH密钥对
resource "alicloud_key_pair" "main" {
  key_name   = "terraform-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

# 创建ECS实例（多实例）
resource "alicloud_instance" "web" {
  count = 2

  instance_name = "web-${count.index}"
  instance_type = "ecs.g6.large"

  # 使用数据源获取的镜像
  image_id      = data.alicloud_images.centos.images[0].id

  security_groups = [alicloud_security_group.web.id]
  vswitch_id      = count.index == 0 ? alicloud_vswitch.public_a.id : alicloud_vswitch.public_b.id

  key_name = alicloud_key_pair.main.key_name

  # 系统盘
  system_disk_size = 40
  system_disk_category = "cloud_efficiency"

  # 分配公网IP
  internet_max_bandwidth_out = 10

  # 标签
  tags = {
    Name    = "web-${count.index}"
    Project = var.tags["Project"]
    Env     = var.tags["Env"]
  }

  # 用户数据（初始化脚本）
  user_data = <<-EOF
              #!/bin/bash
              yum install -y nginx
              systemctl start nginx
              systemctl enable nginx
              echo "<h1>Hello from Web Server ${count.index}</h1>" > /usr/share/nginx/html/index.html
              EOF
}
```

## 实战：创建负载均衡

```hcl
# slb.tf

# 创建负载均衡
resource "alicloud_slb_load_balancer" "web" {
  load_balancer_name = "web-slb"
  load_balancer_spec = "slb.s1.small"
  address_type       = "internet"
  vswitch_id         = alicloud_vswitch.public_a.id

  tags = {
    Project = var.tags["Project"]
  }
}

# 添加后端服务器
resource "alicloud_slb_attachment" "web" {
  load_balancer_id = alicloud_slb_load_balancer.web.id
  instance_ids     = alicloud_instance.web[*].id
}

# 配置HTTP监听
resource "alicloud_slb_listener" "http" {
  load_balancer_id          = alicloud_slb_load_balancer.web.id
  frontend_port             = 80
  protocol                  = "http"
  backend_port              = 80
  bandwidth                 = -1
  scheduler                 = "wrr"
  health_check_connect_port = "80"
  healthy_threshold         = 2
  unhealthy_threshold       = 2
  health_check_timeout      = 3
  health_check_interval     = 2
  health_check_type         = "tcp"
}
```

## Terraform工作流

### 初始化项目

```bash
# 初始化（下载Provider插件）
terraform init

# 查看初始化结果
ls -la .terraform/
```

### 预览变更

```bash
# 生成执行计划
terraform plan

# 保存执行计划
terraform plan -out=tfplan
```

### 应用变更

```bash
# 应用配置
terraform apply

# 自动确认
terraform apply -auto-approve

# 应用保存的计划
terraform apply tfplan
```

### 销毁资源

```bash
# 销毁所有资源
terraform destroy

# 销毁特定资源
terraform destroy -target=alicloud_instance.web[0]
```

## 状态管理

### 本地状态（默认）

```
terraform.tfstate       # 状态文件
terraform.tfstate.backup # 备份文件
```

### 远程状态（推荐）

```hcl
# 使用OSS作为后端存储
terraform {
  backend "oss" {
    bucket = "terraform-state-bucket"
    prefix = "state/"
    key    = "terraform.tfstate"
    region = "cn-hangzhou"
  }
}

# 使用S3作为后端存储
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```

### 状态操作

```bash
# 查看状态
terraform show

# 列出资源
terraform state list

# 查看特定资源状态
terraform state show alicloud_instance.web[0]

# 移除资源（从状态中移除但不删除实际资源）
terraform state rm alicloud_instance.web[0]

# 刷新状态
terraform refresh
```

## 模块化

### 创建Module

```
modules/
└── web_server/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

```hcl
# modules/web_server/variables.tf
variable "instance_count" {
  type    = number
  default = 2
}

variable "instance_type" {
  type    = string
  default = "ecs.g6.large"
}

variable "vpc_id" {
  type = string
}

# modules/web_server/main.tf
resource "alicloud_instance" "web" {
  count         = var.instance_count
  instance_type = var.instance_type
  vpc_id        = var.vpc_id
  # ... 其他配置
}
```

### 使用Module

```hcl
# main.tf
module "web_server" {
  source = "./modules/web_server"

  instance_count = 3
  instance_type  = "ecs.g6.xlarge"
  vpc_id         = alicloud_vpc.main.id
}
```

## Workspace多环境

```bash
# 列出workspace
terraform workspace list

# 创建新workspace
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# 切换workspace
terraform workspace select dev

# 在配置中使用workspace
locals {
  env = terraform.workspace
  instance_count = local.env == "prod" ? 3 : 1
}
```

## 完整项目结构

```
terraform-project/
├── main.tf              # 主配置文件
├── variables.tf         # 变量定义
├── outputs.tf           # 输出值
├── provider.tf          # Provider配置
├── terraform.tfvars     # 变量值（不提交到Git）
├── terraform.tfvars.example  # 示例配置
├── .gitignore           # Git忽略规则
└── modules/             # 自定义模块
    ├── vpc/
    ├── security/
    └── compute/
```

## .gitignore示例

```
# Terraform
.terraform/
*.tfstate
*.tfstate.backup
*.tfplan
crash.log
terraform.tfvars
.terraform.lock.hcl
```

## 最佳实践

1. **版本控制**：所有配置都提交到Git
2. **代码审查**：使用Pull Request审查变更
3. **自动化**：CI/CD运行terraform plan
4. **远程状态**：使用远程后端存储状态
5. **状态锁定**：启用状态锁防止并发修改
6. **模块化**：常用配置封装为Module
7. **变量化**：不要硬编码，使用变量
8. **文档化**：为变量和输出添加描述

## 常见问题

### 1. 执行计划与预期不符

```bash
# 刷新状态
terraform refresh

# 重新初始化
terraform init -reconfigure
```

### 2. 资源创建失败

```bash
# 查看详细日志
TF_LOG=DEBUG terraform apply

# 检查资源依赖关系
```

### 3. 状态文件损坏

```bash
# 从备份恢复
cp terraform.tfstate.backup terraform.tfstate

# 重新导入资源
terraform import alicloud_instance.web i-xxx123456
```

## 学习检查清单

- [ ] 理解Terraform核心概念
- [ ] 能够编写基本的Terraform配置
- [ ] 理解HCL变量、输出、数据源
- [ ] 能够创建VPC和子网
- [ ] 能够创建安全组规则
- [ ] 能够创建ECS实例
- [ ] 理解Terraform工作流（init/plan/apply）
- [ ] 了解状态管理的重要性
- [ ] 能够创建简单的Module
- [ ] 理解Workspace的用途
