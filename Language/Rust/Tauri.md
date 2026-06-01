Tauri 是一个开源框架，用于使用 Web 技术（HTML、JavaScript、CSS）构建体积小巧、运行快速、安全可靠的跨平台应用程序。

## 核心优势

- **更小的应用体积**：Tauri 不打包完整的浏览器引擎，而是直接调用操作系统自带的 WebView 来渲染界面。这使得一个最小应用的体积可以小于 600KB，远小于 Electron 等框架打包出的动辄几十上百 MB 的应用。
- **更高的安全性**：Tauri 基于 Rust 语言构建，天然继承了其在内存、线程和类型方面的安全性。框架本身会定期进行安全审计，并采用"最小权限原则"设计，帮助开发者构建更安全的应用。
- **灵活的技术栈**：你可以使用任何主流前端框架（如 React、Vue、Svelte 等）来搭建用户界面。除了使用 JavaScript 与后端交互，你还可以在必要时使用 Rust、Swift 或 Kotlin 编写更底层、性能更强的后端逻辑。

> [!tip] 竞品
> Tauri 最大的竞争优势在于极致的轻量与安全，这使其成为 Electron 等传统框架的理想替代者，尤其适合对性能敏感的场景。

> [!info] “跨平台开发”领域竞品：[Wails](../Golang/框架/Wails.md)
> Wails 与 Tauri 理念几乎完全一致，但后端使用的是 Go 语言而不是 Rust。


## 工作原理

Tauri 应用采用多进程架构，主要由两部分协同工作：

1. **核心进程（后端）**：由 Rust 编写，是应用的入口，拥有直接访问操作系统底层能力的权限，负责窗口管理、文件系统访问等。
2. **WebView 进程（前端）**：负责通过系统自带的 WebView 渲染你的 HTML/CSS/JS 代码，构建用户界面。

两部分之间通过进程间通信（IPC）进行消息传递，前端使用 JavaScript 调用后端提供的功能，实现了前后端的分离。


## 开发

### 环境准备（先决条件）

在开始之前，需要安装以下系统依赖：

#### Windows

- **Microsoft C++ Build Tools**：安装时勾选"使用 C++ 的桌面开发"
- **Microsoft Edge WebView2**：系统通常已自带，如无则下载安装
- **Rust**：通过 [rustup](https://rust-lang.net.cn/tools/install) 安装

#### macOS

- **Xcode**：从 App Store 下载并启动完成初始化
- **Rust**：通过 `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` 安装

#### Linux

- **系统依赖**：根据发行版安装对应依赖（如 Ubuntu 需 `libwebkit2gtk`、`build-essential` 等）
- **Rust**：同上使用 rustup 安装

#### 通用要求

- **Node.js**（LTS 版本）：用于前端开发
- **可选包管理器**：pnpm、yarn（通过 `corepack enable` 启用）


### 创建项目

#### 方式一：使用 create-tauri-app（推荐）

在目标文件夹运行以下命令：
```bash
# npm
npm create tauri-app@latest
# yarn
yarn create tauri-app
# pnpm
pnpm create tauri-app
# bun
bun create tauri-app
```

按照提示依次选择：
1. **项目名称**（如 `my-tauri-app`）
2. **Bundle Identifier**（应用唯一标识，如 `com.myapp.dev`）
3. **前端语言**：TypeScript / JavaScript
4. **包管理器**：npm / yarn / pnpm
5. **前端框架**：React、Vue、Svelte、Vanilla 等

完成后进入项目目录并安装依赖：

```bash
cd my-tauri-app
npm install
```

#### 方式二：添加到现有项目

如果已有前端项目（如 Vite + React）：
```bash
# 添加 Tauri CLI（全局安装）
npm install -g @tauri-apps/cli
# 在项目根目录初始化 Tauri
npx tauri init
```

然后修改 `src-tauri/tauri.conf.json` 中的配置：
- `build.frontendDist`：前端构建输出目录（如 `../dist`）
- `build.devUrl`：开发服务器地址（如 `http://localhost:5173`）

### 开发与调试

#### 启动开发服务器

```bash
# 桌面端开发
npm run tauri dev
# Android 开发
npm run tauri android dev
# iOS 开发
npm run tauri ios dev
```

首次运行会下载 Rust 依赖包，**可能需要几分钟**，后续会快很多。

#### 调试技巧

- **打开 Web 检查器**：右键点击应用窗口 → 检查，或使用快捷键 `Ctrl/Cmd + Shift + I`
- **移动端调试**：
    - **Android**：在 Chrome 访问 `chrome://inspect`
    - **iOS**：在 Safari 的"开发"菜单中选择模拟器或设备
- **热重载**：修改前端代码时 WebView 会自动更新；修改 `src-tauri` 中的 Rust 代码会自动重新编译应用

#### 常用配置（tauri.conf.json）

```json
{
  "productName": "my-app",
  "version": "0.1.0",
  "identifier": "com.myapp.dev",
  "build": {
    "frontendDist": "../dist",      // 前端构建输出路径
    "devUrl": "http://localhost:5173", // 开发服务器地址
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build"
  },
  "app": {
    "windows": [{
      "title": "My App",
      "width": 1024,
      "height": 768,
      "resizable": true
    }]
  }
}
```

### 打包构建

#### 打包为桌面应用

```bash
npm run tauri build
```

生成的安装包位于：`src-tauri/target/release/bundle/`

常见打包问题及解决：

|错误提示|原因|解决方法|
|---|---|---|
|`identifier com.tauri.dev is not allowed`|使用了默认 identifier|改为唯一值，如 `com.yourname.appname`|
|`failed to run light.exe` (Windows)|VBSCRIPT 功能被禁用|设置 → 应用 → 可选功能 → 开启 VBSCRIPT|

#### 移动端打包准备

如需发布到 Android/iOS，还需额外配置：

- **Android**：安装 Android Studio，配置 `ANDROID_HOME` 环境变量
- **iOS**：通过 rustup 添加 iOS 目标：`rustup target add aarch64-apple-ios`

### 项目结构说明

```
my-tauri-app/
├── src/                    # 前端源代码（React/Vue 等）
├── src-tauri/              # Rust 后端
│   ├── src/
│   │   ├── lib.rs          # 主要 Rust 逻辑（移动端入口）
│   │   └── main.rs         # 桌面端入口（通常无需修改）
│   ├── Cargo.toml          # Rust 依赖管理
│   ├── tauri.conf.json     # Tauri 核心配置
│   ├── capabilities/       # 权限配置（声明 API 权限）
│   └── icons/              # 应用图标
├── index.html
└── package.json
```

> 📌 请将 `src-tauri/Cargo.lock` 提交到 git，但**不要提交** `src-tauri/target/` 文件夹。