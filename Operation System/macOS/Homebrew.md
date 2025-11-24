Homebrew 是 macOS（和 Linux）上的一个开源包管理工具，用于快速安装、更新、卸载和管理各种软件包（命令行工具、开发库、应用程序等），被称为“macOS 上缺失的包管理器”，极大简化了软件的安装和管理流程。

- `Formulae`：用于管理命令行工具和开发库
- `Cask`：用于管理 macOS 应用程序

## 安装

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装后，根据提示将 Homebrew 路径添加到环境变量（如 `~/.zshrc` 或 `~/.bashrc`）。



## 基本命令

- **更新 Homebrew**
```bash
brew update
```

- **搜索软件包**
```bash
brew search package_name
```

- **安装软件包**
```bash
brew install package_name
```

- **卸载软件包**
```bash
brew uninstall package_name
```

- **查看已安装的软件包**
```bash
brew list
```

- **查看软件包信息**
```bash
brew info package_name
```


## 更新与清理

- **更新所有软件包**
```bash
brew upgrade
```

- **更新指定软件包**
```bash
brew upgrade package_name
```

- **查看可更新的软件包**
```bash
brew outdated
```

- **清理旧版本软件包**
```bash
brew cleanup
```

- **清理指定软件包的旧版本**
```bash
brew cleanup package_name
```


## 服务管理

- **列出所有服务**
```bash
brew services list
```

- **启动服务**
```bash
brew services start service_name
```

- **停止服务**
```bash
brew services stop service_name
```

- **重启服务**
```bash
brew services restart service_name
```

## Cask 扩展

- **安装 Cask 软件**
```bash
brew install --cask application_name
```

- **卸载 Cask 软件**
```bash
brew uninstall --cask application_name
```


## 诊断与配置

- **检查 Homebrew 环境**
```bash
brew doctor
```

- **查看 Homebrew 配置**
```bash
brew config
```

- **查看版本信息**
```bash
brew --version
```

## 其他有用命令

- **查看软件包依赖关系**
```bash
brew deps package_name
```

- **查看软件包可选依赖**
```bash
brew options package_name
```

- **查看软件包安装路径**
```bash
brew ls --full package_name
```

- **创建符号链接（symlinks）**
```bash
brew link package_name
```


## 目录结构

Homebrew 通过一套精巧的目录结构和符号链接来管理软件：
1. `/opt/homebrew/Cellar/`：所有通过 Homebrew 安装的软件，其具体的文件都存放在这里，并按"软件名@版本/版本号"的方式组织。
2. `/opt/homebrew/bin/`：存放了指向 Cellar 或 opt 目录中真实可执行文件的符号链接（你可以理解为快捷方式）。
3. `/opt/homebrew/opt/`：这里存放着指向 Cellar 中某个软件当前激活版本的符号链接，为其他软件或依赖提供了一个固定不变的访问路径。

使用 Homebrew 安装的软件，一般都是在 `/opt/homebrew/bin/` 中存放符号链接（即设置好环境变量），方便调用，可以用以下命令建立或者切换版本：
```bash
brew link [--overwrite] package_name
```

