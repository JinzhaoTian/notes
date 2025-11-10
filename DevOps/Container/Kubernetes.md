Kubernetes 是一个开源的容器编排平台，用于自动化应用的部署、扩展和管理。它最初由Google开发，后来捐赠给Cloud Native Computing Foundation（CNCF）。如果想要在生产环境中大批量的使用容器，还离不开的容器的编排技术，Kubernetes 就是帮助用户管理容器化的应用程序，并简化其在多种环境（如本地服务器、云服务等）中的运行。

![](imgs/Pasted%20image%2020240906112923.png)

Kubernetes 是舵手的意思，我们把 Docker 比喻成一个个集装箱，而 Kubernetes 正是运输这些集装箱的舵手。


### 核心功能

1. **容器编排**
    - 自动化管理容器（如 Docker）的生命周期，包括部署、调度、伸缩和负载均衡。
    - 解决多容器跨主机部署的复杂性问题。

2. **高可用与容错**
    - 自动重启失败的容器、替换不可用的节点，确保应用持续运行。
    - 支持滚动更新和回滚，避免服务中断。

3. **弹性伸缩**
    - 根据 CPU、内存等指标或自定义规则自动扩缩容（Horizontal Pod Autoscaler）。
    - 支持集群节点的动态增减（如云厂商的自动伸缩组）。

4. **服务发现与负载均衡**
    - 通过内置的 DNS 和 Service 机制，自动分配 IP 和域名，实现流量分发。

5. **配置与密钥管理**
    - 通过 `ConfigMap` 和 `Secret` 集中管理配置和敏感信息，避免硬编码。

6. **多云与混合云支持**
    - 可在公有云、私有云或本地数据中心统一部署，提供一致的运行环境。


### 核心概念

- **Pod** ：Kubernetes 的最小调度单元，包含一个或多个共享资源的容器（如网络、存储）。
- **Deployment**  ：定义应用的副本数、更新策略等，确保 Pod 的期望状态与实际状态一致。
- **Service** ：为一组 Pod 提供稳定的访问入口（如 ClusterIP、NodePort、LoadBalancer）。
- **Ingress** ：管理外部访问集群服务的路由规则（如 HTTP/HTTPS）。
- **Node** ：集群中的工作节点，运行 Pod 的虚拟机或物理机。
- **Control Plane（控制平面）**  
    - **kube-apiserver**：集群的入口，处理 REST 操作。
    - **etcd**：分布式键值存储，保存集群状态。
    - **kube-scheduler**：决定 Pod 在哪个节点运行。
    - **kube-controller-manager**：监控集群状态并调整（如节点故障恢复）。



## 安装 Kubernetes

安装 Kubernetes 有几种方式，取决于你的需求和环境。以下是常见的几种方法：

- **Minikube**：适合本地开发，单节点 Kubernetes 集群。
    - 安装 Minikube：`minikube start`
    - 安装 kubectl：`brew install kubectl`（Mac），或参考 Kubernetes 文档 安装。
- **Kubeadm**：适合手动配置 Kubernetes 集群。
    - 使用 `kubeadm` 安装 Kubernetes 集群，详细步骤可以参考官方文档。
- **云托管 Kubernetes 服务**：许多云服务提供商（如 AWS、Google Cloud 和 Azure）提供托管的 Kubernetes 服务（如 EKS、GKE、AKS）。你可以通过这些平台快速启动和运行 Kubernetes 集群。

## 配置 Kubernetes 集群

在安装 Kubernetes 并启动集群后，你可以通过 `kubectl` 命令行工具与 Kubernetes 集群进行交互。`kubectl` 允许你管理集群资源、查看状态、部署应用等。

- 检查集群状态：
```bash
kubectl cluster-info
```

- 列出所有节点：
```bash
kubectl get nodes
```

## 创建并部署应用

Kubernetes 使用 **YAML 配置文件** 来定义应用和资源，如部署、服务、配置等。你需要创建这些 YAML 文件，然后使用 `kubectl` 将其应用到集群。

- **编写部署文件**（`deployment.yaml`）：
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:1.19
        ports:
        - containerPort: 80
```

- **部署应用到集群**：
```bash
kubectl apply -f deployment.yaml
```

- **查看部署状态**：
```bash
kubectl get deployments
```

## 管理服务

为了让外界访问你的应用，Kubernetes 提供了 **Service**，用于定义网络访问方式（如 ClusterIP、NodePort、LoadBalancer）。

- **创建服务文件**（`service.yaml`）：
```bash
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

- **应用服务配置**：
```bash
kubectl apply -f service.yaml
```

- **查看服务状态**：
```bash
kubectl get services
```

## 监控和日志管理

Kubernetes 提供了监控工具，可以查看 Pod 的日志和应用的健康状态。

- **查看 Pod 日志**：
```bash
kubectl logs <pod-name>
```

- **监控 Pod 状态**：
```bash
kubectl get pods --watch
```

## 自动扩展

Kubernetes 支持根据负载自动扩展 Pod。

- **创建自动扩展器**：
```bash
kubectl autoscale deployment my-app --min=2 --max=10 --cpu-percent=80
```

## 管理存储

Kubernetes 允许你为容器化应用提供持久存储。
- **创建 PersistentVolume（PV）** 和 **PersistentVolumeClaim（PVC）** 来管理存储。

## 滚动更新和回滚

- **滚动更新应用**：
```bash
kubectl set image deployment/my-app my-container=nginx:1.20
```

- **回滚更新**：
```bash
kubectl rollout undo deployment/my-app
```


