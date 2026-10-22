DeepSeek Harness 的架构设计思想：**一切皆插件，插件皆可逆**。

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
        │                        │                        │
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
                  │        ctx.tools            │
                  │ Tool Execution Pipeline     │
                  ├─────────────────────────────┤
                  │ pre-execute                 │
                  │ permissions / sandbox       │
                  │ monotonic guards            │
                  │ execute                     │
                  │ timeout / retry / metrics   │
                  │ post-execute                │
                  │ result normalization        │
                  └──────────────┬──────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
       FS                     Shell                   Subagents
        │                        │                        │
 local/remote             local/sandbox       DSH / Claude / Codex
```

## 项目架构

### 1. 底层：Cordis 插件框架

- **插件**：一个函数或 `Service` 子类。
- **上下文 `ctx`**：放服务的容器，比如 `ctx.tools`、`ctx.llm`、`ctx.sessions`。插件按 key 查找服务，不直接 import 具体实现。
- **`inject`**：声明依赖，依赖就绪后插件才启动。
- **类型化事件**：有 `emit`、`waterfall`、`parallel`、`serial`、`bail` 五种分发方式。
- **注册是可逆的副作用**：通过 `ctx.effect()` 注册，插件卸载时自动撤销，所以支持热重载。

模型适配器、工具注册表、会话日志、agent loop 全都是插件，都能通过配置替换。

### 2. 运行时组合：Profile → Bundle → Patch

**一个 dsh 应用**，在 Cordis 眼里**就是一张插件清单，Profile、Bundle、Patch 一块组成这张清单**，拼好之后交给 Cordis 的 Loader：Loader 把插件清单中的每一个条目 import 进来，并发地启动。某个插件要用的服务（Cordis 中的 *Service* 概念）还没出现，它就先等着。

- **entry**：插件清单里的一个条目
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

- **Patch**：一个 YAML 数组，每一项是对清单条目的一次修改，只有三种：
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

- **Bundle**：一个普通的 npm 包
	- 首先，`package.json` 里使用 `"dsh": { "bundle": { "patch": "./cordis.patch.yml" } }` 指向 patch 文件；在 dependencies 指定插件包
	- 其次，`cordis.patch.yml` 里指明每个插件 entry 的配置、设置、依赖关系

- **Profile**：一个应用组合，放在 `~/.dsh/profiles/<name>/` 这个目录下。叠哪些 bundle、按什么顺序叠。同目录下的 `cordis.patch.yml` 是属于你自己的那一层。一个 profile 的组成：
	- **名字**：`web`、`headless`，或者你自己起的名字。
	- **Bundles**：指定这个应用组合叠哪些 bundle、按什么顺序叠，可以在 `package.json` 里通过`dsh.profile.bundles` 决定
> [!question] 为什么 Profile 存成一个目录，而不是一个配置文件




最终清单的算法就是：**从空数组 `[]` 开始，把所有 patch 按顺序执行一遍。**



#### Patch

Patch 是一个 YAML 数组，语义实现在 `applyEntryPatches`(`vendor/include/src/index.ts#L57`)。

**写法**：
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

**真实示例**：

`apps/web/tests/pin-browse-picker.overlay.yml`：先禁用原生的目录选择器，再插入浏览器内的选择器，一条禁用加一条插入就完成了替换。

#### Bundle

Bundle 是一个普通的 npm 包，在自己的 `package.json` 里声明：

```json
"dsh": { "bundle": { "patch": "./cordis.patch.yml" } }
```

它把**配置**（patch 文件）和**代码**（patch 里引用的插件包，作为它的依赖）一起分发。`patch` 字段可以是单个文件，也可以是按顺序排列的文件列表。

内置的 Bundle 在 `packages/bundle/` 下：

| Bundle                        | 作用                                          |
| ----------------------------- | ------------------------------------------- |
| `dsh-base`                    | 共享核心：用一条 `insert` 插入几十行，包括 LLM、会话、凭据、存储、工具等 |
| `dsh-web-app`                 | 叠在 base 上的浏览器应用                             |
| `dsh-headless`                | 一次性命令行任务，没有服务器                              |
| `dsh-sdk-app` / `dsh-acp-app` | SDK JSON-RPC 服务器 / ACP 服务器                  |
| `dsh-sdk-minimal`             | 例外：独立提供一棵完整的配置树，不依赖 base                    |

Bundle 之间靠 id 覆盖，以 web-app 为例：

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

另外几点：

- 第三方插件也可以做成 Bundle，用 `dsh plugin --profile <name> add <package>` 安装。
- 实验功能大多以**可选 Bundle** 的形式随安装附带，默认关闭，可以在插件管理器里打开（清单见 `packages/boot/app-boot/src/profile.ts#L223`）。
- 只有直接列在 Profile 里的 Bundle 才会生效，不会传递地带入它依赖的其他 Bundle。
- 某个 Bundle 加载失败时，只会跳过它并报告，不会让整个启动失败。

#### Profile

Profile 是 Harness home 下的一个目录。默认位置是 `~/.dsh`，可以用 `$DSH_HOME` 改：

```
~/.dsh/profiles/web/
├── package.json         # dsh.profile.bundles：按顺序的 Bundle 列表
│                        # dependencies：你装的第三方插件（由 pnpm 管理）
├── cordis.patch.yml     # 这个 profile 自己的 patch 层，初始内容是 []
├── pnpm-workspace.yaml  # 让第三方插件和安装共用同一个 cordis 实例
└── compatibility.json   # 可选，记录插件版本豁免
```

内置模板写死在 `packages/boot/app-boot/src/profile.ts#L179`：

| Profile       | Bundle 栈                    |
| ------------- | --------------------------- |
| `web`         | `dsh-base` + `dsh-web-app`  |
| `headless`    | `dsh-base` + `dsh-headless` |
| `sdk`         | `dsh-base` + `dsh-sdk-app`  |
| `acp`         | `dsh-base` + `dsh-acp-app`  |
| `sdk-minimal` | `dsh-sdk-minimal`           |

- 第一次运行 `dsh --profile web` 时，会按模板自动初始化这个目录。之后再运行，已有文件一律不覆盖。
- `dsh --profile my --from-default-profile web` 会从模板复制出一个独立的新 profile。它不记录继承关系，以后模板更新不会影响它；对已有的 `my` 再执行一次，会把它重置。
- Desktop 应用使用保留的 `desktop` profile。

#### 叠加顺序

具体实现在 `packages/boot/app-boot/src/profile-context.ts#L74-L80` ：

```
空列表 []
 ① Bundle 层   按 dsh.profile.bundles 的顺序：dsh-base → dsh-web-app
 ② Profile 层  ~/.dsh/profiles/web/cordis.patch.yml   只对这个 profile 生效
 ③ Home 层     ~/.dsh/cordis.patch.yml                对所有 profile 生效
 ④ 命令行      --patch a.yml --patch b.yml            只对这次启动生效
 ─────────────────────────────────────
 = 最终条目列表 → 交给 Cordis Loader 挂载
```

后面的层覆盖前面的层，同一个 id 以最后一次写入为准。所以你不用 fork 仓库，只要在 ②、③ 或 ④ 里写一条 `- id: xxx` 就能改掉产品里的任何一行。

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

完整工具清单见 [docs/tool-catalog.zh.md](vscode-webview://09d0qov5j8ic8cql960hkoppsi4qn0vbk7gq68f1gjcm51lej76i/docs/tool-catalog.zh.md)。这份清单由脚本实际启动每个工具插件生成。

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




## 插件开发

> [!quote]
> 插件 = 一个「bundle」包

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



## Cordis

`cordiverse/cordis` 是一个元框架（Meta Framework），用来支持 DeepSeek Harness 底层插件系统的**无损热插拔**。

> [!caution] 核心问题
> **如何真正实现 Agent 插件系统的无损热插拔**，也就是**如何解决插件的时间、空间可组合性**。
> 
> 未来的自进化 Agent 需要一边服务一边生成、部署、替换自己的组件。如果没有时间可组合性，每次自我修改就都要重启、丢弃全部进程内状态，且一个错误的自我修改可能禁用掉用于恢复的进程本身；如果没有空间可组合性，模块只能用临时手段检测依赖的变化，或破坏依赖、引入循环依赖。

> [!quote]- 相关论文：*A Programming Paradigm for Spatiotemporal Composability*
> 论文一作 Yifan Shi（即 `cordiverse/cordis` 项目作者）用接近 80 页的论文讨论一个非常工程化的问题：
> >[!question] 
> >当一个软件系统由大量可以动态加入、移除、替换的组件组成时，我们能不能做到：删掉一个组件时，把它留下的副作用完整撤销；它依赖的组件发生变化时，又能自动、安全地重新组织依赖关系，而不需要重启整个进程？
> 
> 作者把前者叫 **Temporal Composability**（时间可组合性），后者叫 **Spatial Composability**（空间可组合性），这两个概念是整篇论文真正的主线。
> 
> 1. **要解决的核心痛点**：传统软件组件（函数/模块）是静态组合的，而未来的 Agent 需要在运行时动态生成、替换自己的工具和记忆模块。如果每次变动都重启进程，会丢失缓存和连接；如果手动写清理代码，极易遗漏，导致系统“烂摊子”越来越多。Cordis 试图把“进程/容器级”的生命周期隔离能力，下沉到“组件/模块级”。
> 2. **两大核心机制**
> 	- **时间可组合性（撤销副作用）**：借鉴“事务日志”思想。组件做任何操作（如注册路由、启动定时器）时，必须同时提供“逆操作”（cleanup）。运行时自动按“后进先出”顺序执行逆操作，确保卸载组件时能把它的所有影响彻底擦除。
> 	- **空间可组合性（动态依赖管理）**：组件明确声明自己“需要什么”（依赖）和“提供什么”。运行时根据依赖是否满足，自动驱动组件激活或停用。如果某个服务（如数据库）被卸载，依赖它的组件会先安全退出，等清理完再销毁数据库，保证生命周期有序。
> 3. **理想目标**：**汇合性（Confluence）**
> 	- Cordis 追求的不是“当前能运行”，而是“**运行历史不可观察**”。无论中间经历了多少次安装、卸载、回滚，只要最终留下的组件配置是一样的，系统的最终状态就应该等同于“直接从头加载这些组件”，不会残留任何幽灵状态。
> 4. **明确的边界与局限（重要）**
> 	- **无法撤销外部世界**：发邮件、扣款、写外部文件等“对外发射”的副作用，Cordis 无法撤销，只能靠业务补偿（如退款）。
> 	- **依赖开发者“诚实”**：框架不会魔法般自动修复代码，如果开发者在逆操作里忘了写清理代码，或者多个操作互相冲突（如中间件顺序），形式化证明的前提就不成立了。
> 	- **尚未完全验证**：论文目前主要验证于已有的 **Koishi** 聊天机器人生态，**还未真正在“Agent 高频自进化”的场景下做严格的性能对比测试**。

### 核心概念

1. **基本定义**
	- **组合**（**Composition**）：把简单的部分组装成复杂系统。
	- **插件**（**Plugin**）：核心程序之外的附加程序（核心和插件的范围是相对的）。
	- **静态组合**（**Static Composition**）：模块导入、函数调用等在运行前就确定，执行期不变。
	- **动态组合**（**Dynamic Composition**）：组件在运行时加载、卸载、重新配置。
2. **要解决的两大核心问题**
	- **时间可组合性**（**Temporal Composability**）：组件被移除时，它对共享环境做的所有修改必须能**完全、安全地逆转**（能撤销）。
	- **空间可组合性**（**Spatial Composability**）：组件必须能声明和解析彼此依赖，并在依赖变化时**协调生命周期**（依赖能随变）。
3. **理论工具**
	- **效应**（**Effect**）：组件**对环境做了什么**（修改）。
	- **共效应**（**Coeffect**）：组件**对环境需要什么**（依赖）。
4. **实现机制**
	- **可逆效应 → 保证时间可组合**：采用 **“LIFO（后进先出）撤销”**。每个修改动作都自动记录逆向操作，卸载时倒序执行。注意：恢复的是“**可观测到的原状态**”（观察等价），而非绝对的物理原样。
	- **响应式共效应 → 保证空间可组合**：采用 **“响应式声明依赖”**。组件声明依赖，运行时动态响应。核心规则是：**依赖缺失时组件不启动，依赖要退出时，必须等依赖它的消费者（组件）先退出，提供者才能销毁**。
5. **统一管理的中枢**
	- **上下文**（**Context**）：插件的共享运行时状态，充当 **“操作账本 + 当前状态 + 依赖表”** 的统一实体。所有跨组件交互都必须经过它，从而变得**可追踪、可撤销、可反应**。
6. **最终愿景（汇合性，Confluence）**
	- 无论经历过多少次加载、卸载、替换，系统**安静下来后的最终状态**，必须等价于 **“只用最后那套配置从头加载一次”** 的结果。
	- 也就是：**系统的状态由最终配置决定，而不是由历史操作决定**（类比 NixOS 的配置回滚思想，但粒度是组件级）。
7. **核心局限性（边界意识）**
	- **无法撤销外部影响**：只能撤销被 Context 记录的内部操作，无法撤销已发出的外部交互（如发邮件、扣款），只能靠业务补偿。
	- **依赖开发者可靠性**：框架提供规范，但无法自动验证插件作者写的逆操作（cleanup）是否正确。
	- **未来方向**：需结合静态分析或编程语言/操作系统协同设计，来约束 Agent 自我生成的插件。

### 核心用法

#### 开发示例

1. **克隆仓库并安装依赖**：
```bash
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
```

2. **编写插件**（教程推荐创建在 `tmp/cordis-tutorial/` 下，因为 `tmp/` 已被 git 忽略，随便写不影响）：
```typescript
// hello.ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello'

export function apply(ctx: Context) {
  console.log('hello from my first plugin')
}
```

3. **配置插件**：创建 `cordis.yml`：
```yaml
- name: './hello.ts'
```

4. **运行插件**：
```bash
node --import tsx ../../vendor/cordis/bin.js

# 预期输出：
# hello from my first plugin
```

具体运行过程如下：

1. 启动器创建根 `Context`，并挂载 `Loader` 插件。
2. Loader 读取 `cordis.yml`，解析 `./hello.ts`，然后将其作为子插件挂载。
3. Cordis 调用你的 `apply(ctx)`。

当没有任何内容继续运行时，进程会自行退出。

#### 运行机制

1. **插件形式**：Cordis 接受三种形式的插件：
	- **函数插件**（最常用，直接导出 `apply` 函数）
	- **对象插件**（导出含 `apply` 方法的对象）
	- **类插件**（继承 `Service` 类）

```typescript
import { Service, type Context } from '@deepseek-ai/cordis'

// 1. Function plugin (what you just wrote).
export function apply(ctx: Context) {}

// 2. Object plugin: an object with an `apply` method.
export const objectPlugin = {
  name: 'object-plugin',
  apply(ctx: Context) {},
}

// 3. Class plugin: a Service subclass (covered in chapter 3).
export class MyService extends Service {
  constructor(ctx: Context) {
    super(ctx, 'myTutorialService')
  }
}
```


2. **上下文**（**Context**）：插件通过 `apply(ctx: Context)` 函数与框架交互，`ctx: Context` 是插件注册其贡献（如服务、事件监听器）的唯一入口，所有 API 都挂载在它下面。

3. **依赖注入**（**Injection**）：插件通过 `inject` 数组声明它依赖的服务。Cordis 会确保所有依赖的服务就绪后，才激活该插件，加载顺序由依赖关系决定，而非 `cordis.yml` 中的配置顺序。

```typescript
// consumer.ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'consumer'
export const inject = ['greeter']

export function apply(ctx: Context) {
  console.log(ctx.greeter.greet('world'))
}
```


4. **生命周期与副作用**（**Effect**）：这是 Cordis 最核心的机制。Cordis 插件可能因修改配置、热重载、显式资源释放或所需服务消失而卸载。插件通过 `ctx.effect()` 注册的副作用和通过 `ctx.on()` 注册的事件监听器等，都会在插件卸载时**自动且逆向地撤销**。这种设计让热插拔和热重载变得安全可靠。
	- **Fiber 状态机**：
		- Fiber 可以理解为“这个插件的运行实例”，它保存了：
			- 插件本身及其配置
			- 父级 Context
			- 声明的依赖服务
			- 插件当前状态
			- 该插件创建的全部资源
			- 该插件提供的全部服务
		- 每个已加载插件实例都拥有一个 fiber，并在以下状态之间转换：
			- `PENDING`：已经声明，但所需服务尚不可用。
			- `LOADING` / `ACTIVE`：`apply` 正在运行／已经完成。
			- `FAILED`：`apply` 或配置校验抛出异常。
			- `UNLOADING` / `DISPOSED`：disposer 正在运行／一切均已拆除。
```
PENDING → LOADING → ACTIVE → UNLOADING → DISPOSED
                  ↘ FAILED
```

```typescript
// lifecycle.ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'lifecycle-demo'

function heartbeat(ctx: Context) {
  console.log('heartbeat plugin loading')
  ctx.effect(() => {
    const timer = setInterval(() => console.log('tick'), 200)
    return () => {
      clearInterval(timer)
      console.log('heartbeat cleaned up')
    }
  })
}

export function apply(ctx: Context) {
  // Mount a child plugin and keep its fiber to dispose it later.
  const fiber = ctx.plugin(heartbeat)
  // The demo timer is itself an effect: if THIS plugin is unloaded first,
  // the pending callback is cancelled instead of firing on a dead app.
  ctx.effect(() => {
    const timer = setTimeout(async () => {
      await fiber.dispose()
      console.log('disposed')
      process.exit(0)
    }, 700)
    return () => clearTimeout(timer)
  })
}
```

 `cordis.yml`：
```yaml
- name: './lifecycle.ts'
```


5. **服务**（**Service**）：服务是插件提供、其他插件通过 `ctx` 消费的具名能力。例如 `ctx.tools`、`ctx.llm` 都是服务。服务通过继承 `Service` 类来创建，并使用 `declare module` 进行 TypeScript 类型合并，以获得类型安全。

6. **事件**（**Event**）：事件是插件间解耦的通信方式。插件通过 `ctx.emit()` 等方法发送事件，其他插件通过 `ctx.on()` 监听。监听器本身也是一个 effect，会随插件自动移除。Cordis 提供了五种事件分发模式。
	- **`emit`**：
		- 同步广播；不会等待或收集返回的 promise 与值
		- 调用：`ctx.emit(name, ...args)`
	- **`parallel`**：
		- 所有监听器并发运行，并一同等待
		- 调用：`await ctx.parallel(name, ...args)`
	- **`serial`**：
		- 监听器按顺序运行并等待；第一个非 null/false/undefined 返回值胜出，并停止后续监听器。
		- 调用：`await ctx.serial(name, ...args)`
	- **`bail`**：
		- serial 的同步版本
		- 调用：`ctx.bail(name, ...args)`
	- **`waterfall`**（瀑布式事件）
		- 环绕中间件，转换或短路，每个监听器都会收到参数和一个 `next()` continuation；它可以转换 `next()` 的返回值，也可以不调用 `next()` 就直接返回，从而短路链条的其余部分。
		- 调用：`ctx.waterfall(name, ...args, next)`

#### 核心 API

| API                                       | 所属模块      | 说明                                                                      |
| ----------------------------------------- | --------- | ----------------------------------------------------------------------- |
| **`apply(ctx: Context)`**                 | 插件自身      | 插件的入口函数，Cordis 加载插件时会调用此函数，并传入 `Context` 对象。                            |
| **`ctx.plugin(plugin)`**                  | `Context` | **挂载子插件**。将一个插件挂载为当前插件的子插件，返回一个 `Fiber` 实例。父插件卸载时，所有子插件也会被递归卸载。         |
| **`ctx.effect()`**                        | `Context` | **注册可逆副作用**。接收一个函数，该函数应返回一个 `disposer`（资源释放函数）。插件卸载时，`disposer` 会被自动调用。 |
| **`ctx.on(event, listener)`**             | `Context` | **注册事件监听器**。监听指定事件。监听器本身是一个 effect，会在插件卸载时自动移除。                         |
| **`ctx.emit(event, ...args)`**            | `Context` | **触发事件**。同步广播事件，不等待监听器返回。                                               |
| **`ctx.parallel(event, ...args)`**        | `Context` | **触发事件**。所有监听器并发执行，并一同等待。                                               |
| **`ctx.serial(event, ...args)`**          | `Context` | **触发事件**。监听器按顺序执行并等待，第一个非空返回值胜出。                                        |
| **`ctx.bail(event, ...args)`**            | `Context` | **触发事件**。`serial` 的同步版本。                                                |
| **`ctx.waterfall(event, ...args, next)`** | `Context` | **触发事件**。实现拦截模式，监听器可转换 `next()` 的返回值或短路。                                |
| **`ctx.get(key)`**                        | `Context` | **获取服务实例**。用于获取可选依赖的服务，若服务不存在则返回 `undefined`。                           |
| **`inject: string[]`**                    | 插件导出      | **声明硬性依赖**。插件通过导出 `inject` 数组来声明它依赖的服务名称。                               |
| **`Config`**                              | 插件导出      | **声明配置结构**。插件通过导出 `Config` 对象（使用 Schema 定义）来声明其可接收的配置。                  |
| **`fiber.dispose()`**                     | `Fiber`   | **手动卸载插件**。手动触发一个插件的卸载流程，会等待所有清理工作完成。                                   |
