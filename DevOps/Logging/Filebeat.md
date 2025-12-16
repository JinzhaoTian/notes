Filebeat 是一个轻量级的日志数据收集器，属于 Elastic Stack（ELK Stack）的一部分，由 Elastic 公司开发。它专门用于转发和集中日志数据。

![](_imgs/Pasted%20image%2020250707224721.png)

## 主要功能

1. **日志收集**：监控指定的日志文件或位置
2. **日志传输**：将日志事件发送到 Elasticsearch 或 Logstash 进行进一步处理
3. **轻量高效**：占用系统资源少，适合长期运行
4. **可靠性**：具备处理中断和重新连接的能力


## 工作原理

Filebeat 通过以下步骤工作：
- 启动一个或多个输入（inputs）来监控日志文件位置
- 为每个找到的日志文件启动收集器（harvester）
- 收集器逐行读取日志文件并将新数据发送到输出（output）

## 常见用途

- 将服务器日志发送到 Elasticsearch 或 Logstash
- 集中管理分布式系统的日志
- 作为日志管道的第一部分，与 Logstash 配合使用

## 主要特点

- 支持多种模块（如 [Nginx](../../Backend/架构设计/Nginx.md)、MySQL、System 等），可快速配置
- 内置支持压缩和加密传输
- 可处理日志轮转和文件重命名
- 提供负载平衡和重试机制


## 安装

1. **下载**：从[Filebeat官网](https://www.elastic.co/downloads/beats/filebeat)下载最新版本
2. **配置**：在文件 `filebeat.yml` 中配置
```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - D:\YourAppPath\Logs\*.json # 指向你的 .NET 应用日志目录
  json.keys_under_root: true # JSON日志处理
  json.add_error_key: true
  tags: ["api"] # 添加标签便于筛选

output.elasticsearch:
  hosts: ["localhost:9200"]
  username: "elastic" # 替换为你的用户名
  password: "your_password" # 替换为你的密码
  indices:
    - index: "api-logs-%{+yyyy.MM.dd}" # 按日期分索引

setup.kibana:
  host: "localhost:5601"
```
3. **启动**：
	- Windows：以管理员身份运行 PowerShell 并执行：
```powershell
cd "C:\Program Files\Filebeat"
.\filebeat.exe -c filebeat.yml setup # 加载索引模板等
.\filebeat.exe -c filebeat.yml # 启动Filebeat
# 或安装为Windows服务（推荐）：
.\filebeat.exe install service -c filebeat.yml
Start-Service filebeat
```

