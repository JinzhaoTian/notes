Clangd 是 LLVM 项目中的一个工具，它是一个基于 Clang 的语言服务器协议（LSP，Language Server Protocol）实现，专门为 C、C++ 和 Objective-C 等语言提供代码补全、跳转定义、代码格式化、错误诊断等智能编辑功能。

### 适用场景

- **编辑器/IDE 集成**：通过 LSP 协议与 VS Code、Vim、Emacs、Sublime Text 等编辑器配合使用。
- **替代传统工具**：相比 `cquery` 或 `ccls`，`clangd` 是官方维护的现代解决方案。
- **大型项目支持**：能处理复杂代码库，依赖 `compile_commands.json` 文件获取编译信息。


### 安装

#### macOS

```bash
brew install llvm       # macOS（需手动将 clangd 添加到 PATH）
```


#### Linux

```bash
sudo apt install clangd  # Ubuntu
```

### 配置

1. 生成 `compile_commands.json`（若使用 CMake）：
```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 ..
```

2. 编辑器集成
	- **VS Code**：安装 `clangd` 扩展，禁用默认的 C/C++ 插件以避免冲突。
	- **Vim/Neovim**：通过 `coc.nvim` 或 `LSP 插件` 配置。


[Configuration](https://clangd.llvm.org/config.html)

