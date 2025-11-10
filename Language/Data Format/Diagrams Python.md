Diagrams 是一个基于 Python 的开源库，用于通过代码绘制云系统架构图。它使用简单的 Python 代码生成图表，支持多种云服务提供商（如 AWS、Azure、GCP、Kubernetes 等）的图标，并允许用户创建专业级的系统架构可视化图表，而无需手动拖拽图形工具（如 Visio 或 Draw.io）。

## 核心特点

1. **代码即图表**：用 Python 编写逻辑，自动生成图表。
2. **丰富的节点库**：支持 AWS、Azure、GCP、阿里云、Kubernetes 等服务的预置图标。
3. **灵活的输出格式**：生成 PNG、JPG、SVG 等格式的图片。
4. **基于 Graphviz**：底层依赖 Graphviz 进行图形渲染，布局自动化。


## 安装方法

```bash
pip install diagrams
```
**注意**：需要提前安装 [Graphviz](https://graphviz.org/download/)（确保其二进制文件在系统路径中）。


## 基础用法示例

#### 1. 绘制一个简单的 Web 服务架构
```python
from diagrams import Diagram
from diagrams.aws.compute import EC2
from diagrams.aws.database import RDS
from diagrams.aws.network import ELB

with Diagram("Web Service", show=False):
    ELB("Load Balancer") >> EC2("Web Server") >> RDS("Database")
```
运行后会生成如下架构图：
```
[Load Balancer] → [Web Server] → [Database]
```

#### 2. 复杂架构（带分组）
```python
from diagrams import Diagram, Cluster
from diagrams.aws.compute import ECS
from diagrams.aws.database import ElastiCache, RDS
from diagrams.aws.network import Route53

with Diagram("Microservices", show=False):
    dns = Route53("DNS")
    with Cluster("App Cluster"):
        svc_group = [ECS("Service 1"),
                     ECS("Service 2")]
    db = RDS("User Database")
    cache = ElastiCache("Cache")

    dns >> svc_group >> db
    svc_group - cache  # 虚线连接
```


## 关键组件

- **Diagram**：主类，定义一个图表。
- **Cluster**：创建分组（如虚线框内的逻辑集合）。
- **节点**：从模块导入（如 `diagrams.aws.compute.EC2`）。
- **连接**：用 `>>`（实线）、`-`（虚线）等操作符表示组件关系。

## 适用场景

- 自动化文档生成（如 CI/CD 流程中更新架构图）。
- 快速原型设计云架构。
- 教学或演示中的可视化辅助。



## 优缺点

- **优点**：
  - 版本可控（代码即图表，适合 Git 管理）。
  - 避免手动调整图形工具的繁琐操作。
- **缺点**：
  - 复杂布局需依赖 Graphviz 的自动排列。
  - 自定义样式（如颜色、线条）需要学习 Graphviz 语法。


## 扩展资源

- [官方 GitHub](https://github.com/mingrammer/diagrams)
- [图标库列表](https://diagrams.mingrammer.com/docs/nodes/aws)

通过 Diagrams，开发者可以高效地将架构设计融入开发流程，提升文档的可维护性。