项目技术栈：ASP.NET Core 8.0 + CAP (RabbitMQ) + SQL Server + Redis，进程外托管（Kestrel 在 IIS 反向代理之后）。

## 问题场景

线上部署架构：多个地区，每个地区多个服务实例，每个地区只开放一个服务地址给客户端，任务通过 RabbitMQ 分发到各实例。IIS 应用池已设置启动模式为 "AlwaysRunning"，但未暴露地址的实例仍会被回收。

## 根因分析

### 1. Idle Timeout（空闲超时）

**"AlwaysRunning" 启动模式 ≠ 禁止空闲回收。**

IIS 应用池有两组独立设置：

| 设置 | 作用 | 默认值 |
|------|------|--------|
| **Start Mode: AlwaysRunning** | 控制应用池**何时启动**（IIS 启动时立即拉起 w3wp.exe） | OnDemand |
| **Idle Time-out (minutes)** | 控制应用池**何时关闭**（无 HTTP 请求 N 分钟后关闭） | **20 分钟** |

"AlwaysRunning" 只解决"启动"问题，不解决"保持运行"问题。如果一个实例没有收到任何 HTTP 请求超过 20 分钟（默认），IIS 会主动 `Shutdown` 该 worker process。

**为什么没收到 HTTP 请求却仍在工作？** 

因为任务是通过 CAP 订阅 RabbitMQ 消息来驱动的——CAP 在进程内维护与 RabbitMQ 的长连接来消费消息。这种 TCP 层面的活动对 IIS 是**不可见的**，IIS 只将 HTTP 请求视为"活动"。

```
┌──────────────────────────────┐
│  IIS (w3wp.exe)              │
│                              │
│  ┌────────────────────────┐  │
│  │  ASP.NET Core (Kestrel)│  │
│  │                        │  │
│  │  CAP Subscriber ───────┼──┼──► RabbitMQ (消费消息)
│  │  (TCP 长连接)           │  │     ↑ IIS 看不到这个活动
│  │                        │  │
│  │  HTTP 端点 ─────────────┼──┼──► 客户端请求
│  │                        │  │     ↑ 只有这个重置 idle timer
│  └────────────────────────┘  │
└──────────────────────────────┘
```

结果：即使实例正在消费消息、执行转换/出图/打印任务，超过 20 分钟无 HTTP 请求后，IIS 仍会回收进程。

### 2. 定期回收（Regular Time Interval）

默认每 **1740 分钟（29 小时）** IIS 会无条件回收一次应用池，无论是否有活动。这对于需要长时间稳定运行的消息消费者来说是致命的——至少每天会被打断一次。

### 3. 进程外托管（Out-of-Process）的额外风险

项目未设置 `AspNetCoreHostingModel`，默认为**进程外（OutOfProcess）**：

- IIS 的 `w3wp.exe` 启动一个独立的 Kestrel 进程（`dotnet.exe` / `ThBimEngine.Web.exe`）
- IIS 通过 `AspNetCoreModuleV2` 将 HTTP 请求反向代理到 Kestrel
- 在这种模式下，IIS 对 Kestrel 进程的生命周期管理更加"积极"——因为它认为这是一个可替换的后端进程

### 4. 没有 Application Initialization / Preload

`AlwaysRunning` 启动应用池后，还需要配合：
- **`preloadEnabled="true"`**（Site 级别）
- **`applicationInitialization`** 配置（Application Pool 或 web.config）

这些配置告诉 IIS 在应用池启动后立即发送一个"预热"HTTP 请求到应用。没有这个预热请求，CAP 的 `ICapSubscribe` 消费者可能不会在首次 HTTP 请求之前完成初始化，导致进程处于"半就绪"状态。

### 5. 内存压力（次要因素）

- 物理内存持续增长可能触发 IIS 的 **Private Memory Limit** 回收阈值
- 默认情况下这个限制可能未被配置，但在某些服务器管理策略中会被设置

### 6. 异常崩溃（次要因素）

CAP 订阅器中的未处理异常可能导致进程崩溃，IIS 检测到进程退出后会重新启动（但此时消息消费已中断，且重新初始化需要时间）。


## 解决方案

### 必须立即执行

1. **将 Idle Time-out 设为 0（禁用）**
   ```
   IIS 管理器 → 应用程序池 → 高级设置 → 空闲超时(分钟) = 0
   ```
   这从根源解决了"无 HTTP 请求被回收"的问题。

2. **禁用定期回收**
   ```
   IIS 管理器 → 应用程序池 → 高级设置 → 定期时间间隔(分钟) = 0
   ```

### 强烈建议

3. **启用 Application Initialization 模块**
	- 安装 IIS 模块：`Install-WindowsFeature Web-AppInit`
	- 应用池设置：`Start Mode = AlwaysRunning`
	- 站点设置：`preloadEnabled = true`
	- 在 publish 时确保 `web.config` 包含：
 ```xml
 <system.webServer>
   <applicationInitialization doAppInitAfterRestart="true">
	 <add initializationPage="/api/health" />
   </applicationInitialization>
 </system.webServer>
 ```

4. **添加健康检查端点**（需要代码改动）

在 `Program.cs` 中添加：
```csharp
builder.Services.AddHealthCheures();
// ...
app.MapHealthChecks("/api/health");
```
这个端点有两个作用：
- 供 Application Initialization 模块做预热请求
- 供负载均衡器/监控系统做存活检测


5. **添加自我 ping 的保活机制**（推荐）

在应用中添加一个简单的定时器或后台服务，定期向自己的 HTTP 端点发请求：
```csharp
// 在 Program.cs 或一个 BackgroundService 中
// 每 10 分钟向自身发送一个 HTTP 请求
```

或者：注册一个 `IHostedService`，在应用启动后定时执行`HttpClient.GetAsync("http://localhost/...")` 作为心跳。

6. **考虑将消息消费独立为 Windows Service**

这是架构层面的改进——将 CAP 消费者从 IIS 进程中分离出来，部署为 Windows Service：
- Windows Service 不受 IIS 应用池生命周期管理
- 不会因为 HTTP 空闲超时被回收

当前使用的是 `WebApplication.CreateBuilder`，可以通过 `builder.Services.AddWindowsService()` 同时支持 IIS 托管和 Windows Service 模式。

### 即时排查

7. **检查 Windows 事件日志**

```
事件查看器 → Windows 日志 → 系统
筛选来源: WAS (Windows Process Activation Service)
查找 Event ID: 5079, 5117, 5186, 5187, 5189, 5190
```

这些事件会明确告诉你回收的**触发原因**（空闲超时、定期回收、内存限制等），从而确认主要根因。

## 验证方法

1. 在 IIS 管理器中关闭 Idle Timeout（设为 0）和 Regular Time Interval（设为 0）
2. 重新启动应用池
3. 检查事件日志，确认 WAS 不再触发 5xxx 回收事件
4. 观察 24 小时，确认未暴露地址的实例不再被回收
5. 如果仍有回收，检查 IIS 日志和事件日志中的**具体回收原因代码**：
	- `0` = 手动回收
	- `1` = 配置变更
	- `2` = 配置隔离变更
	- `3` = 进程内存限制
	- `4` = 进程无响应（ping 失败）
	- `5` = 进程请求限制
	- `6` = 进程请求超时
	- `7` = 计划回收
	- `8` = 空闲超时 ← **最可能触发**
	- `9` = 虚拟内存限制
