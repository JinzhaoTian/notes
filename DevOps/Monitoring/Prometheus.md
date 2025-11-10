Prometheus 是一个开源的系统监控和警报工具包，最初由 SoundCloud 开发，现在是 CNCF (云原生计算基金会) 的毕业项目，与 Kubernetes 一起成为云原生生态的核心组件之一。它主要用于收集、存储和查询时间序列数据（如服务器性能指标、应用监控数据等），并提供强大的告警功能。

## 核心特性

1. **时间序列数据库（TSDB）**
	- 专门存储带时间戳的监控数据（如 CPU 使用率、内存占用、请求延迟等）。
	- 数据以 键值对（Metric + Labels） 形式存储，支持高效查询。

2. **灵活的查询语言（PromQL）**
	- 通过 PromQL 可以对时间序列数据进行复杂的聚合、计算和分析。
```promql
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
```

3. **拉取（Pull）模型**
	- Prometheus 主动从目标（如应用、服务器）拉取数据（通过 HTTP 暴露的 /metrics 端点）。
	- 适合监控动态环境（如 Kubernetes），但也可以通过 Pushgateway 支持推送（Push）模式。

4. **多维度数据模型**
	- 每个指标可以附带多个标签（Labels），方便分类和过滤。
```promql
http_requests_total{method="GET", path="/api", status="200"}
```

5. **告警管理（Alertmanager）**
    - 支持基于 PromQL 定义告警规则，触发后由 Alertmanager 统一处理，支持去重、分组、静默，并可通过邮件、Slack、Webhook 等通知。

6. **可视化**
    - 内置简单的图形界面，但通常与 **Grafana** 集成，实现更丰富的仪表盘。



## 核心组件

- **Prometheus Server**：主服务，负责数据采集、存储和查询。
- **Exporters**：暴露监控数据的工具（如 Node Exporter 用于服务器监控，MySQL Exporter 用于数据库监控）。
- **Pushgateway**：临时存储推送的指标（适用于短生命周期任务）。
- **Alertmanager**：处理告警通知。
- **Client Libraries**：支持多种语言（Go、Java、Python 等），方便应用集成自定义指标。


## 适用场景

- **基础设施监控**：服务器、容器（如 Docker、Kubernetes）、数据库等。
- **应用性能监控（APM）**：跟踪 HTTP 请求、错误率、延迟等。
- **业务指标监控**：如用户活跃数、订单量等。


## 使用示例

1. **启动 Prometheus**：配置 `prometheus.yml` 定义监控目标。
2. **暴露指标**：应用通过 `/metrics` 端点提供数据（或使用 Exporter）。
3. **查询**：在 Prometheus Web UI 或 Grafana 中使用 PromQL 查询数据。
4. **告警**：定义规则如 `CPU 使用率 > 90%`，触发告警。


### 安装部署

**方式 1**：**直接下载二进制文件（Linux/macOS）**
```bash
# 下载 Prometheus（替换版本号）
wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
tar -xvf prometheus-2.47.0.linux-amd64.tar.gz
cd prometheus-2.47.0.linux-amd64

# 启动 Prometheus（默认加载当前目录下的 prometheus.yml）
./prometheus --config.file=prometheus.yml
```


**方式 2**：**Docker 运行**
```bash
docker run -d -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```


**方式 3**：**Kubernetes 部署**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```


### 配置监控目标

编辑 `prometheus.yml`，定义要监控的目标（如服务器、应用、数据库等）：
```yaml
global:
  scrape_interval: 15s # 抓取间隔

scrape_configs:
  # 监控 Prometheus 自身
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # 监控 Linux 服务器（需安装 Node Exporter）
  - job_name: "node"
    static_configs:
      - targets: ["192.168.1.100:9100"]  # Node Exporter 默认端口 9100

  # 监控 Kubernetes（需配置 kube-state-metrics 和 cAdvisor）
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```


### 收集指标

#### 监控服务器

**安装 Node Exporter**：
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-*.tar.gz
cd node_exporter-*
./node_exporter
```

#### 监控应用


##### Java

1. **配置**：Maven
```xml
<dependency>
    <groupId>io.prometheus</groupId>
    <artifactId>prometheus-metrics-core</artifactId>
    <version>1.3.0</version>
</dependency>
<dependency>
    <groupId>io.prometheus</groupId>
    <artifactId>prometheus-metrics-exporter-pushgateway</artifactId>
    <version>1.3.0</version>
</dependency>
```

2. **使用**
```java
public class ExampleBatchJob {
    private static PushGateway pushGateway = PushGateway.builder()
            .address("localhost:9091") // not needed as localhost:9091 is the default
            .job("example")
            .build();

    private static Gauge dataProcessedInBytes = Gauge.builder()
            .name("data_processed")
            .help("data processed in the last batch job run")
            .unit(Unit.BYTES)
            .register();

    public static void main(String[] args) throws Exception {
        try {
            long bytesProcessed = processData();
            dataProcessedInBytes.set(bytesProcessed);
        } finally {
            pushGateway.push();
        }
    }

    public static long processData() {
        // Imagine a batch job here that processes data
        // and returns the number of Bytes processed.
        return 42;
    }
}
```




##### Go

通过 Prometheus 监控 Go 的 [Gin](../../Language/Golang/Gin.md) Web 框架

1. **项目依赖**
```
go get github.com/gin-gonic/gin
go get github.com/prometheus/client_golang/prometheus
```

2. **集成 Prometheus**
```go
var (
	httpRequestsTotal = promauto.NewCounterVec(prometheus.CounterOpts{
		Name: "http_requests_total",
		Help: "Count of all HTTP requests",
	}, []string{"method", "path", "status"})
	
	httpRequestDuration = promauto.NewHistogramVec(prometheus.HistogramOpts{
		Name: "http_request_duration_seconds",
		Help: "Duration of HTTP requests",
		Buckets: []float64{0.1, 0.3, 0.5, 0.7, 1, 1.5, 2, 3},
	}, []string{"method", "path"})
)
```

3. **定义监控中间件**
```go
func prometheusMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.FullPath()
		
		c.Next()
		
		duration := time.Since(start).Seconds()
		status := c.Writer.Status()
		
		httpRequestsTotal.WithLabelValues(c.Request.Method,path,http.StatusText(status))
						 .Inc()
		httpRequestDuration.WithLabelValues(c.Request.Method,path)
						   .Observe(duration)
	}
}
```

4. **暴露指标路由**
```go
func main() {
	r := gin.Default()
	
	// 使用 Prometheus 中间件
	r.Use(prometheusMiddleware())
	
	// 添加 Prometheus 指标路由
	r.GET("/metrics", gin.WrapH(promhttp.Handler()))
	
	// 业务路由
	r.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "Hello, Prometheus!"})
	})
	
	r.GET("/api", func(c *gin.Context) {
		// 模拟处理时间
		time.Sleep(100 * time.Millisecond) 
		c.JSON(200, gin.H{"message": "API response"})
	})
	
	r.Run(":8080")
}
```




##### .NET

1. **安装必要的 NuGet 包**
```bash
dotnet add package prometheus-net.AspNetCore
```

2. **配置 ASP.NET Core 应用**

**基本配置**：在 `Program.cs` 中配置 Prometheus：
```cs
using Prometheus;

var builder = WebApplication.CreateBuilder(args);

// 添加其他服务...

var app = builder.Build();

// 配置中间件管道...

// 使用 Prometheus 中间件
app.UseRouting();
app.UseHttpMetrics(); // 捕获 HTTP 指标
app.UseEndpoints(endpoints =>
{
    endpoints.MapMetrics(); // 暴露 metrics 端点
    endpoints.MapControllers();
});

app.Run();
```

**高级配置**：
```cs
app.UseHttpMetrics(options =>
{
    options.RequestDuration.Enabled = true;
    options.InProgress.Enabled = true;
    options.RequestCount.Enabled = true;
    
    // 可以添加自定义标签
    options.AddCustomLabel("host", context => context.Request.Host.Host);
});
```

3. **自定义指标**：
```cs
public class SampleMetrics
{
    private readonly Counter _requestCounter;
    private readonly Histogram _responseTimeHistogram;

    public SampleMetrics()
    {
        _requestCounter = Metrics.CreateCounter(
            "myapp_requests_total", 
            "Total number of requests to my API.");
            
        _responseTimeHistogram = Metrics.CreateHistogram(
            "myapp_response_time_seconds",
            "The duration in seconds between the response to a request.",
            new HistogramConfiguration
            {
                Buckets = Histogram.LinearBuckets(start: 1, width: 1, count: 5)
            });
    }

    public void IncrementRequestCounter()
    {
        _requestCounter.Inc();
    }

    public void RecordResponseTime(double time)
    {
        _responseTimeHistogram.Observe(time);
    }
}
```

4. **使用**
```cs
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly SampleMetrics _metrics = new SampleMetrics();

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        var stopwatch = Stopwatch.StartNew();
        
        // 业务逻辑...
        
        _metrics.IncrementRequestCounter();
        _metrics.RecordResponseTime(stopwatch.Elapsed.TotalSeconds);
        
        return results;
    }
}
```




##### Python

`prometheus-flask-exporter`是一个开源项目，基于 Python 的 [Flask](../../Language/Python/Flask.md) Web 框架，该库能够收集 HTTP 请求指标并能导出到 Prometheus 中，降低开发人员在监控方面的成本。

1. **安装**
```bash
pip install prometheus-flask-exporter
```

2. **快速使用**
```python
import time
import random

from flask import Flask
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
PrometheusMetrics(app)


@app.route("/first_route")
def first_route():
    time.sleep(random.random() * 0.2)
    return "ok"


@app.route("/second_route")
def second_route():
    time.sleep(random.random() * 0.4)
    return "ok"


@app.route("/three_route")
def three_route():
    time.sleep(random.random() * 0.6)
    return "ok"


if __name__ == "__main__":
    app.run("0.0.0.0", 5000, threaded=True)
```


`prometheus-flask-exporter`是支持多进程项目的。有时项目中会使用 gunicore 部署多进程，Gunicorn 启动项目之后一定会有一个主进程 Master 和一个或多个工作进程，在这种多进程应用中，需要使用 multiprocess 模块中的 `GunicornPrometheusMetrics`。





#### 查询数据（PromQL）

在 Prometheus Web UI（`http://localhost:9090/graph`）中输入 PromQL 查询：

- 查看服务器 CPU 使用率：
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100
```

- 统计 HTTP 请求速率：
```promql
rate(http_requests_total[5m])
```


#### 设置告警

1. **定义告警规则**：创建 `alert.rules.yml`：
```yaml
groups:
- name: example
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100 > 80
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is {{ $value }}%"
```
在 `prometheus.yml` 中加载规则：
```yaml
rule_files:
  - "alert.rules.yml"
```


2. **启动 Alertmanager**：下载并配置 `alertmanager.yml`（如邮件通知）：
```yaml
route:
  receiver: email-alert
receivers:
- name: email-alert
  email_configs:
  - to: admin@example.com
    from: alertmanager@example.com
    smarthost: smtp.example.com:587
    auth_username: "user"
    auth_password: "password"
```
启动 Alertmanager：
```bash
./alertmanager --config.file=alertmanager.yml
```



### 可视化（Grafana）