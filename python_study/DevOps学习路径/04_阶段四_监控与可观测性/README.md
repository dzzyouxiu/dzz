# 阶段四：监控与可观测性

## 学习目标
- 理解监控与可观测性概念
- 掌握Prometheus监控系统
- 熟练使用Grafana可视化
- 掌握ELK Stack日志管理
- 了解APM应用性能监控

## 主要内容
1. **Prometheus监控**：指标采集、告警规则、PromQL
2. **Grafana可视化**：Dashboard创建、图表配置、告警
3. **ELK日志系统**：Elasticsearch、Logstash、Kibana
4. **APM应用监控**：链路追踪、性能分析

## 学习路径

### 第1-2周：Prometheus监控系统

#### 知识点
- 监控基础概念（指标、日志、链路）
- Prometheus架构和原理
- 数据模型和时间序列
- PromQL查询语言
- Exporter概念和使用
- Alertmanager告警

#### 实操环境搭建
1. 安装Docker和Docker Compose
2. 使用Docker Compose启动Prometheus
3. 安装Node Exporter监控主机
4. 配置Prometheus采集规则

#### 实操练习
1. 监控Linux服务器基础指标
2. 监控Nginx服务状态
3. 监控MySQL数据库
4. 监控Docker容器
5. 编写PromQL查询语句
6. 配置告警规则

#### 免费资源
- 官方文档：https://prometheus.io/docs
- 官方教程：https://prometheus.io/tutorials
- Awesome Prometheus：https://github.com/roaldnefs/awesome-prometheus
- Exporter目录：https://prometheus.io/docs/instrumenting/exporters

### 第3-4周：Grafana可视化

#### 知识点
- Grafana架构和安装
- 数据源配置
- Dashboard创建
- 图表类型（Graph、Table、Gauge等）
- 变量和模板
- 告警配置
- 权限管理

#### 实操环境搭建
1. Docker安装Grafana
2. 配置Prometheus数据源
3. 导入官方Dashboard模板

#### 实操练习
1. 创建系统监控Dashboard
2. 创建应用监控Dashboard
3. 配置告警通知（邮件/钉钉）
4. 导入社区Dashboard
5. 创建自定义图表

#### 免费资源
- 官方文档：https://grafana.com/docs
- Grafana Labs：https://grafana.com/grafana
- Dashboard市场：https://grafana.com/grafana/dashboards
- YouTube：Grafana官方教程

### 第5-6周：ELK日志系统

#### 知识点
- 日志管理基础
- Elasticsearch原理和使用
- Logstash数据收集和处理
- Kibana可视化和查询
- Beats轻量采集器
- 日志解析和过滤

#### 实操环境搭建
1. Docker Compose部署ELK Stack
2. 安装Filebeat采集日志
3. 配置Logstash过滤规则

#### 实操练习
1. 收集Nginx日志
2. 收集应用日志
3. 解析JSON格式日志
4. 创建Kibana Dashboard
5. 配置日志告警

#### 免费资源
- 官方文档：https://www.elastic.co/guide
- Elastic社区：https://discuss.elastic.co
- 免费课程：https://www.elastic.co/training/free
- YouTube：Elastic官方频道

### 第7-8周：APM和链路追踪

#### 知识点
- 可观测性三大支柱
- 分布式追踪概念
- OpenTelemetry标准
- Jaeger/Zipkin使用
- APM工具对比

#### 实操练习
1. 集成OpenTelemetry到应用
2. 部署Jaeger收集追踪数据
3. 在Kibana查看追踪
4. 分析性能瓶颈

#### 免费资源
- OpenTelemetry：https://opentelemetry.io
- Jaeger：https://www.jaegertracing.io
- Zipkin：https://zipkin.io

## 学习方法
1. **边学边练**：搭建完整监控环境
2. **实际场景**：模拟真实系统监控
3. **故障演练**：制造问题并排查
4. **社区学习**：参考优秀Dashboard和规则

## 时间规划
- 总时长：8周
- 每天学习时间：2-3小时
- 周末完成综合实验

## 预期成果
- 能够部署完整的监控系统
- 能够编写PromQL查询
- 能够创建Grafana Dashboard
- 能够配置ELK收集分析日志
- 能够配置告警通知

## 后续阶段
完成本阶段后，将进入**阶段五：安全与合规**，学习DevOps安全和合规相关知识。
