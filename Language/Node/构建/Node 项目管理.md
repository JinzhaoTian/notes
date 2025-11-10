

## 项目配置

`package.json` 是一个包含项目元数据的文件，定义了项目的依赖、脚本和其他配置。而 `package-lock.json` 则记录了确切的依赖版本，确保在不同环境中安装时，依赖树的一致性和可靠性。简而言之，`package.json` 用于定义和管理依赖，`package-lock.json` 确保依赖版本的一致性。

#### package.json

**作用**：
- **初始化项目**：使用 `npm init` 创建新的 Node.js 项目时生成。
- **安装依赖**：运行 `npm install <package>` 时，依赖会被添加到 `dependencies` 或 `devDependencies` 中。
- **定义脚本**：可以定义可通过 `npm run <script-name>` 执行的自定义命令。
- **版本控制**：指定项目的版本号和其他元数据。



**字段**：
- `dependencies` ：项目在运行时所需的包。
- `devDependencies` ：开发时所需的包，但在生产环境中不需要，用于测试、构建工具、代码质量检查等。例如，像 Webpack、Babel、ESLint等工具通常会放在`devDependencies`中。

在安装依赖时，如果你使用`npm install`，它会安装`dependencies`和`devDependencies`。但如果你在生产环境中安装（使用`npm install --production`），只会安装`dependencies`。



使用 `Typescript` 时，要注意添加的是哪种类型的依赖。因为 `Typescript` 是一个开发工具，而且`TypeScript` 类型不存在于运行时，**与 `TypeScript` 相关的包一般属于 `devDependencies`** 。



#### package-lock.json

**作用**：
- **锁定依赖版本**：每次安装依赖时，`package-lock.json` 会更新以反映安装的确切依赖版本及其依赖树。
- **确保一致性**：在不同环境（如开发、测试、生产）中安装依赖时，保证每次安装得到相同的版本，避免因依赖版本差异引发的问题。
- **快速安装**：在后续安装时，`npm` 可以直接使用 `package-lock.json` 中的信息，快速安装所有依赖。