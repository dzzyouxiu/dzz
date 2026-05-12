# ELK日志系统

## ELK Stack简介

### 组件介绍

| 组件 | 功能 | 端口 |
|------|------|------|
| Elasticsearch | 分布式搜索和分析引擎 | 9200 |
| Logstash | 数据收集和处理管道 | 5044 |
| Kibana | 数据可视化界面 | 5601 |
| Beats | 轻量级数据采集器 | 各种 |

### 数据流

```
日志源 → Beats → Logstash → Elasticsearch → Kibana
              ↓              ↓
         Filebeat      过滤/转换
```

## Docker部署ELK

### docker-compose.yml

```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    ulimits:
      memlock:
        soft: -1
        hard: -1

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
    ports:
      - "5044:5044"
      - "9600:9600"
    environment:
      - "LS_JAVA_OPTS=-Xms256m -Xmx256m"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  es_data:
```

### Logstash配置

```yaml
# logstash/config/logstash.yml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
```

```ruby
# logstash/pipeline/logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [type] == "nginx" {
    grok {
      match => { "message" => '%{IPORHOST:clientip} %{NGUSER:ident} %{NGUSER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response} (?:%{NUMBER:bytes}|-) (?:"(?:%{URI:referrer}|-)"|%{QS:referrer}) %{QS:agent}' }
    }
    date {
      match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
    }
    geoip {
      source => "clientip"
    }
    useragent {
      source => "agent"
      target => "useragent"
    }
  }

  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGLINE}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```

## Filebeat安装配置

### 安装Filebeat

```bash
# Debian/Ubuntu
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.11.0-amd64.deb
sudo dpkg -i filebeat-8.11.0-amd64.deb

# CentOS/RHEL
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.11.0-x86_64.rpm
sudo rpm -vi filebeat-8.11.0-x86_64.rpm

# Docker
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/var/log:/var/log:ro" \
  --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
  docker.elastic.co/beats/filebeat:8.11.0
```

### 配置Filebeat

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    fields:
      type: nginx

  - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/messages
    fields:
      type: syslog

  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 1

setup.kibana:
  host: "http://kibana:5601"

output.logstash:
  hosts: ["logstash:5044"]

# 或者直接输出到Elasticsearch
# output.elasticsearch:
#   hosts: ["http://elasticsearch:9200"]

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```

### 启用Filebeat模块

```bash
# 列出可用模块
filebeat modules list

# 启用nginx模块
filebeat modules enable nginx

# 配置nginx模块
# /etc/filebeat/modules.d/nginx.yml
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]
```

## Grok模式

### 常用Grok模式

```
# IP地址
%{IP:client_ip}

# 时间戳
%{HTTPDATE:timestamp}

# 数字
%{NUMBER:response:int}
%{NUMBER:bytes:int}

# URI
%{URIPATH:path}
%{URIPARAM:query}

# 用户代理
%{QS:user_agent}

# 组合
%{IP:client} %{WORD:method} %{URIPATH:path}
```

### Nginx日志模式

```
# 标准Nginx combined格式
%{IPORHOST:clientip} %{NGUSER:ident} %{NGUSER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent}

# 解析示例
输入：192.168.1.1 - - [24/Feb/2026:10:30:00 +0800] "GET /api/users HTTP/1.1" 200 1234 "https://example.com/" "Mozilla/5.0..."

解析后：
{
  "clientip": "192.168.1.1",
  "timestamp": "24/Feb/2026:10:30:00 +0800",
  "verb": "GET",
  "request": "/api/users",
  "response": "200",
  "bytes": "1234"
}
```

### 测试Grok模式

在线工具：https://grokdebug.herokuapp.com

```bash
# 使用logstash测试
bin/logstash -e 'input { stdin {} } filter { grok { match => { "message" => "%{IP:client}" } } } output { stdout {} }'
```

## Logstash过滤器

### Mutate过滤器

```ruby
filter {
  mutate {
    # 重命名字段
    rename => { "old_field" => "new_field" }
    
    # 删除字段
    remove_field => [ "field1", "field2" ]
    
    # 更新字段
    update => { "field" => "new_value" }
    
    # 替换字段
    replace => { "field" => "value" }
    
    # 转换类型
    convert => { "bytes" => "integer" }
    
    # 合并字段
    merge => { "dest_field" => "added_field" }
  }
}
```

### Date过滤器

```ruby
filter {
  date {
    match => [ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" ]
    target => "@timestamp"
    timezone => "Asia/Shanghai"
  }
}
```

### JSON过滤器

```ruby
filter {
  json {
    source => "message"
    target => "json_content"
  }
}
```

### GeoIP过滤器

```ruby
filter {
  geoip {
    source => "clientip"
    target => "geoip"
    database => "/path/to/GeoLite2-City.mmdb"
    add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
    add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
  }
  mutate {
    convert => [ "[geoip][coordinates]", "float"]
  }
}
```

### User Agent过滤器

```ruby
filter {
  useragent {
    source => "agent"
    target => "useragent"
  }
}
```

## Kibana使用

### 创建索引模式

1. 访问 http://localhost:5601
2. Management → Stack Management → Index Patterns
3. Create index pattern
4. 输入模式：filebeat-*
5. 选择时间字段：@timestamp
6. Create index pattern

### Discover探索日志

1. 点击左侧菜单Discover
2. 选择索引模式
3. 设置时间范围
4. 搜索和过滤

### KQL查询

```kql
# 精确匹配
response:404

# 模糊匹配
message:*error*

# 范围查询
response:>=400 AND response:<500

# 逻辑操作
type:nginx AND response:404

# IP范围
clientip:192.168.1.0/24

# 存在性检查
_exists_:geoip.country_name

# 通配符
path:/api/*
```

### 创建可视化

1. Visualize → Create visualization
2. 选择图表类型
3. 配置数据源和字段
4. 保存到Dashboard

### 常用图表

| 图表类型 | 用途 |
|----------|------|
| Area | 请求趋势图 |
| Vertical Bar | 状态码分布 |
| Pie | 来源IP分布 |
| Table | 详细日志列表 |
| Maps | 地理位置分布 |
| Tag Cloud | 关键词云 |

### 创建Dashboard

1. Dashboard → Create dashboard
2. 添加已保存的可视化
3. 调整布局
4. 设置时间范围选择器
5. 保存Dashboard

## 监控告警

### 创建Watcher

1. Stack Management → Alerting → Watches
2. Create watch
3. 配置触发条件和动作

### 阈值告警示例

```json
{
  "trigger": {
    "schedule": {
      "interval": "1m"
    }
  },
  "input": {
    "search": {
      "request": {
        "indices": ["filebeat-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {"match": {"response": 500}},
                {"range": {"@timestamp": {"gte": "now-1m"}}}
              ]
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {"gt": 10}
    }
  },
  "actions": {
    "email_admin": {
      "email": {
        "to": "admin@example.com",
        "subject": "High error rate detected",
        "body": "Errors: {{ctx.payload.hits.total}}"
      }
    }
  }
}
```

## 性能优化

### Elasticsearch优化

```yaml
# elasticsearch.yml
indices.memory.index_buffer_size: 30%
indices.queries.cache.size: 10%

thread_pool.search.size: 32
thread_pool.write.size: 32
```

### Logstash优化

```yaml
# logstash.yml
pipeline.workers: 4
pipeline.batch.size: 125
pipeline.batch.delay: 50
```

### Filebeat优化

```yaml
# filebeat.yml
queue.mem:
  events: 4096
  flush.min_events: 512
  flush.timeout: 1s

output.elasticsearch:
  worker: 2
  bulk_max_size: 50
```

## 常见问题排查

```bash
# 检查Elasticsearch状态
curl http://localhost:9200/_cluster/health?pretty

# 查看索引
curl http://localhost:9200/_cat/indices?v

# 检查Filebeat日志
journalctl -u filebeat

# 测试配置文件
filebeat test config
filebeat test output

# 查看Logstash日志
docker logs logstash
```

## 学习检查清单

- [ ] 理解ELK Stack各组件功能
- [ ] 能够使用Docker部署ELK
- [ ] 能够安装和配置Filebeat
- [ ] 理解Grok模式语法
- [ ] 能够编写Logstash过滤器
- [ ] 能够在Kibana创建索引模式
- [ ] 能够使用KQL查询日志
- [ ] 能够创建可视化图表
- [ ] 能够创建Dashboard
- [ ] 了解基本的性能优化方法
