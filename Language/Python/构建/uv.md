uv 是一个由 Rust 编写的、速度极快的 Python 包和项目管理工具，旨在成为一个统一的、高性能的解决方案，项目地址是：[astral-sh/uv](https://github.com/astral-sh/uv)。

## 核心优势

uv 通过 Rust 语言实现并行下载和智能缓存，性能远超传统工具。
1. **极快的速度**：比 pip 快 10-100 倍，安装依赖如 pandas 等大型库时，速度提升极为明显。
2. **统一工具链**：一个工具就能完成多个工具的工作，简化了开发环境。
3. **项目即环境**：摒弃了手动创建和激活虚拟环境的繁琐流程，通过 `uv run` 命令即可在项目专属环境中自动运行脚本。



## 命令

```
An extremely fast Python package manager.

Usage: uv [OPTIONS] <COMMAND>

Commands:
  auth     Manage authentication
  run      Run a command or script
  init     Create a new project
  add      Add dependencies to the project
  remove   Remove dependencies from the project
  version  Read or update the project's version
  sync     Update the project's environment
  lock     Update the project's lockfile
  export   Export the project's lockfile to an alternate format
  tree     Display the project's dependency tree
  format   Format Python code in the project
  audit    Audit the project's dependencies
  tool     Run and install commands provided by Python packages
  python   Manage Python versions and installations
  pip      Manage Python packages with a pip-compatible interface
  venv     Create a virtual environment
  build    Build Python packages into source distributions and wheels
  publish  Upload distributions to an index
  cache    Manage uv's cache
  self     Manage the uv executable
  help     Display documentation for a command

Cache options:
  -n, --no-cache               Avoid reading from or writing to the cache, instead using a temporary directory for the
                               duration of the operation [env: UV_NO_CACHE=]
      --cache-dir <CACHE_DIR>  Path to the cache directory [env: UV_CACHE_DIR=]

Python options:
      --managed-python       Require use of uv-managed Python versions [env: UV_MANAGED_PYTHON=]
      --no-managed-python    Disable use of uv-managed Python versions [env: UV_NO_MANAGED_PYTHON=]
      --no-python-downloads  Disable automatic downloads of Python. [env: "UV_PYTHON_DOWNLOADS=never"]

Global options:
  -q, --quiet...
          Use quiet output
  -v, --verbose...
          Use verbose output
      --color <COLOR_CHOICE>
          Control the use of color in output [possible values: auto, always, never]
      --system-certs
          Whether to load TLS certificates from the platform's native certificate store [env: UV_SYSTEM_CERTS=]
      --offline
          Disable network access [env: UV_OFFLINE=]
      --allow-insecure-host <ALLOW_INSECURE_HOST>
          Allow insecure connections to a host [env: UV_INSECURE_HOST=]
      --no-progress
          Hide all progress outputs [env: UV_NO_PROGRESS=]
      --directory <DIRECTORY>
          Change to the given directory prior to running the command [env: UV_WORKING_DIR=]
      --project <PROJECT>
          Discover a project in the given directory [env: UV_PROJECT=]
      --config-file <CONFIG_FILE>
          The path to a `uv.toml` file to use for configuration [env: UV_CONFIG_FILE=]
      --no-config
          Avoid discovering configuration files (`pyproject.toml`, `uv.toml`) [env: UV_NO_CONFIG=]
  -h, --help
          Display the concise help for this command
  -V, --version
          Display the uv version

Use `uv help` for more details.
```

1. **项目管理**
	- **`uv init`**：初始化新项目
		- `uv init myproject`
		- `uv init --app`：创建应用
		- `uv init --lib`：创建库
	- **`uv add`**：添加依赖包
		- `uv add requests`
		- `uv add --dev pytest`：开发依赖
		- `uv add "pandas>=1.0"`
	- **`uv remove`**：移除依赖包
		- `uv remove requests`
	- **`uv sync`**：同步安装所有依赖（根据 `pyproject.toml` 和 `uv.lock`）
		- `uv sync`
		- `uv sync --no-dev`：不安装开发依赖
	- **`uv lock`**：锁定依赖版本，生成/更新 `uv.lock` 文件
		- `uv lock`
		- `uv lock --upgrade`：升级所有依赖
	- **`uv run`**：在项目环境中运行命令
		- `uv run python script.py`
		- `uv run pytest`
		- `uv run --no-sync python script.py`：不先同步依赖
	- **`uv tree`**：显示依赖树
		- `uv tree`
		- `uv tree --depth 2`
	- **`uv build`**：构建分发包（生成 `.whl` 和 `.tar.gz`）
		- `uv build`
2. **环境与工具管理**
	- **`uv venv`**：创建虚拟环境
		- `uv venv`：创建 `.venv` 
		- `uv venv --python 3.11`：指定 Python 版本
		- `uv venv myenv`：自定义环境名
	- **`uv python`**：管理 Python 版本
		- `uv python list`：列出可用版本
		- `uv python install 3.11`：安装指定版本
		- `uv python pin 3.11`：固定项目使用的版本
	- **`uv tool`**：管理全局安装的工具
		- `uv tool install ruff`：安装全局工具
		- `uv tool list`：列出已安装工具
		- `uv tool upgrade ruff`：升级工具
		- `uv tool uninstall ruff`：卸载
	- **`uvx`**：运行工具（不持久安装）
		- `uvx ruff check .`：临时运行
		- `uvx --with pandas jupyter lab`：带依赖运行
3. **pip 兼容命令**
	- **`uv pip install`**：安装包（类似 `pip install`，但更快）
	- **`uv pip uninstall`**：卸载包
	- **`uv pip list`**：列出已安装包
	- **`uv pip freeze`**：输出 requirements 格式
	- **`uv pip compile`**：生成锁定的 `requirements.txt`
	- **`uv pip sync`**：同步到锁定的依赖集
4. **实用命令**
	- **`uv cache clean`**：清理缓存
	- **`uv cache list`**：查看缓存内容
	- **`uv --help`**：查看帮助
	- **`uv <command> --help`**：查看特定命令的帮助



### 工作流

```bash
# 1. 创建新项目
uv init myproject
cd myproject

# 2. 添加依赖
uv add fastapi uvicorn
uv add --dev pytest httpx

# 3. 查看依赖树
uv tree

# 4. 运行开发服务器
uv run uvicorn main:app --reload

# 5. 运行测试
uv run pytest

# 6. 锁定依赖（通常 uv add 会自动做，手动触发用）
uv lock

# 7. 构建项目
uv build
```

### 常用命令

```bash
# 最常用命令（日常开发）
uv init          # 开始新项目
uv add <pkg>     # 添加依赖
uv sync          # 同步环境
uv run <cmd>     # 运行命令
uvx <tool>       # 临时运行工具

# 环境管理
uv venv          # 创建虚拟环境
uv python install <ver>  # 安装 Python 版本

# 维护
uv lock --upgrade  # 升级所有依赖
uv cache clean     # 清理缓存
```




## 安装

```shell
# On macOS and Linux.
curl -LsSf https://astral.sh/uv/install.sh | sh

# Use Homebrew
brew install uv
```

```shell
# On Windows.
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Use WinGet
winget install --id=astral-sh.uv  -e
```

