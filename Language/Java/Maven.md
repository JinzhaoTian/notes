Maven 是一个强大的项目管理和构建工具，主要用于 Java 项目的构建、依赖管理和项目信息管理。它由 Apache 软件基金会开发和维护，旨在简化项目的构建过程，并提供一种标准化的方式来管理项目结构、依赖项和构建生命周期。

### 核心概念

#### POM（Project Object Model）

POM 是 Maven 的核心文件，通常命名为 `pom.xml` ，是一个 XML 文件，包含项目的配置信息，例如：
- 项目的基本信息（名称、版本、描述等）。
- 依赖项（dependencies）。
- 构建配置（插件、目标等）。


#### 依赖管理

Maven 使用中央仓库（Central Repository）来存储和分发第三方库，开发者只需在 `pom.xml` 中声明依赖项，Maven 会自动下载并管理这些依赖。支持传递性依赖（Transitive Dependencies），即如果 A 依赖 B，B 依赖 C，则 Maven 会自动引入 C。


#### 构建生命周期

- Maven 定义了一组标准的构建生命周期，包括以下三个主要阶段：
    - **Clean** ：清理项目，删除生成的文件（如 `target` 目录）。
    - **Default** ：构建项目的核心生命周期，包括编译、测试、打包、安装等。
    - **Site** ：生成项目文档和报告。
- 每个生命周期由多个阶段（Phase）组成，例如：
    - `compile`：编译源代码。
    - `test`：运行单元测试。
    - `package`：将编译后的代码打包为 JAR、WAR 等格式。
    - `install`：将生成的包安装到本地仓库。


#### 插件机制

- Maven 的功能通过插件实现，每个插件提供一组目标（Goals）。
- 插件可以扩展 Maven 的功能，例如生成代码覆盖率报告、运行静态代码分析等。
- 示例插件：
    - `maven-compiler-plugin`：用于编译 Java 源代码。
    - `maven-surefire-plugin`：用于运行单元测试。


#### 仓库（Repository）

- **本地仓库** ：位于开发者机器上的目录（默认路径为 `~/.m2/repository`），用于存储下载的依赖项和构建结果。
- **远程仓库** ：中央仓库或私有仓库，用于共享和分发依赖项。


## 安装

### Windows

**二进制安装**：
- 下载：[Download Apache Maven – Maven](https://maven.apache.org/download.cgi)
- 解压：`unzip apache-maven-3.9.11-bin.zip` ， `tar xzvf apache-maven-3.9.11-bin.tar.gz`
- 将 `apache-maven-3.9.11` 下面的 `bin` 目录添加到环境变量 `PATH` 中

**包管理器安装**：
```bash
scoop install main/maven
```

### macOS

```bash
brew install maven
```


### Linux

```bash
sudo apt install maven
sudo yum install maven
```