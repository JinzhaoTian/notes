Pi Agent 是一个可自由定制的 AI 编程智能体框架，追求极简设计，但赋予了开发者极高的灵活度，能将其改造成任何形态。

![](_imgs/Pasted%20image%2020260721161521.png)

Pi Agent（`@earendil-works/pi-coding-agent`）遵循极简主义（Minimalist）设计理念，这是其区别于其他 Agent 框架的核心特征：
1. **原子工具集**：只提供 4 个原子工具
	- `read`：读取文件内容
	- `write`：覆盖/创建文件
	- `edit`：基于字符串匹配的精确修改（手术刀式编辑）
	- `bash`：执行任意 Shell 命令
2. **System Prompt**：<1000 Token
3. **安全哲学**：**放弃无效的安全剧场（Security Theater）**，直接采用完全信任模式（Full Trust / YOLO模式）：
	- 没有权限拦截
	- 没有命令预检查
	- 完整的网络和文件系统访问权限
> [!tip] 理由
> 一旦允许 Agent 写代码并运行代码，它完全可以写 Python 脚本绕过任何文件系统沙箱。与其做无用防御，不如让开发者在隔离环境（容器/虚拟机）中运行 Pi

4. **进程管理**：Pi 不尝试在后台管理长时进程，而是直接使用 `tmux` ：
	- Agent 如需运行开发服务器或调试器，会在 `tmux` 会话中启动
	- 用户可随时 `attach` 到会话查看日志、接管调试
	- 这是最高级的可观测性



## 分层架构

Pi 的源码组织如下：
```
pi
├── packages/
│	├── ai/                # 统一模型、消息与 Provider API
│	├── agent/             # Agent 状态、Agent Loop、工具调度与事件
│	├── coding-agent/      # CLI、AgentSession、工具、会话、扩展与运行模式
│	└── tui/               # 终端输入、组件和差量渲染
├── scripts/
│   └── ...
├── package-lock.json
├── package.json
└── README.md
```

Pi 同时要处理模型协议、Agent Loop、终端显示和编码工具，与 Pi Agent 执行直接相关的代码主要在 `packages/` 下，依赖关系是：
```
pi-coding-agent ──→ pi-agent-core ──→ pi-ai     pi-tui
 │                                      ↑         ↑ 
 ├──────────────────────────────────────┘         │
 └────────────────────────────────────────────────┘ 
```







### 沙箱机制

Pi 最核心的安全特性是其沙箱机制（Cell Isolation）机制，设计目标是安全地执行不可信代码：
1. **隔离执行**：每个任务在独立的临时进程中运行，进程级别隔离
2. **权限最小化**：通过 Linux 命名空间（Namespace）和 Capabilities 机制限制资源访问
3. **环境快照与恢复**：执行前快照环境，执行后清理残留
4. **超时与资源限制**：可配置 `TOOL_EXEC_TIMEOUT_MS`、`TOOL_MAX_STDOUT_KB` 等参数
5. **网络访问控制**：默认阻断网络，仅允许显式白名单的主机

对于 Python 脚本执行，Pi 会在独立的虚拟环境中运行，避免污染主环境。


## 安装

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```


### 配置




## 扩展

Pi 原生只提供 4 个基础工具，其他所有能力都通过扩展系统来添加。Pi 的扩展生态很丰富，从增强核心能力到改善用户体验的各类工具都有。
### 核心能力

| 扩展名称                       | 功能描述                                                  | 安装方式                                    |
| -------------------------- | ----------------------------------------------------- | --------------------------------------- |
| **pi-mcp-adapter**         | 以懒加载方式接入 MCP (Model Context Protocol) 服务器，连接外部工具和数据源。 | `pi install npm:pi-mcp-adapter`         |
| **pi-web-access**          | 支持多个提供商的搜索聚合，让 Pi 具备实时联网搜索能力。                         | `pi install npm:pi-web-access`          |
| **pi-subagents**           | 为 Pi 增加子智能体支持，用于处理更复杂的任务分解。                           | `pi install npm:pi-subagents`           |
| **pi-agent-browser**       | 让 Pi 能控制一个无头浏览器，进行自动化网页操作。                            | `pi install npm:pi-agent-browser`       |
| **pi-planning-with-files** | 基于文件的规划功能，帮助 Agent 进行结构化任务管理。                         | `pi install npm:pi-planning-with-files` |

### 交互与工具优化

|扩展名称|功能描述|安装方式|
|---|---|---|
|**pi-agent-extensions**|一个大型扩展合集，包含会话管理(`/sessions`)、代码审查(`review`)、任务列表(`todos`)等众多实用工具。|`pi install npm:pi-agent-extensions`|
|**pi-extensions (by tmustier)**|另一个扩展合集，提供终端文件浏览器(`/readfiles`)、会话标签状态(`tab-status`)等。|`pi install git:github.com/tmustier/pi-extensions`|
|**sigma**|更好的提问工具，支持数字键导航和长文本换行。|见下方“如何安装”|
|**delta**|为 Pi 增加持久化内存（键值存储、情节记忆、项目笔记）。|见下方“如何安装”|
|**epsilon**|SQLite 支持的任务管理，含子任务、优先级和状态。|见下方“如何安装”|
