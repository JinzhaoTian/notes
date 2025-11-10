Grafana 是一款开源**数据可视化和监控分析工具**，主要用于将时序数据（如指标、日志等）通过图表、仪表盘等形式直观展示，帮助用户实时监控和分析系统性能、应用程序状态或业务数据。

![](../Logging/imgs/Pasted%20image%2020250704182135.png)

## 核心功能

1. **数据可视化**
    - 支持多种图表类型（折线图、柱状图、仪表盘、热力图等）。
    - 可自定义仪表盘，通过拖拽组件快速构建可视化界面。

2. **多数据源支持**
    - 兼容多种数据库和时序数据库，如 [Prometheus](Prometheus.md)、InfluxDB、[Elasticsearch](../Logging/Elasticsearch.md)、[MySQL](../../Database/MySQL/MySQL.md)、[PostgreSQL](../../Database/PostgreSQL/PostgreSQL.md) 等。
    - 支持云服务商的数据源（如 AWS CloudWatch、Google BigQuery）。

3. **告警与通知**
    - 可设置阈值触发告警，通过邮件、Slack、PagerDuty 等渠道通知用户。

4. **插件生态**
    - 提供丰富的官方和社区插件，扩展数据源或可视化功能。

5. **协作与共享**
    - 支持团队协作编辑仪表盘，可通过链接或快照分享可视化结果。

## 应用场景

- **IT 运维监控**：结合 Prometheus 监控服务器、容器（如 Kubernetes）性能。
- **物联网（IoT）**：展示传感器实时数据。
- **业务分析**：可视化数据库中的业务指标（如销售额、用户活跃度）。



## 使用示例


### 安装部署

**方式 1：直接[下载](https://grafana.com/grafana/download)安装**
```bash
./grafana-server
```

**方式 2：Docker 快速启动**
```bash
# 启动 Prometheus
docker run -d -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

# 启动 Grafana
docker run -d -p 3000:3000 grafana/grafana
```


### 配置数据源（Prometheus）

1. 访问 Grafana（`http://localhost:3000`），默认账号/密码：`admin/admin`。
2. 进入 **Configuration > Data Sources**，点击 **Add data source**。
3. 选择 **Prometheus**，填写以下信息：
    - **URL**: `http://<Prometheus-IP>:9090`（若同机部署，用 `http://localhost:9090`）
    - 其他选项保持默认，点击 **Save & Test**，确认连接成功。


### 导入或创建仪表盘


**方式 1：导入官方模板**：
1. 进入 **Dashboards > New > Import**。
2. 输入模板 ID（如 `1860`，这是 Prometheus 官方推荐的节点监控模板）。
3. 选择 Prometheus 数据源，点击 **Import**。


**方式 2：手动创建仪表盘**：
1. 点击 **Create Dashboard > Add Panel**。
2. 在 **Query** 选项卡中：
    - 数据源选择 `Prometheus`。
    - 输入 PromQL 查询语句（如 `up` 查看服务状态，`node_cpu_seconds_total` 监控 CPU 使用率）。
3. 调整图表类型（如折线图、仪表盘），设置标题和单位。
4. 点击 **Save** 保存仪表盘。


