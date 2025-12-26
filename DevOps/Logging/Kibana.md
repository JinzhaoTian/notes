Kibana 是一个开源的数据可视化和分析平台，专为与 Elasticsearch（一个分布式搜索和分析引擎）协同工作而设计。

Kibana 是 Elastic Stack（原 ELK Stack，包括 Elasticsearch、Logstash、Kibana 和 Beats）的核心组件之一，主要用于交互式地探索、分析和可视化存储在 Elasticsearch 中的数据。

## 主要功能

1. **数据可视化**
    - 提供丰富的图表类型（柱状图、折线图、饼图、地图等），帮助用户直观理解数据。
    - 支持自定义仪表盘（Dashboards），将多个可视化图表整合到一个界面中。

2. **数据探索与分析**
    - 通过 **Discover** 功能直接查询 Elasticsearch 中的数据，支持全文搜索、过滤和字段统计。
    - 使用 **Aggregations**（聚合）对数据进行分组、统计和趋势分析。

3. **日志和指标分析**
    - 常用于运维监控，可视化日志（通过 Filebeat 收集）或系统指标（通过 Metricbeat 收集）。
    - 与 **APM**（应用性能监控）集成，分析应用程序性能数据。

4. **机器学习集成**
    - 结合 Elasticsearch 的机器学习功能，检测数据中的异常模式（如流量突增、日志错误激增）。

5. **地理空间分析**
    - 支持地图可视化，展示基于地理位置的数据（如用户分布、服务器地理位置）。

6. **告警与通知**
    - 通过 **Kibana Alerting** 设置规则，在数据达到阈值时触发告警（如发送邮件或 Slack 通知）。


## 安装

1. **下载**：从 [Kibana 官网](https://www.elastic.co/downloads/kibana) 下载最新版本
2. **配置**：从文件 `config/kibana.yml` 中配置
```yaml
server.port: 5601
server.host: "0.0.0.0"

elasticsearch.hosts: ["https://localhost:9200"]

# 1. 使用 username / password, elasticsearch 默认用户 elastic 不能使用
elasticsearch.username: "kibana" # 使用Kibana用户
elasticsearch.password: "your_kibana_password"

# 2. 使用 Token
elasticsearch.serviceAccountToken: "your_kibana_token"

elasticsearch.ssl.certificateAuthorities: [ "C:/Program Files/elasticsearch-9.2.2/config/certs/http_ca.crt" ]


i18n.locale: "zh-CN" # 可选：中文界面
```
3. **启动**：
	- Linux/macOS：运行 `bin/kibana` 
	- Windows：运行 `bin\kibana.bat` 
4. 访问：`http://localhost:5601`



## 使用

1. **访问 Kibana**：打开浏览器，访问 `http://localhost:5601`。
2. **创建索引模式**：进入 **Stack Management > 索引模式 > 创建索引模式**，输入 `dotnet-api-logs-*`，选择 `@timestamp` 字段作为时间字段。
3. **发现日志**：进入 **Analytics > Discover**，选择你创建的索引模式，即可搜索、筛选和查看所有日志。
    
4. **创建可视化与仪表板**：
    
    - 进入 **Analytics > Dashboard** 创建新的仪表板。
        
    - 点击 “创建可视化”，选择如 **TSVB (时间序列可视化构建器)** 或 **Lens**。
        
    - 可以创建以下有用的图表：
        
        - **请求量/错误率趋势图**（基于时间）
            
        - **HTTP状态码分布**（饼图）
            
        - **慢请求追踪**（筛选 `Duration` 大于某阈值的日志）
            
        - **异常类型统计**（筛选 `Level: Error` 并聚合 `Exception` 字段）