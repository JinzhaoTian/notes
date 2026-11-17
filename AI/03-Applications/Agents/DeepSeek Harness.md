
> [!quote] 一切皆插件，插件皆可逆。

```
                  ┌─────────────────────────────┐
                  │           UI / API          │
                  │   Web / Headless / SDK ...  │
                  └──────────────┬──────────────┘
                                 │
                         session/event
                                 │
              ┌──────────────────▼──────────────────┐
              │           Cordis Kernel             │
              │  plugin tree / dependency / effects │
              └──────────────────┬──────────────────┘
                                  │
        ┌────────────────────────┼────────────────────────┐
        │                          │                        │
   ctx.agentLoop             ctx.sessions              ctx.llm
        │                        │                        │
 Agent / Turn / Step     Append-only Event Log       Model Adapters
        │                        │
        │                 deriveMessages()
        │                        │
        └───────────────► Model Context
                                 │
                          Model Response
                                 │
                          Tool Calls
                                 │
                  ┌──────────────▼──────────────┐
                  │           ctx.tools            │
                  │    Tool Execution Pipeline     │
                  ├─────────────────────────────┤
                  │    pre-execute                 │
                  │    permissions / sandbox       │
                  │    monotonic guards            │
                  │    execute                     │
                  │    timeout / retry / metrics   │
                  │    post-execute                │
                  │    result normalization        │
                  └──────────────┬──────────────┘
                                  │
        ┌───────────────────────┼────────────────────────┐
        │                         │                          │
       FS                       Shell                    Subagents
        │                         │                          │
 local/remote              local/sandbox            DSH / Claude / Codex
```

## 项目架构

### 1. 底层：[Cordis](Cordis.md) 插件框架

- **插件**：一个函数或 `Service` 子类。
- **上下文 `ctx`**：放服务的容器，比如 `ctx.tools`、`ctx.llm`、`ctx.sessions`。插件按 key 查找服务，不直接 import 具体实现。
- **`inject`**：声明依赖，依赖就绪后插件才启动。
- **类型化事件**：有 `emit`、`waterfall`、`parallel`、`serial`、`bail` 五种分发方式。
- **注册是可逆的副作用**：通过 `ctx.effect()` 注册，插件卸载时自动撤销，所以支持热重载。

模型适配器、工具注册表、会话日志、agent loop 全都是插件，都能通过配置替换。

### 2. 运行时组合：Profile → Bundle → Patch

**一个 DeepSeek Harness 应用**，在 Cordis 眼里**就是一张插件清单，Profile、Bundle、Patch 一块组成这张清单**，拼好之后交给 Cordis 的 Loader：Loader 把插件清单中的每一个条目 import 进来，并发地启动。某个插件要用的服务（Cordis 中的 *Service* 概念）还没出现，它就先等着。

#### Entry

> [!quote] Entry 是串联 Cordis 和 DeepSeek Harness 的节点。 

**entry** 是插件清单里的一个条目。
- `name` 是要 import 的模块名。
- `id` 是这一个 entry 的唯一编号，后面所有修改都靠 id 找到它。
```yaml
- id: agent-default-model                        # 行的唯一标识，patch 靠它定位
  name: '@deepseek-ai/dsh-agent-default-model'   # 要加载的插件包
  config:                                        # 传给插件的配置
    provider: deepseek-official
    model: deepseek-flash
  disabled: false                                # 可选，也可以写成 !!js 表达式
```

DeepSeek Harness 中最终插件清单的算法就是：**从空数组 `[]` 开始，把所有 patch 按顺序执行一遍。**

#### Patch

一个 patch 是对插件清单条目（entry）的一次修改，语义实现在 `applyEntryPatches`(`vendor/include/src/index.ts#L57`)。

一般保存在 patch 文件 `cordis.patch.yml` 中，该 YAML 文件**顶层必须是数组**，里面按顺序放着若干个 patch。代码里管它叫 patch list（见 `parsePatchList`）。


**写法**：
- `- insert: [...]`：追加新行
- `- id: X` 加上 `config: {...}`：把 X 这一行的 config **整块换掉**
- `- id: X` 加上 `disabled: true`：关掉 X 这一行
```yaml
# ① 插入新行：不写 id 就追加到根列表；写了 id，就插到那个 group 行里面
- insert:
    - id: my-plugin
      name: '@someone/dsh-my-plugin'

# ② 按 id 覆盖某一行的字段
- id: session-title
  config: { ... }

# ③ 按 id 禁用某一行
- id: llm-pi-ai
  disabled: true
```

> [!caution] 注意
> 
> - **覆盖是整个字段替换，不做深合并。** 例如 base 里 `session-title` 的配置有三个字段，你只想改其中一个，也必须把三个都写出来：
> ```yaml
> - id: session-title
>   config:
>     fallbackMaxWords: 8     # 想改的
>     fallbackMaxBytes: 40    # 不改的也要重写，否则 base 里的值就丢了
>     maxTitleBytes: 80
> ```
> - **写了 `name` 就是一个断言。** 包名对不上时，这条 patch 会被跳过并给出警告。
> - **找不到对应 id 的 patch 只警告、不报错。**
> - **后面的层可以修改前面层 `insert` 进来的行。**
> - **`!!js` 表达式只能出现在 `config` 和 `disabled` 里**，在运行时求值，例如 `disabled: !!js "!ctx.get('profileContext')"`。
> - **想要一个空的 patch 层，写 `[]`。** 空文件或只有注释的文件会导致启动失败。

#### Bundle

Bundle 是一个 npm 包，把**配置**（patch 文件）和**代码**（patch 里引用的插件包，作为它的依赖）一起打包分发。

1. 在 `package.json` 里指定 patch 文件，可以指定单个文件，也可以指定按顺序排列的文件列表。
```json
"dsh": {
  "bundle": {
    "patch": [
      "./cordis.patch.yml",
      "./presets/standard.patch.yml",
      "./presets/ptc.patch.yml",
      "./presets/minimal.patch.yml",
      "./presets/cordis.patch.yml"
    ]
  }
},
```
2. 在 `package.json` 的 `dependencies` 指定所使用的插件代码包。
3. 在 `cordis.patch.yml` 等 patch 文件里指明该 bundle 中所涉及的插件清单条目的配置、设置、依赖关系。


**注意**：

- 只有直接列在 Profile 里的 Bundle 才会生效，不会传递地带入它依赖的其他 Bundle。
- 某个 Bundle 加载失败时，只会跳过它并报告，不会让整个启动失败。

##### 内置 Bundle

在 DeepSeek Harness 里内置了一些 Bundle 在 `packages/bundle/` 下：

1. **`dsh-base`**：共享核心，配置了包括 LLM、会话、凭据、存储、工具等
	- **`base/cordis.patch.yml`**：整个文件只有**一个 `insert`**，里面有
		- 模型（`llm`、`llm-deepseek`）、
		- 会话（`session`、`session-persistence-jsonl`）、
		- 工具（`tool-bash`、`tool-fs`…）、
		- 沙箱和审批、
		- `agent-loop`、
		- `hmr` 
2. **`dsh-web-app`**：叠加在 `dsh-base` 上的浏览器应用
	- **`web-app/cordis.patch.yml`**：
	    - `insert` Web 相关的条目（`webserver`、`web-runtime`、`connection`、各种 `ui-*`）。
	    - 调整 `dsh-base` 里属于单个 agent 的工具行关掉，比如 `tool-bash`。这些改由 agent preset 来挂。
	    - 补充 `presets/standard.patch.yml` 等 4 个 preset 文件。
3. **`dsh-headless`**：一次性的命令行任务，不用启动服务器
	- **`headless/cordis.patch.yml`**：
	    - 调整 `dsh-base` 里 `system-prompt` 的 config。
	    - 插入 `headless-startup` 解析命令行，`headless-runner` 跑完一个任务就退出。
	    - 关掉 `hmr`、`credentials`、`authorization`、`deepseek-account`。
4. **`dsh-sdk-app` / `dsh-acp-app`**：SDK JSON-RPC 服务器 / ACP 服务器
5. **`dsh-sdk-minimal`**：例外：独立提供一棵完整的配置树，不依赖 `dsh-base` 


##### 可选 Bundle

实验功能大多以**可选 Bundle** 的形式随安装附带，默认关闭，可以在插件管理器里打开（清单见 `packages/boot/app-boot/src/profile.ts#L223`）。



##### 插件 Bundle

第三方插件也可以做成 Bundle，用 `dsh plugin --profile <name> add <package>` 安装。



#### Profile

Profile 是一个应用组合，放在 `$DSH_HOME/profiles/<name>/` 这个目录下，`$DSH_HOME` 默认是 `~/.dsh`。

**目录结构**：
```
~/.dsh/profiles/<name>/
├── package.json
├── cordis.yml 
├── cordis.patch.yml
├── pnpm-workspace.yaml
└── compatibility.json   # 可选，记录插件版本豁免
```

**组成**：
- **`<name>`**：`web`、`headless`，或者你自己起的名字。
- **`package.json`**：可以在 `package.json` 里通过 `dsh.profile.bundles` 指定这个应用组合叠哪些 bundle、按什么顺序叠
- **`cordis.yml`**：默认为空 `[]`，**插件配置树的起点**
- **`cordis.patch.yml`**：这个 profile 自己的 patch 层，初始内容是 `[]`。可以保存自己的配置，比如账号的配置，UI 的设置。
- **`pnpm-workspace.yaml`**：pnpm 的设置，让第三方插件和安装共用同一个 cordis 实例
- **`compatibility.json`**：可选，记录插件版本豁免

> [!question] 为什么 Profile 是在一个目录里，而不是一个配置文件？
> DeepSeek Harness 可以通过 `dsh plugin --profile web add xxx` 安装插件包，此命令就是在这个目录里执行 pnpm，因此需要有一个能装插件包的目录空间，用来放置 `package.json` 加 `node_modules/`，所以每个 profile 在磁盘上都是一个私有的 npm 项目。

> [!question] 为什么 `cordis.yml` 是空的 `[]` ？为什么还要存在？
> 1. **Cordis 设计**：
> 	- 在 Cordis 的设计里，**配置文件就是插件树的存档**。程序运行时你对树做的改动（加一个插件、改一个插件的配置、某个插件把自己关掉），都会写回这个文件，下次启动时按文件原样恢复。
> 2. **DeepSeek Harness 机制**：
> 	- dsh 约定（`vendor/include/src/index.ts:181`）把文件所在的目录当作解析相对路径的起点，这样相对路径就都从 profile 目录算起。`--dump-config` 也用同一个文件当起点，保证打印出来的清单和真正启动时拼出来的一致。
> 	- dsh 约定 ** `cordis.yml` 必须保持 `[]`，启动器每次启动都会把这个文件重写成 `[]`（`apps/cli/src/profile-boot.ts:198`）。**  Include 在某些情况下会把当前的行树写回这个文件。比如某个插件自己退出了，Loader 会把这一行标成 `disabled: true`，然后触发写回（`loader/src/index.ts:164`）。写进去的是已经拼好的全部行。如果不清掉它，下次启动时，文件里的这些行会和 patch 再插入一次的行重复。所以启动器每次启动都先把它覆盖成 `[]`（`profile-boot.ts:179` 的注释专门解释了这一点）。


##### 内置 Profile

内置模板写死在 `packages/boot/app-boot/src/profile.ts#L179`：

| Profile       | Bundles                     |
| ------------- | --------------------------- |
| `web`         | `dsh-base` + `dsh-web-app`  |
| `headless`    | `dsh-base` + `dsh-headless` |
| `sdk`         | `dsh-base` + `dsh-sdk-app`  |
| `acp`         | `dsh-base` + `dsh-acp-app`  |
| `sdk-minimal` | `dsh-sdk-minimal`           |

`desktop` 使用保留的 `desktop` profile，它的 bundles 是 `dsh-base` + `dsh-web-app`，和 `web` 一样。


**运行机制**：

- 第一次运行 `dsh --profile web` 时，会**按模板自动初始化这个 Profile 目录**。之后再运行，已有文件一律不覆盖。
- `dsh --profile my --from-default-profile web` 会从模板复制出一个独立的新 profile。它不记录继承关系，以后模板更新不会影响它；对已有的 `my` 再执行一次，会把它重置。


#### 叠加顺序

以 DeepSeek Harness 内置的 `web` Profile 为例，其 bundles 是 `dsh-base` + `dsh-web-app`，在 `packages/boot/app-boot/src/profile-context.ts#L74-L80` 中把所有层按下面的顺序拼成一个列表，**后面的覆盖前面的**：

```
[ ] ~/.dsh/profiles/web/cordis.yml         → Profile 目录下，每次启动都置为 []
 +  base/cordis.patch.yml                  → Bundles：dsh-base
 +  web-app/cordis.patch.yml               →          dsh-web-app
 +  ~/.dsh/profiles/web/cordis.patch.yml   → Profile 目录下
 +  ~/.dsh/cordis.patch.yml                → 对所有 profile 的修改
 +  --patch a.yml --patch b.yml            → 命令行临时叠加，只对这次启动生效
```

 `dsh-web-app` 为例：

- `packages/bundle/base/cordis.patch.yml` 插入的 `system-prompt` 行里 `personaPrefix: ''`，
```yaml
- insert:
    
	# ...
    
    # The deployment persona is a deployment choice; plan-mode and tool plugins own
    # their own prompt sections.
    - id: system-prompt
      name: '@deepseek-ai/dsh-system-prompt'
      config:
        personaPrefix: ''
    
	# ...
```

- `packages/bundle/web-app/cordis.patch.yml` 用 `- id: system-prompt` 把它覆盖成 `"You are a coding agent powered by the {{model}} model."`，然后再插入 Web 专属的行。
```yaml
- id: system-prompt
  config:
    personaPrefix: >-
      You are a coding agent powered by the {{model}} model.
      
- insert:
    
	# ...
    
```

两个实用操作：

- **查看最终结果**：`dsh --profile web --dump-config`。它和真正启动走的是同一个 `applyEntryPatches`，打印出来的就是实际加载的内容。
- **修改何时生效**：`web` profile 默认开启热重载（HMR），修改 profile 的 `package.json` 或两份用户 patch 会实时重新组合；`headless`、`sdk`、`acp` 默认关闭 HMR，改完需要重启。



### 3. 核心：Agent 循环 + 会话日志

核心循环是 `agent.ts`（`packages/core/agent-loop/src/agent.ts`）里的 `ReactLoopAgent`。
- 一个 `step` 是一次模型请求加上它调用的工具
- 一个 `turn` 包含 0 到 n 个 step

```
turn/start
  ┌ step 循环 ──────────────────────────────────────────
  │ agent/pre-step        插件可改写或拒绝本步输入（waterfall）
  │ 组装 system prompt + 工具 schema
  │ agent/request → prepareCall   确定模型路由
  │ 写日志：system/message、user/message
  │ ctx.llm.stream() → assistant/message
  │ tool/call → tools/pre-execute → execute → post-execute → tool/result
  └ 还有待处理的工具结果或新输入 → 进入下一个 step
agent/turn-stopping       插件可在这里注入输入，让本轮继续
turn/end
```

对应代码在 `turn()`（`packages/core/agent-loop/src/agent.ts#L298`）和 `step()`（`packages/core/agent-loop/src/agent.ts#L400`）。

整个设计围绕一条不变量：**模型能看到的内容，一定写进了日志**。发给模型的每个请求都必须能从只追加的 session 日志重建出来，所以 fork、resume、转录和遥测都从日志推导。

会话存成带版本号的 JSONL 文件（`session.vN.jsonl`）。

升级时由迁移包逐级转换（`packages/session/` 下的 `v0-to-v1` 到 `v3-to-v4`），已有的旧版本文件不会被覆盖或删除。

### 4. 能力接缝（Capability seam）

每种能力都分成三个角色：Service Definition（接口）、Service Provider（实现）、Consumer（通常是给模型用的工具）。换一个 provider，整个产品的行为就跟着变。比如把文件系统和子进程的 provider 指向远端，Bash、PTY 和 LSP 会一起切过去。

| 领域    | 内容                                                                                                              |
| ----- | --------------------------------------------------------------------------------------------------------------- |
| 模型    | DeepSeek 官方（API key 或账号登录）；`pi-ai` 多提供方（默认不注册任何路由，配置后才生效）；请求重试；token 计量                                         |
| 执行    | bash/pwsh、持久终端（PTY）、SSH、后台 jobs；PTC 模式（`run_code`：模型写一段 TS 程序，实验性支持 Python，把宿主工具当 async 函数调用，每次子调用仍然走完整的工具守卫流程） |
| 沙箱    | Linux 用 bwrap/Landlock，macOS 用 Seatbelt，Windows 用受限令牌加 ACL                                                      |
| 文件/代码 | read/write/edit、glob/grep（自带 ripgrep）、LSP                                                                       |
| 协作    | subagent：可以在进程内新建或 fork 子 agent，也可以把任务委派给真正的 Codex 或 Claude Code；另有 todo、plan 模式、goal、定时任务                      |
| 扩展    | MCP、skill、上下文压缩；插件管理器（模型在获得批准后可以安装或启用插件）                                                                        |

### 5. 交付形态

- **CLI**：`/apps/cli/` 里的 `dsh` 是唯一受支持的启动入口，运行 `npx @deepseek-ai/dsh web` 即可。
- **Web UI**：分 Host（Node）和 Client（浏览器 React）两半。Client 也是插件化的，`packages/client/` 下约 66 个包，大多是 `ui-*` 插件。两端通过 Typert 通信：它从 TS 类型自动生成 schema 和类型化 RPC，不用手写协议。
- **Desktop**：基于 Electron，内置完整运行时，复用整套 Web 应用。
- **SDK**：TypeScript 的 JSON-RPC SDK，以及 Python SDK。Python SDK 打包的是同一个 `dsh`，启动时用 `--profile sdk`。
- **ACP**：只面向自动化场景的 Agent Client Protocol 服务器。

`tsconfig.base.client.json` 是 Client 端的编译配置（浏览器 lib、React JSX、不带 Node 类型），和 `tsconfig.host.json` 一起，把 Host 和 Client 分成两套独立的 TypeScript 编译配置。

### 6. 工程体系

- **规模**：约 340 个包，`src` 下约 44 万行 TS。2026-06-10 开始，到现在约 2.2 万次提交（含合并提交）。
- **门禁很严**：
    - TS 开 `strict`；
    - `packages/*/*/src` 要求逐文件 100% 覆盖率；
    - 有 76 个 `verify-*` 检查脚本，比如导出必须写 JSDoc、UI 文案必须走 i18n、启动入口有白名单；
    - 不允许新增 `as unknown`。
- **快照测试**：录下来的真实会话可以在没有 API key 的情况下回放，覆盖各个内置 profile。
- **文档**：中英双语成对维护。工具目录、配置目录、模块依赖图都由脚本生成，并检查是否过期。
- **由 agent 参与开发**：`.agents/` 下有约 2500 篇决策记录和一组开发用 skill，`AGENTS.md` 也写得非常细。看得出这个仓库本身有大量 coding agent 参与开发。
- **实验包**：`packages/experimental/` 下有 47 个，包括 Agent Teams、浏览器和桌面操控（Playwright/Stagehand/CUA）、直接复用 Claude Code 的 `hooks.json`、语音输入、GitHub webhook、worktree 等。


## 启动流程

DeepSeek Harness 是一个基于 Cordis 插件运行时、完全由插件拼成的应用，再加上一个按照 profile 把这些插件组装起来的启动器。


### 总览

```
await runCli()                                // bin.ts:77
├── parseDshArgs()                            // bin.ts:29  ← 解析 --profile、--patch 等启动器参数
└── runProfile()                              // bin.ts:35  ← 仅 profile 模式进入
    ├── composeProfile()                      // profile-boot.ts:292  ← 加载 profile，cordis.yml 重写为 []
    ├── readProfilePatches()                  // profile-boot.ts:323  ← 按层收集 patch，作为 boot 的参数
    ├── boot(NAME, cordis.yml, patches, prepare)    // profile-boot.ts:323
    │   ├── new Context()                     // index.ts:1204  ← 根 ctx + Fiber#0
    │   ├── ctx.plugin(Loader)                // index.ts:1226  ← Loader
    │   ├── prepare(ctx)                      // index.ts:1227  ← 启动器往根上挂服务
    │   ├── mountRootInclude()                // index.ts:1229  ← Include
    │   │   └── Include.init → root.update    // include/index.ts:262  ← cordis.yml 的每一行 → 一个 Fiber
    │   ├── loader.await()                    // index.ts:1235  ← 等整棵树稳定
    │   └── auditStartupEntries()             // index.ts:1237  ← 必需行失败 → StartupError
    └── appReady.commit()                     // profile-boot.ts:344  ← 通知各 app 开始工作
```

### 逐行

1. **`boot(NAME, cordis.yml, patches, prepare)`**
	- **`const ctx = new Context()`**（`packages/boot/app-boot/src/index.ts:1204`）：
	    - **功能**：创建一个 **Cordis 框架依赖注入系统的核心代理对象 [`Context`](Cordis.md#`Context`)**。
	    - **调用**：`vendor/cordis/src/context.ts:71`
    - **`const diagnostics = new Context()`** 
	    - **功能**：建立第二个独立的代理对象 `Context`，并且和 `ctx: Context` 共用一个 `logger`，只用来收集启动期间的警告和错误日志，当 `ctx: Context` 在 `catch` 里被销毁时，清理过程中的报错仍然收得进来，会在 `finally` 里销毁（`packages/boot/app-boot/src/index.ts:1264`）。
	- **`ctx.baseUrl = <config path>`**（`packages/boot/app-boot/src/index.ts:1219`）：
	    - **功能**：设置 `ctx` 上的 `baseUrl`，通过 `set` trap 直接放行（`reflect.ts:180-181`），后续在插件 `ctx` 上做同样的赋值会抛错。Loader 构造时会把它复制进自己的行树（`tree.ts:17`），所以必须在 `plugin(Loader)` 之前设好。
	- **`ctx.provide('dshHomePath', …)`**（:1220）→ `reflect.ts:277`
	    - 功能：作为根 Fiber 的 effect 登记，服务实现写进全局 store 和根 Fiber 的 `store`（:292-293）。
	    - 读一个没有 inject 的服务时，会沿 Fiber 链逐级查 `store`，一直查到根（`reflect.ts:153-166`）。所以根上的服务所有插件都读得到。
	- **`ctx.on('internal/update', …, { global, prepend })`**（:1223）
	    - `Fiber.update()` 把重启交给 `internal/update` 瀑布，但丢弃了返回的 promise（`fiber.ts:777-781`）。
	    - 这个全局、前置的钩子给 `next()` 接上 `.catch`，把插件重启失败记进日志。
	- **`await ctx.plugin(Loader)`**（:1226）→ `registry.ts:330`，创建 Loader#1
	    - Loader 没有 inject，Fiber 构造时 `_refresh()` 直接转为 LOADING（`fiber.ts:318`）。
	    - `_reload` 等一个微任务后执行 `new Loader(ctx)`（`fiber.ts:253`）。构造函数依次：
	        - `provide('loader')`（loader/index.ts:102）；
	        - 注册 `internal/config`、两个 `internal/update`、`internal/plugin` 钩子（:104-166）；
	        - `ctx.plugin(isolate)`，生成 isolate#2（:168）。
	    - 这里的 `await` 实际等的是 `fiber.await()`（`registry.ts:332-334`），也就是等 Loader#1 自己进入 ACTIVE。之后 `ctx.loader` 就能用了。
	- **`await prepare?.(ctx)`**（:1227）→ `profile-boot.ts:323-339`  
	    回调拿到的就是根 ctx，此时 Loader 已经就位。按顺序做三件事：
	    
	    - provide `profileContext` 和启动环境快照（`DSH_LAUNCH_ENVIRONMENT_KEY`）（:325、:328）；
	    - `await plugin(PluginPackages, { resolution })`（:329）：提供 `pluginPackages` 服务，并按 profile 的包表接管 Node 的 ESM/CJS 解析；
	    - `provideCmdline`（:334）：提供 `cmdlineArgs`、`appExit`、`appReady`。
	    
	    之后 `stage` 切换为 `plugin tree failed to load`（:1228）。在这之前的失败都标为 `host preparation failed`。
    
	- **`await mountRootInclude(…)`**（:1229 → :738）
	    
	    - 登记两个 builtin：`include`（:745）和 `group`（:763）。
	    - `prepareProfilePatches`（:769）：在空列表上套完全部 patch 层，做兼容性预检，再包成一个 `{ insert: [...] }`（`compatibility-preflight.ts:180-187`）。
	    - 执行 `ctx.loader.create({ id: 'include', name: 'cordis:include', config })`（:774-779），展开见第二、三节。
	    
	    `create` 在 Include 的 Fiber 建好后就返回，这时它还在 LOADING。随后把这一行记为 bootstrap include，供审计使用（:780-783）。
	    
	- **`await ctx.get('loader')?.await()`**（:1235）→ `tree.ts:44`
	    - 反复收集整棵树（含 Include 子树）每一行的 `_initTask`（正在 import）和 `fiber.inertia`（正在加载或卸载），直到两者都为空。
	    - 需要这一步，是因为前面的 await 都不覆盖整棵树：
	        - `ctx.plugin()` 只等这个 Fiber 自己的状态转换，停在 PENDING 的 Fiber 会立即返回；
	        - `loader.create()` 只等 import 完成和 Fiber 创建。
	    - :1236 再检查一次 loader：如果启动途中已有界面把整棵树 dispose 了，就直接返回。
	- **`await auditStartupEntries(ctx, binName)`**（:1237 → :1150）
	    - `inactiveEntries`（:1039）遍历 `ctx.loader.entries()`，跳过 disabled 的行，收集 import 失败、FAILED、PENDING 等没有进入 ACTIVE 的行。
	    - 如果其中有 bootstrap include 或必需行（`requiredStartupEntryIds`，:946），就抛 `StartupError`；其余只发警告。
	    - 失败路径：`catch` 先 dispose 根 Fiber。如果是 `StartupError`，附上启动日志后抛出；其他错误用 `stage` 标签包装后抛出（:1239-1259）。
- `boot()` 返回后，`runProfile` 调用 `appReady.commit()`（`profile-boot.ts:344`），通知各个 app：启动成功，可以开始工作了。

#### Loader 怎么把 `include` 这一行变成运行中的插件

Fiber 在构造时会同步发出 `internal/plugin` 事件。我在这个事件上抓调用栈，就能看到是谁创建了这个 Fiber：
```
[internal/plugin] new Fiber for Include#3, created from:
    at new Fiber (vendor/cordis/src/fiber.ts:302:22)
    at Proxy.plugin (vendor/cordis/src/registry.ts:330:19)
    at Entry._init (vendor/loader/src/config/entry.ts:236:36)
    at async Entry.init (vendor/loader/src/config/entry.ts:212:7)
    at async Entry.update (vendor/loader/src/config/entry.ts:155:7)
    at async EntryGroup.create (vendor/loader/src/config/group.ts:27:5)
```

再往上几层没有出现在异步调用栈里，因为 `EntryTree.create` 是直接 `return group.create(...)`，没有 await。从源码补全是：`mountRootInclude` → `EntryTree.create`（`tree.ts:82`）→ `EntryGroup.create`。

每一层做的事：

- **`EntryTree.create`**：把这一行加进 Loader 的清单，然后交给 `EntryGroup.create`。
- **`EntryGroup.create`**（`group.ts:20`）：执行 `new Entry(loader)`。Entry 的 ctx 是 `loader.ctx.extend(...)` 得到的视图，上面多挂了一个"我是哪一行"的标记（`entry.ts:58`）。
- **`Entry.update` → `init` → `_init`**（`entry.ts:223-238`）：
    - `tree.import('cordis:include')`：名字以 `cordis:` 开头，所以去内置表里取出 Include 类（`tree.ts:113`）；
    - `_patchContext`：把 Entry 的 ctx 的原型接到它所在组的 ctx 上，这样 `ctx.fiber` 就指向 Loader#1；
    - `ctx.registry.plugin(Include, config)`：从这里回到 Cordis 核心。
- **`new Fiber`**（`fiber.ts:222`）：Include#3 诞生，它的父 Fiber 是 Loader#1。
    - 构造过程中发出 `internal/plugin`，Loader 的钩子据此把 `fiber.entry` 设为这一行。
    - Include 声明了 `inject = ['loader']`，而 loader 已经就绪，所以它马上进入 LOADING。

#### Include 怎么把 `cordis.yml` 里的每一行挂起来

Include#3 进入 LOADING 后，Fiber 状态机先执行 `new Include(ctx, config)`，再调用它的 `async* [Service.init]()`（`include/src/index.ts:245`）。这个方法按顺序做三件事：

1. 读 `cordis.yml`；
2. `yield () => this.stop()`，把清理函数登记上；
3. `await this.root.update(套完 patch 的行)`。

每一行都会再走一遍第四节的流程。抓到的栈完整显示了这条链：

```
[internal/plugin] new Fiber for probe#4, created from:
    at new Fiber (vendor/cordis/src/fiber.ts:302:22)
    at Proxy.plugin (vendor/cordis/src/registry.ts:330:19)
    at Entry._init (vendor/loader/src/config/entry.ts:236:36)
    at async Entry.init (vendor/loader/src/config/entry.ts:212:7)
    at async Entry.update (vendor/loader/src/config/entry.ts:155:7)
    at async EntryGroup.create (vendor/loader/src/config/group.ts:27:5)
    at async <anonymous> (vendor/loader/src/config/group.ts:58:9)
    at async EntryGroup.update (vendor/loader/src/config/group.ts:56:5)
    at async Include.[cordis.init] (vendor/include/src/index.ts:262:5)
    at async <anonymous> (vendor/cordis/src/fiber.ts:391:28)
    at async Fiber._reload (vendor/cordis/src/fiber.ts:656:9)
```

从下往上读：

1. Include#3 的 `Fiber._reload` 执行它的 async generator；
2. generator 里调用 `root.update`；
3. `EntryGroup.update` 对每一行调用 `create`；
4. 又回到 `Entry`，经 `registry.plugin` 创建出新的 Fiber。

和 `include` 那一行相比，唯一的区别在 `tree.import(name)`：

- 名字不再以 `cordis:` 开头，所以改用 Node 内部的 ESM 加载器来 import（`tree.ts:122`）。
- 解析的基准是 `ctx.baseUrl`，Include 已经把它设成 `cordis.yml` 所在的目录。
- 结果是，插件包名从 profile 目录开始解析，而不是从 Loader 代码所在的位置。

#### 插件函数体真正执行的时刻

一行的 Fiber 创建好以后，插件函数并不会在 `create` 的调用栈里同步执行，而是交给状态机处理：

- 依赖还没齐，就停在 PENDING；
- 依赖齐了，`_reload` 先等一个微任务，再执行插件（`fiber.ts:646-673`）。

所以在插件函数体里抓到的栈，最上面只能到 `Fiber._reload`：

```
[probe] stack:
    at Object.apply [as callback] (…/probe.mjs:11:36)
    at Fiber.execute (vendor/cordis/src/fiber.ts:259:28)
    at <anonymous> (vendor/cordis/src/fiber.ts:366:45)
    at composeError (vendor/cordis/src/utils.ts:272:25)
    at Fiber._execute (vendor/cordis/src/fiber.ts:358:12)
    at Fiber._reload (vendor/cordis/src/fiber.ts:656:20)
```

测试配置里有两行：

- `probe` 排第一，依赖 `greeting`；
- `greeting-provider` 排第二，负责提供 `greeting`。

probe#4 创建后一直停在 PENDING，所以它创建时没有输出状态变化。下面是实际的状态输出，略去了穿插在中间的调用栈：

```
  [status] Include#3: PENDING -> LOADING
  [status] greeting-provider#5: PENDING -> LOADING
[greeting-provider] body runs, provide("greeting")
  [status] greeting-provider#5: LOADING -> ACTIVE
  [status] probe#4: PENDING -> LOADING
[probe] body runs; ctx.greeting.text = "hello from a sibling row"
  [status] probe#4: LOADING -> ACTIVE
  [status] Include#3: LOADING -> ACTIVE
```

也就是说，yml 里的顺序只决定谁先被创建，谁先执行要看依赖。

#### 启动完成时的 Fiber 树（实测）

```
root#0 ACTIVE  provides/sees=[dshHomePath,demoHostService]
    Loader#1 ACTIVE  provides/sees=[loader]
        isolate#2 ACTIVE  provides/sees=[]
        Include#3 ACTIVE  row="include"  inject=[loader]  provides/sees=[loader]
            probe#4 ACTIVE  row="include:probe"  inject=[greeting]  provides/sees=[greeting]
            greeting-provider#5 ACTIVE  row="include:greeting-provider"  provides/sees=[greeting]

loader entries (full ids): include, include:probe, include:greeting-provider
```

几点说明：

- `provides/sees` 是 `fiber.store` 的内容，包括它自己 provide 的服务，以及它 inject 进来的服务快照。
- `demoHostService` 是我在 `prepare` 里 provide 的，用来模拟启动器挂在根上的那些服务。
- probe 打印的祖先链与这棵树一致：`probe#4(row "include:probe") -> Include#3(row "include") -> Loader#1 -> root#0`。

真实的 `dsh` 启动只是把这棵树放大了：

- 根上多了启动器 provide 的那些服务，以及 `PluginPackages` 这个子插件；
- Include 下面挂着所有 bundle 和 patch 带来的行。

机制完全相同。




### 第 1 层：根 Context

- **Cordis**：`new Context()` 造出根，`ctx.provide()` 提供服务，`ctx.plugin()` 挂插件。这些是最基本的 API。
- **DSH**：决定根上直接挂什么，这些服务也全是 DSH 定义的：
    - `dshHomePath`（index.ts:1223）；
    - 启动器 `prepare` 回调里挂的 `profileContext`、环境快照、`PluginPackages`、`cmdlineArgs` / `appExit` / `appReady`（profile-boot.ts:323-338）。

“patch 改不到，`--dump-config` 也看不到”这句话，前后两半各属一边：

- patch 是 Cordis Include 的功能，只能改行；
- `--dump-config` 是 DSH 的命令，只打印合成出来的行。

根上这些东西是 DSH 在代码里直接挂的，不是行，所以两者都碰不到。

### 第 2 层：Loader

- **Cordis**：Loader 插件本身。
    - "行"和"行树"都是它的概念。它负责把行变成正在运行的插件。
    - 它自己的清单只放在内存里，`write()` 是空函数（loader/src/index.ts:171）。
    - 名字以 `cordis:` 开头的行，会去 `loader.builtins` 这张表里找模块（tree.ts:113）。这张表默认是空的，vendor 里没有任何代码往里登记。
- **DSH**：
    - 往 `builtins` 里登记了 `include` 和 `group`（index.ts:750），所以 `cordis:include` 才找得到东西。
    - Loader 的清单里只放了一行，id 固定写成 `include`（index.ts:779）。

### 第 3 层：Include

- **Cordis**：Include 插件本身，一棵由文件支撑的行树。
    - 它读一个 yml 或 json 文件得到行列表，套上 `config.patches`，再逐行启动（include/src/index.ts:245）。
    - patch 的规则也是 Cordis 的：`insert` 插入行，按 `id` 覆盖字段。
    - 按 Cordis 的原意，插件清单就写在这个文件里，运行时的改动也会写回去（见 include/README.md）。
- **DSH**：没按这个原意用。
    - `cordis.yml` 只是个空壳，每次启动都被重写成 `[]`（profile-boot.ts:198）。
    - 插件全部通过 patch 加进来。分几层、按什么顺序由 DSH 决定（profile-context.ts:74）：各 bundle → profile 的 `cordis.patch.yml` → `$DSH_HOME/cordis.patch.yml` → `--patch` → 遥测开关。bundle 和 profile 都是 DSH 的概念。
    - 交给 Include 之前，DSH 先用 Cordis 的 patch 算法 `applyEntryPatches`，在空列表上把所有层套完，做完兼容性检查，再包成一个 `{ insert: [全部行] }`（compatibility-preflight.ts:180）。Include 拿到的只有这一层。

### 完整 id：Cordis 的规则，加上 DSH 的挂法

- **Cordis 的规则**：完整 id = 启动这棵行树的那一行的 id + `:` + 自己的 id（entry.ts:67）。分组行不开新树，所以不会多一段。`ctx.loader.resolve('a:b')` 这种带冒号的查法也是 Cordis 的。
- **DSH 的挂法**：Cordis 文档里的标准用法是 `root.plugin(Include, { path })`，把 Include 直接挂在根上。那样 Include 不是由任何行启动的，`llm` 的完整 id 就是 `llm`，没有前缀。DSH 的做法是把 Include 作为 Loader 里 id 为 `include` 的一行挂进去，所以所有插件的完整 id 才多出 `include:`。
    - 这样挂的结果是，整棵树都在 Loader 遍历得到的范围里，`ctx.loader.entries()`、`resolve()`、`await()` 都能覆盖 Include 下面的行。`boot()` 正是靠 `ctx.loader.await()` 等整棵树起完的（index.ts:1238）。
- `entryId` 和 `patchId` 是 DSH 的 plugin-manager 起的字段名。前者是 Cordis 规则下的完整 id，后者是你在 patch 里写的短 id。






## 插件开发

> [!quote] 插件 = 一个「bundle」包

插件通过 `package.json` 里的 `dsh.bundle.patch` 指向一个 YAML patch，patch 往 Loader 的插件行列表里插入一行（或按 id 覆盖已有行）。你从不直接改配置文件，安装 bundle 就是改配置的唯一手段。

### 定位

两条完全不同的路径，选错会白做：

- **Host / 全局能力**（新工具、注册表、Service、HTTP 路由、Web UI 插件）→ 走本流程，写 bundle 安装到 profile。
- **Agent preset / 组合**（人格、给某类 agent 的 scoped 工具集）→ 走 `editing-cordis-compositions`，patch 里声明 `@deepseek-ai/dsh-agent-preset` 的 `config.plugins`。

判断依据：**被 Host 插件消费的 Service 必须放 Host 配置**；preset 里提供 Service 的插件必须把 provider 和所有 consumer 一起 `isolate`。纯 UI 装饰、Sidebar 页签、工具，都属于 Host bundle 里的 Client 半边。

### 发现

不要凭记忆猜方法名。顺序是：

1. `cordis_inspect_list` 拿 provider 清单；Host 有 `Service` / `Event` / `Config` / `Tool`，Client 有 `Service` / `Event` / `Builtin` / `Slots` / `Theme`。
2. 定向查询：先省略 `service`/`event` 拿目录，再传精确名字拿完整签名的 coding contract。
3. UI 必查 `Slots.listSubTree`（`root` 传精确 slot key）：拿到该 slot 的 `kind`、`purpose`、`replaceRisk`、`registration` 参数、`keyDomain` 和已占用 key。
4. 给已装插件的 row 写 `config` 前，用 `Config.listConfigs`（先 `name` 过滤，再用 `entry` id）读它的 JSON Schema，并跟进 `$defs`。

UI 落点选择原则：优先**已分配空间**的 slot（如 `conversation.composer.dock`、`sidebar.right.tab.*`），需要浮层才用 `shell.overlay`。`replaceRisk: shadows-shipped-ui` 的 slot 意味着会顶掉自带 UI——除非确实要替换，否则别选。

### 编写

1. **`package.json`**：最小 Host bundle 不需要依赖、构建或 install script：

```json
{
  "name": "@local/my-plugin",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "exports": { ".": "./index.js" },
  "dsh": { "bundle": { "patch": "./cordis.patch.yml" } }
}
```

有 UI 半边时增加 `dsh.client`（`platform` / `immediately` / `inject` / 可选 `external`）、`"./client"` export，以及显示元数据：`meta.title`/`meta.description`（多语言放 `locale/<lang>.json`）、顶层 `icon` 指向包内 SVG/PNG/JPEG/WebP（≤256 KiB，拒绝绝对路径、URL、越界路径、外指符号链接）。缺字段会回退到包名与 `description`。本仓库的 package.json 就是一个带 Client 半边的完整例子。

2. **`cordis.patch.yml`**：patch 方言的要点：
	- `insert: [rows]` 追加行；若 `id` 指向已有 `group: true` 行，则插入该组。
	- 只写 `id` 的 patch 会**覆盖目标行**，且 `config` 是**整体替换而非深合并** —— 必须把该行需要的每个字段重新写全。
	- 无 `insert` 且无有效 `id`、或匹配不到目标行的 patch，会被警告并跳过。
	- 行字段：`id`、`name`、`config`、`disabled`、`inject`、`intercept`、`isolate`。
	- `!!js` 是 Loader 表达式（不是 `!js`）：只在 `config` 内或 `disabled` 中使用，在声明注入激活后按该插件 context 求值，可用 `ctx.<service>`。

3. **`index.js`**：（Host 半边）二选一，不要混用：
	- `export function apply(ctx, config) {}`，可选 `export const inject = [...]` 与 `export const Config`；
	- 或 default export 一个 service class。

**所有资源都在 `apply` 内用 `ctx.effect` / `ctx.on` 注册并返回清理函数** —— 这是插件能被正确卸载/热更的前提。本仓库 index.js 里 `ctx.effect(() => ctx.webServer.register({...}))` 就是标准写法。

4. **`client.js`**：（浏览器半边，可选）产物是 `window.__ModuleLoader__.load({ id, factory })`，`id` 必须等于包名；React 从浏览器模块表 `require('react')` 取，**不要**自带 React、CDN 或 UMD。factory 内保持无副作用，用 `ctx.slots.inject(slot, () => ctx.slots.register(options, Component))` 注册。样式只用主题 token（本仓库用的是 `--dsw-alias-*`），可见文案走 Client locale service；不要替换 app root 或往 `document.body` 追加第二份应用，也不要读别的插件的 DOM/CSS 来「估算位置」。

### 安装

`plugin_manager` `action: install_bundle`，`target` = **bundle 的绝对包目录**。它自己完成装包与 bundle 选中，**不要**用 shell 复刻这些步骤。若返回 `pendingBuilds`，只有在用户明确同意后才能传 `approvedBuilds`。安装会在 Host 进程执行插件代码，因此需要 Full access 或审批。
