# Grafana可视化

## Grafana简介

### 什么是Grafana
- 开源的数据可视化和监控分析平台
- 支持多种数据源
- 丰富的图表类型
- 灵活的告警系统
- 活跃的社区生态

### 主要特性
- 多数据源支持（Prometheus、MySQL、Elasticsearch等）
- 丰富的可视化面板
- 灵活的告警机制
- 用户权限管理
- Dashboard模板市场

## 安装Grafana

### Docker安装（推荐）

```bash
# 运行容器
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin123" \
  grafana/grafana:10.1.0

# Docker Compose
version: '3.8'
services:
  grafana:
    image: grafana/grafana:10.1.0
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped

volumes:
  grafana_data:
```

### Linux安装

```bash
# Ubuntu/Debian
sudo apt-get install -y apt-transport-https software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana

# 启动服务
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

## 数据源配置

### 添加Prometheus数据源

1. 访问 http://localhost:3000
2. 登录（默认admin/admin）
3. Configuration → Data Sources → Add data source
4. 选择Prometheus
5. 填写URL：http://prometheus:9090
6. 点击Save & Test

### 配置文件方式

```yaml
# provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true

  - name: MySQL
    type: mysql
    url: mysql:3306
    database: monitoring
    user: grafana
    secureJsonData:
      password: "password"

  - name: Elasticsearch
    type: elasticsearch
    url: http://elasticsearch:9200
    database: "logstash-*"
    jsonData:
      timeField: "@timestamp"
      timeInterval: "1m"
```

## Dashboard基础

### 创建第一个Dashboard

1. 点击左侧菜单 + → Dashboard
2. 添加Panel
3. 选择数据源和指标
4. 配置图表选项
5. 保存Dashboard

### 常用图表类型

| 图表类型 | 用途 | 示例 |
|----------|------|------|
| Time Series | 时间序列数据 | CPU使用率趋势 |
| Stat | 单值显示 | 当前内存使用 |
| Gauge | 仪表盘显示 | 磁盘使用率 |
| Table | 表格展示 | 主机列表 |
| Bar Chart | 柱状图 | 请求量对比 |
| Pie Chart | 饼图 | 请求分布 |
| Heatmap | 热力图 | 响应时间分布 |
| Graph (old) | 旧版图表 | 兼容旧版 |

## 常用图表配置

### CPU使用率图表

```
数据源：Prometheus
查询：100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
图例：{{instance}}
单位：Percent (0-100)
Y轴最小值：0
Y轴最大值：100
```

### 内存使用量图表

```
数据源：Prometheus
查询：node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
图例：{{instance}}
单位：bytes(SI) or IEC
显示格式：Gigabytes
```

### 磁盘使用率

```
数据源：Prometheus
查询：100 - ((node_filesystem_avail_bytes{mountpoint="/",fstype!="rootfs"} / node_filesystem_size_bytes{mountpoint="/",fstype!="rootfs"}) * 100)
图例：{{instance}} - {{mountpoint}}
单位：Percent
```

### 网络流量

```
数据源：Prometheus
查询入站：rate(node_network_receive_bytes_total{device!~"lo|veth.*|docker.*"}[5m])
查询出站：rate(node_network_transmit_bytes_total{device!~"lo|veth.*|docker.*"}[5m])
图例：In: {{device}}, Out: {{device}}
单位：bytes(SI)
显示格式：Bits/sec
```

### 系统负载

```
数据源：Prometheus
查询1：node_load1 (别名：1 min)
查询2：node_load5 (别名：5 min)
查询3：node_load15 (别名：15 min)
图例：{{instance}} - {{__name__}}
```

## 变量和模板

### 创建变量

1. Dashboard settings → Variables → New
2. 选择类型（Query、Custom等）
3. 配置变量

### 常用变量

```
# 实例选择
Name: instance
Type: Query
Query: label_values(node_cpu_seconds_total, instance)

# 挂载点选择
Name: mountpoint
Type: Query
Query: label_values(node_filesystem_avail_bytes, mountpoint)

# 自定义选项
Name: env
Type: Custom
Values: prod, staging, dev
```

### 使用变量

```promql
# 在查询中使用
100 - (avg by(instance) (irate(node_cpu_seconds_total{instance=~"$instance", mode="idle"}[5m])) * 100)

# 在图例中使用
$instance - CPU Usage
```

## 告警配置

### 创建告警通知渠道

1. Alerting → Notification channels → New channel
2. 选择类型：Email、Slack、Webhook等
3. 配置相关参数
4. 测试通知

### 邮件通知配置

```ini
# grafana.ini or environment variables
[smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = your-password
from_address = grafana@example.com
from_name = Grafana
```

### 在Panel中配置告警

1. 编辑Panel → Alert tab
2. 创建告警规则
```
条件：WHEN avg() OF query(A, 5m, now) IS ABOVE 80
持续时间：For 5m
通知：选择通知渠道
```

3. 配置通知消息
```
标题：{{alertname}} - {{state}}
消息：High CPU usage on {{instance}}
```

## 导入Dashboard

### 从Grafana Labs导入

1. 访问 https://grafana.com/grafana/dashboards
2. 搜索需要的Dashboard（如Node Exporter）
3. 复制Dashboard ID
4. Grafana → + → Import → 粘贴ID
5. 配置数据源
6. 点击Import

### 推荐Dashboard

| ID | 名称 | 用途 |
|----|------|------|
| 1860 | Node Exporter Full | Linux主机监控 |
| 405 | MySQL | MySQL监控 |
| 12708 | Nginx | Nginx监控 |
| 7249 | Redis | Redis监控 |
| 14282 | Docker | Docker容器监控 |
| 315 | Kubernetes | K8s集群监控 |

## 导出Dashboard

1. 打开Dashboard
2. 点击Share dashboard
3. 选择Export
4. 保存JSON文件

## 用户和权限管理

### 创建组织

```bash
# 使用CLI
grafana-cli admin org create YourOrg

# 使用API
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"MyOrg"}' \
  http://admin:admin@localhost:3000/api/orgs
```

### 创建用户

```bash
# 使用CLI
grafana-cli admin create-user \
  --email user@example.com \
  --password user123 \
  --username user

# 使用API
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"User","email":"user@example.com","login":"user","password":"user123"}' \
  http://admin:admin@localhost:3000/api/admin/users
```

### 分配权限

1. Configuration → Users
2. 编辑用户
3. 分配角色（Viewer、Editor、Admin）

## 配置文件

```ini
# /etc/grafana/grafana.ini

[server]
http_port = 3000
domain = grafana.example.com

[database]
type = sqlite3
path = /var/lib/grafana/grafana.db

[security]
admin_user = admin
admin_password = admin123
secret_key = SW2YcwTIb9zpOOhoPsMm

[users]
allow_sign_up = false
auto_assign_org = true
auto_assign_org_role = Viewer

[auth.anonymous]
enabled = false

[log]
mode = console
level = info

[metrics]
enabled = true
disable_total_stats = false
```

## API使用

### 获取Dashboard列表

```bash
curl -H "Authorization: Bearer API_KEY" \
  http://localhost:3000/api/search?query=&type=dash-db
```

### 创建API Key

```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"apikey","role":"Admin"}' \
  http://admin:admin@localhost:3000/api/auth/keys
```

## 完整监控Dashboard示例

### 行1：总览

```
Panel 1: Stat - 在线主机数
Query: count(count by(instance)(up))

Panel 2: Stat - 平均CPU
Query: avg(100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100))

Panel 3: Stat - 总内存
Query: sum(node_memory_MemTotal_bytes) / 1024 / 1024 / 1024

Panel 4: Gauge - 磁盘使用率
Query: 100 - ((sum(node_filesystem_avail_bytes{mountpoint="/"}) / sum(node_filesystem_size_bytes{mountpoint="/"})) * 100)
```

### 行2：CPU趋势

```
Panel: Time Series
Query: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Legend: {{instance}}
Unit: Percent (0-100)
```

### 行3：内存使用

```
Panel: Time Series
Query: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / 1024 / 1024 / 1024
Legend: {{instance}}
Unit: Gigabytes
```

### 行4：磁盘使用

```
Panel: Table
Columns: Instance, Mountpoint, Usage %
Query: 100 - ((node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100)
```

### 行5：网络流量

```
Panel: Time Series (两个查询)
Query A (Inbound): rate(node_network_receive_bytes_total{device=~"eth.*|ens.*"}[5m])
Query B (Outbound): -rate(node_network_transmit_bytes_total{device=~"eth.*|ens.*"}[5m])
Unit: bytes(SI)
Display: Bits/sec
```

## 性能优化

```ini
# 减少数据点
max_data_points = 500

# 缓存查询
[database]
cache_mode = "private"

# 增加连接池
[datasource.prometheus]
max_idle_connections = 100
max_open_connections = 100
```

## 故障排查

```bash
# 查看日志
docker logs grafana

# 检查数据源连接
curl http://prometheus:9090/api/v1/status/runtimeinfo

# 测试查询
curl 'http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up'

# 检查权限
ls -la /var/lib/grafana
```

## 学习检查清单

- [ ] 能够安装和配置Grafana
- [ ] 能够添加数据源
- [ ] 能够创建基础Dashboard
- [ ] 了解常用图表类型
- [ ] 能够使用变量和模板
- [ ] 能够配置告警
- [ ] 能够导入/导出Dashboard
- [ ] 了解用户权限管理
