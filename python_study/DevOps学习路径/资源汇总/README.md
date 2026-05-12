# DevOps运维学习路径

## 学习路线图

本学习路径专为想系统学习DevOps的同学设计，涵盖从基础到高级的完整知识体系。

## 路径总览

```
阶段一：基础准备（4周）
    ├── 计算机基础
    ├── Linux系统管理
    └── 实操指南

阶段二：核心DevOps技能（8周）
    ├── Git版本控制
    ├── CI/CD实践
    ├── Docker容器
    └── Kubernetes编排

阶段三：云平台与基础设施（8周）
    ├── 云平台基础
    ├── Terraform实战
    └── Ansible实战

阶段四：监控与可观测性（8周）
    ├── Prometheus监控
    ├── Grafana可视化
    └── ELK日志系统

阶段五：安全与合规（8周）
    └── DevSecOps实践

阶段六：高级技能（8周）
    └── 混沌工程
    └── 性能优化（待补充）

阶段七：实战项目（8周）
    ├── 项目1：Web应用完整部署
    └── 项目2-3（待补充）

总时长：约52周（约1年）
```

## 阶段详解

### [阶段一：基础准备](01_阶段一_基础准备/)

**目标**：建立扎实的计算机和Linux基础

**核心内容**：
- 计算机原理、网络基础、数据库基础
- Linux命令行、系统管理、Shell脚本
- 虚拟机搭建、实验环境配置

**时间**：4周

---

### [阶段二：核心DevOps技能](02_阶段二_核心DevOps技能/)

**目标**：掌握DevOps核心工具链

**核心内容**：
- Git版本控制和工作流
- CI/CD实践（Jenkins、GitHub Actions）
- Docker容器化
- Kubernetes容器编排

**时间**：8周

---

### [阶段三：云平台与基础设施](03_阶段三_云平台与基础设施/)

**目标**：掌握云计算和基础设施即代码

**核心内容**：
- 云平台基础（AWS/阿里云/腾讯云）
- Terraform基础设施即代码
- Ansible配置管理
- 网络架构与负载均衡

**时间**：8周

---

### [阶段四：监控与可观测性](04_阶段四_监控与可观测性/)

**目标**：建立完整的监控体系

**核心内容**：
- Prometheus监控系统
- Grafana可视化
- ELK日志系统
- APM与链路追踪

**时间**：8周

---

### [阶段五：安全与合规](05_阶段五_安全与合规/)

**目标**：将安全集成到DevOps流程中

**核心内容**：
- DevSecOps理念
- 代码安全扫描
- 容器安全
- 合规与审计

**时间**：8周

---

### [阶段六：高级技能](06_阶段六_高级技能/)

**目标**：掌握高级DevOps技能

**核心内容**：
- 混沌工程
- 性能测试与优化
- 高可用架构
- 容灾备份

**时间**：8周

---

### [阶段七：实战项目](07_阶段七_实战项目/)

**目标**：通过项目巩固所学

**核心内容**：
- Web应用完整部署（Git + CI/CD + K8s + 监控）
- 微服务架构实践
- 云原生应用开发

**时间**：8周

---

## 学习方法

### 时间规划

| 阶段 | 时长 | 每天学习 | 总时长 |
|--------|------|----------|---------|
| 阶段一 | 4周 | 2-3小时 | 56-84小时 |
| 阶段二 | 8周 | 2-3小时 | 112-168小时 |
| 阶段三 | 8周 | 2-3小时 | 112-168小时 |
| 阶段四 | 8周 | 2-3小时 | 112-168小时 |
| 阶段五 | 8周 | 2-3小时 | 112-168小时 |
| 阶段六 | 8周 | 2-3小时 | 112-168小时 |
| 阶段七 | 8周 | 2-3小时 | 112-168小时 |

### 学习建议

1. **动手实践**：每个知识点都要实际操作
2. **做笔记**：记录重要概念和命令
3. **循序渐进**：不要跳过基础直接上复杂内容
4. **定期回顾**：每周回顾本周内容
5. **项目驱动**：用项目驱动学习
6. **加入社区**：参与技术社区，学习他人经验

### 实操环境

| 工具 | 免费方案 |
|--------|------------|
| 云平台 | AWS Free Tier / 阿里云免费套餐 |
| K8s集群 | minikube / kind |
| 虚拟机 | VirtualBox / VMware |
| CI/CD | GitLab / GitHub |

## 技能树

```
DevOps Engineer
├── 操作系统
│   ├── Linux (精通)
│   └── Shell脚本
│   └── 系统管理
├── 版本控制
│   └── Git
│   └── GitLab/GitHub
│   └── Git工作流
├── CI/CD
│   ├── Jenkins
│   ├── GitHub Actions
│   └── GitLab CI
├── 容器
│   ├── Docker
│   └── Docker Compose
│   └── 镜像仓库
├── 编排
│   ├── Kubernetes
│   ├── Helm
│   └── Operators
├── 云平台
│   ├── AWS/阿里云/腾讯云
│   ├── VPC网络
│   └── 云服务
├── IaC
│   ├── Terraform
│   └── Ansible
├── 监控
│   ├── Prometheus
│   ├── Grafana
│   └── ELK
├── 安全
│   ├── 代码扫描
│   ├── 容器安全
│   └── 合规审计
└── 高级
    ├── 混沌工程
    ├── 性能优化
    └── 架构设计
```

## 推荐资源

### 官方文档
- [Kubernetes官方文档](https://kubernetes.io/zh-cn/docs/)
- [Docker官方文档](https://docs.docker.com/)
- [Terraform官方文档](https://developer.hashicorp.com/terraform/docs)
- [Prometheus官方文档](https://prometheus.io/docs/)

### 免费课程
- [Kubernetes官方教程](https://kubernetes.io/zh-cn/docs/tutorials/)
- [CNCF免费课程](https://www.cncf.io/certification/training/)
- [AWS Skill Builder](https://explore.skillbuilder.aws/)

### 书籍推荐
1. 《DevOps实践指南》
2. 《持续交付》
3. 《Kubernetes权威指南》
4. 《Terraform: Up and Running》

### YouTube频道
- CNCF
- HashiCorp
- TechWorld with Nana
- Anton Putra

## 学习检查点

每完成一个阶段，应该能够：

- **阶段一**：在Linux上部署一个Web服务
- **阶段二**：将应用容器化并部署到K8s
- **阶段三**：用Terraform创建云基础设施
- **阶段四**：部署完整的监控告警系统
- **阶段五**：在CI/CD中集成安全扫描
- **阶段六**：设计高可用和容灾方案
- **阶段七**：完成至少2个实战项目

## 常见问题

### Q: 没有编程基础可以学DevOps吗？
A: 可以，但建议先学习基础的Python/Shell编程。

### Q: 需要买服务器吗？
A: 不需要，使用免费套餐和本地虚拟机足够。

### Q: 如何保持学习动力？
A: 设定小目标，加入学习群，定期做项目。

### Q: 学完能找到工作吗？
A: 完成所有阶段并完成项目，可以应聘初级DevOps工程师。

## 贡献

欢迎提交Issue和PR完善本学习路径！

## 许可证

MIT License

---

**开始你的DevOps之旅吧！**

> "The best time to plant a tree was 20 years ago. The second best time is now."
