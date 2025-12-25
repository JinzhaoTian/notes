Elasticsearch 是一个开源的分布式搜索和分析引擎，基于 Apache Lucene 构建。它被设计用于处理大规模数据的实时搜索和分析需求。


## 安装

1. **下载**：从 [Elasticsearch官网]([Download Elasticsearch | Elastic](https://www.elastic.co/downloads/elasticsearch))下载最新版本
2. **配置**：在 `config/elasticsearch.yml` 中配置
```yaml
cluster.name: production-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node # 单节点模式（生产环境建议集群）
xpack.security.enabled: true # 启用基础安全（强烈建议）
```
3. **启动**：
	- Linux/macOS：运行 `bin/elasticsearch` 
	- Windows：运行 `bin\elasticsearch.bat` 
4. **初始设置**：首次启动后，执行以下命令设置内置用户密码：
```bash
bin/elasticsearch-setup-passwords interactive
```


## 命令

1. **安装**：
```bash
elasticsearch-service.bat install
```

2. **启动服务**：
```bash
elasticsearch-service.bat start
```

3. **停止服务**：
```bash
elasticsearch-service.bat stop
```

4. **卸载**：
```bash
elasticsearch-service.bat remove
```