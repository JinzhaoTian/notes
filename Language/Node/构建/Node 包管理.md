## npm

npm（Node Package Manager）是 Node 的默认包管理器，会随 Node 一起自动安装。**npm 是独立项目，是用 JavaScript 写的 CLI，运行在 Node 上**。

> [!question] 已经有了 node 了，为什么还要有 npm ?
> node 只提供运行环境和核心 API，通过 Node 只代表你能执行 JavaScript、能调用 `fs`/`http`/`path` 等核心模块；
> 
> npm 负责帮你找到、下载、安装、升级、发布第三方包，处理复杂的依赖关系。
> - 从 registry 查询可用版本
> - 按 SemVer 解析版本范围
> - 递归解析依赖的依赖
> - 处理版本冲突
> - 扁平化安装到 `node_modules`
> - 用 `package-lock.json` 锁定精确版本
> - 校验完整性哈希
> - 使用缓存加速安装
> 
> npm 集成了命令：
> - `npm publish`：把包发布到注册表
> - `npm run`：运行项目脚本，并自动把 `node_modules/.bin` 注入 `PATH`
> - `npm exec` / `npx`：执行包提供的 CLI
> - 生命周期脚本：`preinstall`、`postinstall`、`prepare` 等
> - 访问控制、团队包、私有 registry 等生态能力

### 主要功能

1. **下载和安装**：开发者可以从 npm 的官方仓库（`https://www.npmjs.com/`）下载别人写好的、可复用的代码包（如 React, Vue, Express 等）到自己的项目中使用。
2. **管理依赖**：通过项目中的 `package.json` 文件，`npm` 可以清晰地记录和管理项目所依赖的所有包及其版本。
3. **发布和共享**：开发者也可以将自己编写的代码包打包，通过 `npm` 发布到官方仓库，供全球的开发者使用。
4. **运行脚本**：可以在 `package.json` 中定义脚本命令（如 `npm start`），方便地启动项目、运行测试等。

### npm 运行机制

npm 是一个精巧的 JS 应用（并非独立的原生二进制程序），本质上其命令的运行是 **Node 解释器执行一套 JavaScript 脚本**的过程。

#### 启动链路

当你在终端输入 `npm install` 时，背后发生的第一件事是**操作系统的 PATH 查找与进程替换**。

- **跨平台启动**：在 macOS/Linux 上，`npm` 是一个带 `#!/usr/bin/env node` 声明的 shell 脚本；在 Windows 上则是 `npm.cmd` 批处理文件。它们的作用都是**找到并调用 `node.exe`**，并将真正的 npm 逻辑脚本 `npm-cli.js` 的路径传给它。
- **真正的入口**：最终执行的命令类似于 `node.exe ".../npm/bin/npm-cli.js" install`。因此，任务管理器中看到的进程是 `node.exe`，**npm 的所有功能都源于 `npm-cli.js` 这个 JavaScript 脚本**。

#### 命令解析与配置加载

`npm-cli.js` 被 Node 启动后，会立即进行以下工作：

1. **解析命令行参数**：识别出子命令（如 `install`、`run`）以及附加的标志（如 `--save-dev`）。
2. **加载配置**：按优先级读取配置。优先级从高到低通常为：**命令行标志** > 环境变量（如 `NPM_CONFIG_*`）> 项目级 `.npmrc` 文件 > 用户级 `.npmrc` 文件 > 全局配置 > npm 内置默认值。注册表地址（`registry`）、缓存路径等都在此阶段确定。


#### 核心机制

依赖解析与安装（以 `npm install` 为例）
这是 npm 最核心也最复杂的过程，大致分为四步：**构建依赖树 → 下载 → 缓存 → 解压到 `node_modules`**[](https://developer.aliyun.com/article/1126996#slide-1#1)。

##### 1. 构建依赖树：从语义化版本到精确版本

npm 首先读取 `package.json`，根据 **语义化版本（Semver）** 规则（如 `^1.2.3`），将版本范围解析为注册表中可用的**具体版本号**。然后**递归解析传递依赖**（依赖的依赖），最终构建出一棵完整的依赖树[](https://cloud.tencent.cn/developer/techpedia/2701#1)。

##### 2. 依赖树在 `node_modules` 中的结构：扁平化与嵌套

npm 的目录结构经历过重要演变：

- **早期（npm v2）**：严格嵌套。每个依赖都有自己的 `node_modules`，导致**路径极深**（可能超出 Windows 限制）和**大量重复安装**。
    
- **现代（npm v3+）**：**扁平化（hoisting）**。npm 会尽可能将所有依赖**提升到项目根 `node_modules` 下**，以解决重复和路径过深的问题[](https://developer.aliyun.com/article/1126996#slide-1#1)。
    
- **冲突处理**：扁平化并非万能。当两个依赖需要**同一个包的不兼容版本**时，npm 会将其中一个版本留在根目录，另一个**嵌套安装在需要它的包的 `node_modules` 下**，以实现版本隔离[](https://cloud.tencent.cn/developer/techpedia/2701#1)。
    

##### 3. 锁定确定性：`package-lock.json` 的作用

纯粹基于 Semver 的扁平化算法，可能因解析顺序不同而产生**结构不确定性**（即“幽灵依赖”问题）。`package-lock.json` 正是为了解决这个问题而诞生。它**精确锁定了依赖树中每一个包的确切版本、下载地址和完整性哈希值**，确保任何人在任何时间执行 `npm install`，都会得到**完全一致**的 `node_modules` 结构[](https://cloud.tencent.cn/developer/techpedia/2701#1)[](https://developer.huawei.com/consumer/cn/forum/topic/0204860264272430235)。

#### 缓存机制

`_cacache` 内容寻址缓存

下载的包并不会直接丢弃。npm 使用一个名为 **`_cacache`** 的目录来缓存所有下载的 tarball 和 HTTP 请求数据。

- **内容寻址**：缓存以文件**内容的哈希值**作为键来存储。这意味着**相同内容永远只存一份**，天然避免了重复缓存，并能通过哈希校验确保数据完整性。
    

#### 脚本执行机制

`npm run` 是如何工作的？

`npm run <script>` 的机制与安装命令不同，其核心在于**环境隔离与路径注入**。

1. **创建子 Shell**：npm 会为脚本命令**新建一个子 Shell 环境**来执行，避免污染当前终端会话。
    
2. **注入 `node_modules/.bin` 到 PATH**：这是关键一步。npm 会**临时将项目本地的 `node_modules/.bin` 目录添加到 `PATH` 环境变量的最前端**。这使得你可以直接运行本地安装的 CLI 工具（如 `eslint`），而无需全局安装或输入完整路径[](https://raw.githubusercontent.com/Alessandro-Pang/fe-interview/refs/heads/main/content/docs/engineering/npm/npm-09.md)。
    
3. **注入环境变量**：npm 还会注入一系列以 `npm_` 为前缀的环境变量（如 `npm_package_version`），方便脚本内部读取项目信息[](https://raw.githubusercontent.com/Alessandro-Pang/fe-interview/refs/heads/main/content/docs/engineering/npm/npm-09.md)。
    
4. **执行命令**：最后，Shell 在修改后的 `PATH` 中查找并执行对应的可执行文件（通常是 `.bin` 目录下的软链接，指向包内 `bin` 字段声明的真实脚本）[](https://raw.githubusercontent.com/Alessandro-Pang/fe-interview/refs/heads/main/content/docs/engineering/npm/npm-09.md)。
    

#### 生命周期脚本

npm 在特定操作前后会自动触发“生命周期脚本”，最常见于安装过程：

- **`preinstall`**：在包开始安装**之前**运行。
- **`install`** / **`postinstall`**：在包**安装完成后**运行。许多需要编译原生模块或下载额外二进制的包（如 `node-sass`、`puppeteer`）都依赖 `postinstall` 来完成最终配置。

#### 全局命令与本地命令的差异

npm 命令的执行位置决定了其作用域：

- **本地（Local）**：在项目目录内执行，操作的是**项目专属**的 `node_modules` 和 `package.json`。
- **全局（Global）**：使用 `-g` 标志或 `npm link`，包被安装到系统级的全局目录（如 `/usr/local/lib/node_modules`），其 `bin` 命令会被链接到全局 `PATH` 中，可在任意位置调用。现代最佳实践倾向于**将 CLI 工具作为本地开发依赖安装**，通过 `npx` 或 `npm run` 调用，以获得版本确定性。







### npm 包组成

一个 npm 包是**一个目录 + `package.json` 元数据**，发布时被打包成 `.tgz` tarball，安装后解压到 `node_modules` 中。

#### 最小构成

目录结构：
```
my-package/
├── package.json      # 必需，包的核心描述
├── index.js          # 代码入口，具体由 package.json 指定
├── README.md         # 说明文档，通常发布
└── LICENSE           # 许可证，通常发布
```

`package.json` 最小示例：
```json
{
  "name": "my-package",
  "version": "1.0.0",
  "main": "index.js"
}
```

#### `package.json`

```json
{
  "name": "my-pkg",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./utils": "./dist/utils.js"
  },
  "types": "./dist/index.d.ts"
}
```

1. **基本信息**
	- `name`：包名，可带 scope，如 `@scope/pkg`
	- `version`：语义化版本，如 `1.2.3`
	- `description`、`keywords`、`author`、`license`、`repository`
	- `private`：为 `true` 时禁止发布
2. **入口与模块格式**
	- `main`：CommonJS 入口，如 `index.js`
	- `module`：给打包工具的 ESM 入口，非 Node 官方字段
	- `browser`：浏览器环境入口
	- `types` / `typings`：TypeScript 类型声明入口
	- `exports`：现代 Node 推荐的导出映射，可控制子路径和条件导出
	- `type`：`"module"` 表示 `.js` 按 ESM 处理；默认 CommonJS
3. **依赖**
	- `dependencies`：运行时依赖
	- `devDependencies`：开发/测试/构建依赖，不随包安装给使用者
	- `peerDependencies`：要求宿主项目提供的依赖，如 React 插件
	- `optionalDependencies`：可选依赖，安装失败可继续
	- `bundledDependencies`：随包一起打包的依赖
4. **脚本与生命周期**
	- `scripts`：如 `build`、`test`、`prepublishOnly`
	- `prepare`、`prepack`、`postinstall` 等生命周期钩子
5. **可执行命令**：安装后会在 `node_modules/.bin/` 下创建命令链接。
```json
{
  "bin": {
    "my-cli": "./bin/cli.js"
  }
}
```
6. **发布内容控制**
	- `files`：白名单，指定发布哪些文件/目录
	- `.npmignore`：忽略名单，类似 `.gitignore`
	- 默认会排除 `node_modules`、`.git` 等
	- `README`、`LICENSE`、`package.json` 通常总会被包含


#### 代码与辅助文件

一个完整包可能包含：
```
my-package/
├── package.json
├── README.md
├── LICENSE
├── src/              # 源码
├── dist/             # 构建产物
├── bin/              # CLI 可执行文件
├── types/            # 类型声明
├── test/             # 测试
├── examples/         # 示例
└── .npmignore
```

#### 安装后长什么样

安装后，包会出现在项目的：
```
node_modules/
└── my-package/
    ├── package.json
    ├── index.js
    └── ...
```

如果包提供 `bin`，还会有：
```
node_modules/.bin/my-cli
```

依赖树可能被 npm 提升/嵌套，具体由 npm 解析和 `package-lock.json` 决定。注意：**`package-lock.json` 属于项目，不属于 npm 包本身**；`node_modules` 也不会被打进 npm 包。



## pnpm

pnpm（**Performant NPM**）是一个**快速、高效、磁盘空间友好**的 JavaScript 包管理工具，旨在解决 npm 和 Yarn 在依赖管理和安装速度上的痛点。

### 核心特点
 
1. **高效磁盘利用**
	- 使用**硬链接（hard links）**和**符号链接（symlinks）**在全局存储（`~/.pnpm-store`）中共享依赖，避免重复下载相同的包，显著节省磁盘空间。
	- 即使多个项目使用相同依赖，物理存储仅保留一份。
2. **快速的安装速度**：依赖解析和下载速度通常比 npm/Yarn 更快，尤其是对于已有缓存的包。
3. **严格的依赖结构**：采用**扁平化+隔离**的策略，每个依赖都有独立的版本（避免“幻影依赖”问题），提升确定性。
4. **Monorepo 支持**：内置对多包仓库（Monorepo）的支持，通过 `pnpm-workspace.yaml` 管理，优化本地包之间的链接
5. **兼容性**：兼容 npm 的 `package.json` 和大多数 npm 命令，可无缝替代 npm/Yarn。


### 工作原理

1. **全局存储（Store）**：所有下载的包保存在全局存储中，项目中的依赖通过硬链接指向存储，减少冗余。
2. **依赖隔离**：每个包的依赖被严格放置在独立的目录中，避免版本冲突（例如，项目无法访问未声明的子依赖）。
3. **符号链接组织**：`node_modules` 结构扁平但非完全扁平，通过符号链接指向实际依赖，保持清晰的依赖关系。

### 使用示例



## npx

`npx` 是一个与 Node.js 的包管理工具 `npm` 紧密相关的命令行工具，可以把它看作是 `npm` 的“执行器”，其核心作用是让你能直接运行 npm 包里的命令，而无需全局安装它们。

|特性|**npm**|**npx**|
|---|---|---|
|**主要用途**|管理（安装、卸载、更新）项目依赖包|**执行** npm 包中的命令行程序|
|**使用方式**|通常用于安装包，如 `npm install -g create-react-app`|直接运行包，如 `npx create-react-app my-app`|
|**核心区别**|想要运行一个包的命令，通常需要先全局安装它|**无需提前安装**，npx会临时下载并执行，用完即走|

### 主要优点

1. **避免全局污染**：你不再需要为了偶尔使用某个工具而全局安装它，这能保持全局环境的干净。
2. **始终使用最新版本**：因为每次都是临时下载，所以可以确保使用的是该包的最新版本，避免了本地旧版本带来的问题。
3. **简化执行流程**：对于在 `node_modules/.bin/` 目录下的命令，`npx` 能自动找到并执行，省去了输入完整路径的麻烦。


### 使用场景

1. **创建新项目**：这是最经典的用法。例如，使用 `npx create-react-app my-app` 或 `npx create-next-app@latest my-app` 来一键创建项目模板。
2. **运行一次性脚本**：比如，你想用 `http-server` 快速启动一个静态服务器，但又不想安装它，直接运行 `npx http-server` 即可。
3. **执行本地包命令**：在项目内，可以用 `npx eslint .` 来运行本地的 ESLint 代码检查工具，而不必记住 `./node_modules/.bin/eslint .` 这样的路径。



