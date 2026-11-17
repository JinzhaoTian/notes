`cordiverse/cordis` 是一个元框架（Meta Framework），用来支持 [DeepSeek Harness](DeepSeek%20Harness.md) 底层插件系统的**无损热插拔**。

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

> [!info] Cordis 不是 DSH 专属的骨架，而是一个通用的插件运行时
> Cordis 只提供机制，不含任何业务功能：
> 
> - 插件怎么挂上去；
> - 依赖怎么找到（`provide` / `inject`）；
> - 副作用怎么自动清理（`effect`）；
> - 插件之间怎么通信（事件）；
> - 怎么按配置文件批量加载插件（`Loader` / `Include`）。

> [!info] Cordis 不是搭一次就完事的静态骨架，而是一直在运行
> 从 `new Context()` 到进程退出，Cordis 一直在管理每个插件的生死：
> - 某个服务换了实现，依赖它的插件就自动重启。
> - 热更新、插件管理器启停 bundle、改设置，最终都会变成 Loader 修改某几行，再由 Cordis 卸载或重新加载对应的 Fiber。
> - 退出时，`runProfile` 调用 `app.current?.fiber.dispose()`（`profile-boot.ts:285`），由 Cordis 按逆序把所有插件、服务、监听清理掉。

> [!caution] Cordis 局限
> 虽然在理论模型上做了抽象，但是工程上依然要处理异步、资源 IO、外部第三方服务（无法自动回滚外部 API）这类边界，可逆效应只能管理 Cordis 上下文内部的状态变更。


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

> [!caution] 和 OSGi / Spring Plugin 的本质差异
>  - **OSGi**：类加载隔离，生命周期手动管理，不强制副作用可逆；卸载后很容易残留状态；没有形式化证明。
>  - **Spring**：编译 / 启动期依赖注入，不支持运行时自动卸载组件并回滚状态。
>  - **Cordis**：强制副作用可撤销 + 响应式依赖驱动生命周期 + 形式化可证明，面向 AI Agent 动态组装场景。


### 核心 API

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


### 源码逻辑

#### 入口

在 DeepSeek Harness 下，Cordis 源码在 `vendor/cordis/src/` 下，Cordis 包本身的源码入口 `vendor/cordis/src/index.ts` 不含逻辑，只是把 `context`、`fiber`、`registry`、`events`、`service`、`logger`、`utils` 这几个文件的导出汇总到一起。

#### 启动

 Cordis 自己带了一个最小启动器 `vendor/cordis/bin.js`，全部逻辑如下：
```javascript
#!/usr/bin/env node

import { Context } from '@deepseek-ai/cordis'
import { pathToFileURL } from 'node:url'
import Loader from '@deepseek-ai/cordis-plugin-loader'

const ctx = new Context()
ctx.baseUrl = pathToFileURL(process.cwd()).href + '/'

await ctx.plugin(Loader)
await ctx.loader.create({
  name: '@deepseek-ai/cordis-plugin-include',
  config: {
    path: './cordis.yml',
  },
})

```

> [!tip] DeepSeek Harness 的启动器实现
> DeepSeek Harness 没有使用这个启动器，在 `packages\boot\app-boot\src\index.ts` 的 `boot()` 中实现了一样的逻辑骨架：创建 `Context`，挂 Loader，再往 Loader 里放一行 Include，并且 **DeepSeek Harness 在这几步之间插入了自己的逻辑**。

> [!info] Cordis 用到的 JavaScript 底层机制
> | 机制 | Cordis  | 了解程度 |
> | --- | --- | --- |
> | [原型链](../../../Language/JavaScript/JavaScript%20Prototype%20Chain.md) + [属性描述符](../../../Language/JavaScript/JavaScript%20属性描述符.md) | `extend` / `isolate` / `intercept` | 能手写 `Object.create` 链，说清 `in`、`Object.keys`、赋值三者的差别 |
> | [Proxy](../../../Language/JavaScript/JavaScript%20Proxy.md) + 内部方法 + receiver | `ReflectService.handler` | 能解释 trap 和默认算法的关系，知道 receiver 是谁 |
> | `Reflect` | trap 内部的 `Reflect.get/has/ownKeys` | 知道它是内部方法的函数形式，不是"另一个 API" |
> | [`this`](../../../Language/JavaScript/JavaScript%20this.md) / 调用 / 构造 | `return self`、`mixin` 的 bind、`createCallable` | 能预测"方法被取出后 this 是谁" |
> | Symbol（含 well-known） | `symbols` 表、`toPrimitive`、`hasInstance` | 知道 symbol 也是属性键，`Symbol.for` 是全局注册表 |
> | 迭代协议 + [Generators](../../../Language/JavaScript/JavaScript%20Generators.md) | `_execute` 的四种 effect 形态 | 能写 generator 逐个 yield disposer |
> | [Promise](../../../Language/JavaScript/JavaScript%20Promise.md) / 微任务   | `_reload` 的 `await Promise.resolve()`、`inertia` | 能排出微任务顺序，理解 thenable |

##### `Context`

`Context` 是 Cordis 框架依赖注入系统的核心代理对象，基于 [Proxy](../../../Language/JavaScript/JavaScript%20Proxy.md) 语法实现，

###### 主要功能

在 `Context` 实例内部：
- 所有属性访问操作（ `get`、`set` 等）拦截下来，转交给 `ReflectService.handler` 处理。
- 创建根 `Fiber`，它没有 runtime，构造时直接就是 ACTIVE（`vendor/cordis/src/fiber.ts:320-324`）。
- 装载 `reflect`、`registry`、`events`、`logger` 四个内置服务。
- 清空根 `Fiber` 的 `_disposables`（`vendor/cordis/src/context.ts:82`），这些内置能力就不会随根 dispose 被卸载。

###### 源码

```typescript
// vendor/cordis/src/context.ts

/**
 * Root and child dependency containers for Cordis plugins.
 */
export class Context {
  // ...
  
  /** Create the root context and install the built-in services. */
  constructor() {
    this[symbols.isolate] = Object.create(null)
    this[symbols.intercept] = Object.create(null)
    const self = new Proxy<this>(this, ReflectService.handler)
    this.root = self
    this.baseUrl = undefined
    this.fiber = new Fiber(self, {}, Object.create(null), null, () => [])
    this.reflect = new ReflectService(self)
    this.registry = new RegistryService(self)
    this.events = new EventsService(self)
    this.logger = new LoggerService(self)
    this.fiber._disposables.clear()
    return self
  }
  
  // ...
}
```

在构造函数最后 `return self` —— 构造函数返回对象会覆盖 `this`，所以 `new Context()` 拿到的是 `Proxy` 。`Context` 上没有 `services` 字典字段，只有四个常驻服务：`events / logger / reflect / registry`，加 `fiber / root / baseUrl`。

真正的东西全在 `Proxy` 的 `get` / `set` / `has` 三个 trap 里。

```typescript
// vendor/cordis/src/reflect.ts

// ...

/**
 * Reflection and service-resolution layer installed as `ctx.reflect`.
 *
 * This service powers the context proxy, service registration, accessors, and
 * the mixins that expose core service methods directly on `ctx`.
 */
export class ReflectService {
  /** Proxy traps implementing service resolution for every context object. */
  static handler: ProxyHandler<Context> = {
    get: (target, prop, ctx: Context) => { ... },   // 拦截 [[Get]]
    set: (target, prop, value, ctx: Context) => { ... }, // 拦截 [[Set]]
    has: (target, prop) => { ... },        // 拦截 [[HasProperty]]
  }

  // ...
  
  constructor(public ctx: Context) {
    defineProperty(this, symbols.tracker, {
      property: 'ctx',
      noShadow: true,
    })
    this.mixin('reflect', ['get', 'set', 'provide', 'accessor', 'mixin'])
    this.mixin('fiber', ['runtime', 'effect'])
    this.mixin('registry', ['inject', 'plugin'])
    this.mixin('events', ['on', 'once', 'parallel', 'emit', 'serial', 'bail', 'waterfall'])
  }
  
  /**
   * Register a service implementation owned by the current fiber.
   *
   * See the `ctx.provide()` overload above for the full contract.
   *
   * @param name — the service name.
   * @param value — the service value.
   * @param check — optional availability predicate for dependents.
   * @returns a disposer that unregisters the service.
   */
  provide(name: string, value?: any, check?: () => boolean) {
    return this.ctx.fiber.effect(() => {
      // ...
    }, `ctx.provide(${JSON.stringify(name)})`)
  }
}
```


> [!tip] DeepSeek Harness 的 Loader 和 Include
> DeepSeek Harness 还用到另外两个 Cordis 包：Loader（`vendor/loader`）和 Include（`vendor/include`）。


##### `Loader`

> [!quote] Cordis 核心只提供 `ctx.plugin(plugin, config)`，这种使用方式的前提是从代码里引入了插件对象。

Cordis 引入了 Loader（`vendor/loader`）包，将插件的加载模式改为用 entry 条目来加载插件，描述插件的方式为：`{ id, name, config, … }` 的意思是"按 `name` 去 import 这个模块，再用 `config` 启动它"。

Loader 的**核心能力**，是让运行中的插件始终和插件清单保持一致：

1. 通过 `ctx.loader.create / update / remove` 增删改 entry 条目（`vendor/loader/src/config/tree.ts`）；
2. 某一 entry 的配置改了，只重启那一个插件；行被标成 `disabled` 就卸载；行被删掉就销毁（见 `vendor/loader/src/config/entry.ts` 的 `update`）。

围绕这张插件清单，Loader 还提供了这些功能：

3. **分组**：`group: true` 的 entry 条目，它的 config 是一组子 entry（`vendor/loader/src/config/group.ts`）。
	- 单独的 `@cordisjs/plugin-group` 包只是把它重新导出了一下（`vendor/group/src/index.ts`）。
4. **每个 entry 独立的服务作用域**：`isolate` 让某一行里的插件使用独立的服务实例；`intercept` 给某一行附加针对某个服务的配置（`isolate.ts`）。
5. **`!!js` 表达式**：配置里可以写 JS 表达式，求值时以 context 作为作用域（`vendor/loader/src/config/utils.ts`）。
6. **把运行中的变化记回清单**：
    - 插件通过 `fiber.update()` 改了配置，新配置会写回对应的那一个 entry（`vendor/loader/src/index.ts:115-121`）；
    - 插件自己 dispose 了，那一个 entry 会被标成 `disabled`（`vendor/loader/src/index.ts:129-166`）。
7. **模块加载**：以 `cordis:` 开头的名字从内置表里取；其他名字交给 Node 内部的 ESM 加载器，按配置所在的位置解析，这同时也给 HMR 替换模块缓存留了入口（`vendor/loader/src/internal.ts`）。
8. **等待全部加载完**：依赖 `loader` 的插件可以声明 `await: true`，等所有 entry 都加载完再启动（`index.ts:175-179`）。


##### `Include`

Cordis 引入了 Include（`vendor/include`）包来为基于 entry 条目的插件清单提供文件支持，使用这个 Include 包可以将文件中的 entry 清单，加载为实际运行的 entry 清单（`vendor/include/src/index.ts`）。

Include 的**配置对象**共有四个字段（`vendor/include/src/index.ts:146-155`）：`path`、`initial`、`patches`、`enableLogs`。在 **Cordis 的原始设计**中 Include 自己只会去读 `path` 指向的那个文件，**从来不会从任何文件或命令行里读 `patches`**。

Include 的**核心能力**包括：

1. 启动时读 YAML 或 JSON（支持 `!!js` 标签），把内容变成 entry，挂成 Include 的子 entry；
> [!hint] Loader 的清单里始终只有 `include` 这一个 entry
> 通过 Include 在文件中加载的 entry 始终存在 Include 自己的 entry 树里，不在 Loader 的清单里。Loader 的清单里始终只有 `include` 这一个 entry。
2. 文件不存在时，用 `initial` 写一份初始内容；
3. 读完文件后可以叠加 `patches`：插入新 entry，或按 `id`;
> [!hint] Include 的 patches 机制就是 `cordis.patch.yml` 的核心
> 在 Cordis Include 原本的设计中，`patches` 是直接写在配置对象的 `config.patches` 里，后面加载的配置不能修改前面已经插入的 entry。
> 
> 在 DeepSeek Harness 中，将 `patches` 的来源修改为来自文件，定义的**加载顺序**（`packages/boot/app-boot/src/profile-context.ts:74`）是：每个 bundle 的 `cordis.patch.yml` → profile 的 `cordis.patch.yml` → `$DSH_HOME/cordis.patch.yml` → `--patch` 文件 → 遥测开关，**后面的覆盖前面的**，并且**后面的可以修改前面已经插入的 entry**。
> 
> 在 DeepSeek Harness 中，为了实现上述目标，添加了一些定制机制：
> - **文件校验**：顶层必须是数组，每一项必须是映射；可选的 patch 文件不存在，就当没有这一层；`--patch` 指定的文件不存在则直接报错（`packages/boot/app-boot/src/index.ts:515-588`）
> - **启动前检查**：DeepSeek Harness 在启动前先在空列表 `[]` 上把所有层应用一遍，顺带做兼容性检查，再把结果作为一个 `insert` 交给 Include（`packages/boot/app-boot/src/compatibility-preflight.ts:180`）。
4. 运行中 entry 有变化（改配置、插件自我禁用、增删 entry），防抖之后写回文件；
5. 文件被外部修改后，由 HMR 调用 `refresh()` 重读，只调整有差异的 entry。

> [!caution] Cordis 设想的应用形态
> 
> 就是它自带的 `bin.js`：先挂上 Loader，再放一行 Include 指向 `./cordis.yml`。换句话说，**一个 Cordis 应用就是一份 `cordis.yml`**：用户直接编辑这个文件，程序运行中的改动也写回这个文件。

> [!info] 从敲下 `dsh` 到第一次调用 Cordis
> ```
> await runCli()              // apps/cli/src/bin.ts:77 
> ├── parseDshArgs()          // apps/cli/src/bin.ts:29 
> │					        //   ← 解析 --profile、--patch 等启动器参数
> └── runProfile()            // apps/cli/src/bin.ts:35
>     ├── composeProfile()    // apps/cli/src/profile-boot.ts:292
>     │	                    //   ← 读取 profile，置空 cordis.yml，收集 patch
>     └── boot(NAME, cordis.yml, patches, prepare) 
>     	│			        // apps/cli/src/profile-boot.ts:323 
>         └── new Context()   // packages/boot/app-boot/src/index.ts:1207
>                             //   ← 第一次调用 Cordis
> ```
> 注意"加载 Cordis 模块"比"调用 Cordis"早得多：
> - `bin.ts` 一开头就静态导入了 `@deepseek-ai/dsh-app-boot`（`apps/cli/src/bin.ts:9`），app-boot 顶部又导入了 Cordis、Loader、Include、Group（`index.ts:14-17`）。所以进程一启动，Cordis 的代码就已经被求值了。
> - 但在 `new Context()` 之前，没有创建任何 Cordis 对象。
> 
> `runProfile` 在调用 `boot()` 之前做的几件事也都和 Cordis 无关：
> - 安装网络代理（`profile-boot.ts:276`）；
> - 注册 SIGINT/SIGTERM 和未捕获异常的处理（`:307-311`）；
> - 拼出 `profileContext` 对象（`:314-322`）。



### 开发示例

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

### 插件机制

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


4. **生命周期与副作用**（**Effect**）：这是 Cordis **最核心的机制**。Cordis 插件可能因修改配置、热重载、显式资源释放或所需服务消失而卸载。插件通过 `ctx.effect()` 注册的副作用和通过 `ctx.on()` 注册的事件监听器等，都会在插件卸载时**自动且逆向地撤销**。这种设计让热插拔和热重载变得安全可靠。
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


