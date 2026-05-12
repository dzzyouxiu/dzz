# Prometheus监控系统

## Prometheus简介

### 什么是Prometheus
- SoundCloud开发的开源监控系统
- 基于时间序列数据库
- 多维度数据模型
- 强大的PromQL查询语言
- 拉取式（Pull）数据采集

### 架构图

```
                  ┌─────────────────────┐
                  │      用户/系统       │
                  └──────────┬──────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼────┐        ┌─────▼─────┐       ┌─────▼─────┐
    │ Grafana │        │ Alertmanager│       │ Web UI   │
    └────┬────┘        └─────┬─────┘       └───────────┘
         │                   │
         └─────────┬─────────┘
                   │
            ┌──────▼──────┐
            │  Prometheus │
            │   Server    │
            └──────┬──────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
┌───▼───┐    ┌────▼────┐    ┌────▼────┐
│Node   │    │MySQL    │    │Nginx    │
│Exporter│   │Exporter │    │Exporter │
└───────┘    └─────────┘    └─────────┘
```

## 安装Prometheus

### Docker安装（推荐）

```yaml
# docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:v2.47.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    restart: unless-stopped

volumes:
  prometheus_data:
```

### 二进制安装

```bash
# 下载
wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz

# 解压
tar xzf prometheus-2.47.0.linux-amd64.tar.gz
cd prometheus-2.47.0.linux-amd64/

# 复制到系统目录
sudo cp prometheus promtool /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo cp -r consoles console_libraries /etc/prometheus/

# 创建系统服务
sudo tee /etc/systemd/system/prometheus.service <<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF

# 启动服务
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

## 配置文件

### 基础配置 prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node_exporter:9100']

  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx_exporter:9113']

  - job_name: 'mysql'
    static_configs:
      - targets: ['mysqld_exporter:9104']
```

## 常用Exporter

### Node Exporter（监控Linux主机）

```bash
# Docker运行
docker run -d \
  --name node_exporter \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter:v1.6.1 \
  --path.rootfs=/host

# 二进制安装
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xzf node_exporter-1.6.1.linux-amd64.tar.gz
sudo cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
node_exporter &
```

### MySQL Exporter

```bash
# 创建数据库用户
mysql -u root -p <<EOF
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'password';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
EOF

# Docker运行
docker run -d \
  --name mysqld_exporter \
  -e DATA_SOURCE_NAME="exporter:password@(mysql-host:3306)/" \
  prom/mysqld-exporter:v0.15.0
```

### Nginx Exporter

```bash
# 需要nginx开启stub_status
# nginx.conf
server {
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}

# Docker运行
docker run -d \
  --name nginx_exporter \
  -e SCRAPE_URI=http://nginx-host/nginx_status \
  nginx/nginx-prometheus-exporter:0.11
```

### Redis Exporter

```bash
# Docker运行
docker run -d \
  --name redis_exporter \
  -e REDIS_ADDR=redis-host:6379 \
  oliver006/redis_exporter:v1.50.0
```

## PromQL查询语言

### 基础查询

```promql
# 查询所有时间序列
up

# 查询特定job
up{job="node"}

# 查询特定实例
up{instance="localhost:9100"}

# CPU使用率
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 内存使用率
100 - ((node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100)

# 磁盘使用率
100 - ((node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100)
```

### 函数使用

```promql
# 增长率（5分钟）
rate(node_network_receive_bytes_total[5m])

# 瞬时增长率
irate(node_network_receive_bytes_total[5m])

# 求和
sum(rate(http_requests_total[5m])) by (method)

# 平均值
avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance)

# 最大值
max(rate(http_requests_total[5m])) by (path)

# 分位数
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# 时间聚合
avg_over_time(node_memory_MemFree_bytes[1h])
```

### 常用指标查询

```promql
# CPU使用率
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 内存使用量
node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes

# 磁盘使用率
100 - ((node_filesystem_avail_bytes{mountpoint!~"/run/.*"} / node_filesystem_size_bytes) * 100)

# 网络入站流量（字节/秒）
rate(node_network_receive_bytes_total{device!~"lo|docker.*"}[5m])

# 网络出站流量（字节/秒）
rate(node_network_transmit_bytes_total{device!~"lo|docker.*"}[5m])

# 系统负载
node_load1

# 进程数
node_procs_running

# 文件描述符使用
process_open_fds / process_max_fds * 100
```

## 告警规则

### 基础规则文件

```yaml
# rules/alerts.yml
groups:
  - name: node_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Instance {{ $labels.instance }} CPU usage high"
          description: "CPU usage is above 80%\n  VALUE = {{ $value }}"

      - alert: HighMemoryUsage
        expr: 100 - ((node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100) > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Instance {{ $labels.instance }} memory usage high"
          description: "Memory usage is above 85%\n  VALUE = {{ $value }}"

      - alert: HighDiskUsage
        expr: 100 - ((node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100) > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} disk usage high"
          description: "Disk usage is above 90%\n  VALUE = {{ $value }}"

      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

  - name: mysql_alerts
    rules:
      - alert: MysqlDown
        expr: mysql_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "MySQL instance {{ $labels.instance }} is down"

      - alert: MysqlHighConnections
        expr: mysql_global_status_threads_connected / mysql_global_variables_max_connections * 100 > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "MySQL connections high"
          description: "Connections usage is {{ $value }}%"
```

## Alertmanager配置

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'admin@example.com'
        send_resolved: true

  - name: 'webhook'
    webhook_configs:
      - url: 'http://webhook:5001/alert'

  - name: 'dingtalk'
    webhook_configs:
      - url: 'https://oapi.dingtalk.com/robot/send?access_token=xxx'
        http_config:
          headers:
            Content-Type: application/json
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

## 完整Docker Compose示例

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:v2.47.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.26.0
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    restart: unless-stopped

  node_exporter:
    image: prom/node-exporter:v1.6.1
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.1.0
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

## 热加载配置

```bash
# 重新加载配置（需要--web.enable-lifecycle）
curl -X POST http://localhost:9090/-/reload

# 检查配置
promtool check config prometheus.yml

# 检查规则
promtool check rules rules/alerts.yml
```

## 常用操作

```bash
# 查看Prometheus状态
curl http://localhost:9090/api/v1/status/runtimeinfo

# 查看所有targets
curl http://localhost:9090/api/v1/targets

# 执行PromQL查询
curl 'http://localhost:9090/api/v1/query?query=up'

# 查看告警状态
curl http://localhost:9090/api/v1/alerts
```

## 学习检查清单

- [ ] 理解Prometheus架构和原理
- [ ] 能够安装和配置Prometheus
- [ ] 了解常用Exporter并能安装使用
- [ ] 能够编写基础PromQL查询
- [ ] 能够编写告警规则
- [ ] 能够配置Alertmanager
- [ ] 能够使用Docker Compose部署
- [ ] 理解时间序列和标签概念
