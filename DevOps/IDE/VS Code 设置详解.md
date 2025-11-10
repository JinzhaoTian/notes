
## 开发设置

VS Code 内置了对 Node.js 运行时的调试支持，可以调试 JavaScript、TypeScript 或任何其他转换为 JavaScript 的语言。

要调试其他语言和运行时（包括 PHP、Ruby、Go、C#、Python、C++、PowerShell 等），请在 VS Code Marketplace 中查找调试器扩展，或在顶级“运行”菜单中选择“安装其他调试器”。

对于大多数调试场景，VS Code 将调试配置信息保存在位于**工作区**（**项目根文件夹**）的 `.vscode` 文件夹中或用户设置或工作区设置中的 `launch.json` 文件中。

最好是创建启动配置文件 `launch.json`，这样可以配置和保存调试设置详细信息。 

![](imgs/Pasted%20image%2020240112154735.png)


#### 调试模式

在 VS Code 中，有两种核心调试模式：`Launch` 和 `Attach`，分别处理两种不同的工作流程和开发人员部分。
- `Launch` 就是让 IDE 启动进程，并且 IDE 会自动将其调试器附加到新启动的进程。
- `Attach` 就是应用程序或进程已经运行，此时 IDE 只需将其调试器附加到已经打开的应用实例中。

#### `launch.json`

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}\\app.js"
    }
  ]
}
```

`"version"` 是版本号，比较重要的就是 `"configurations"` 。`"configurations"` 是一个列表的obj，说明可以添加多个设置。

在 `"configurations"` 里，必须的字段属性有，
- `"name"` - 给个配置的名字。
- `"type"` - 调试器的类型，对于 node.js 的调试器就是 `"node"` ，对于 Go 的调试器就是 `"go"` 。
- `"request"` - 两种核心调试模式 Launch 和 Attach，分别是 `"launch"` 和  `"attach"`。

可选的字段属性有，
- `"presentation"` - 通常有 `order`, `group`, 和 `hidden` 等属性值。
- `"preLaunchTask"` - 要在调试会话开始之前启动一些任务，请将此属性设置为 `tasks.json`（同样在工作区的 `.vscode` 文件夹中）中指定的任务的标签。或者设置为 `${defaultBuildTask}` 以使用默认构建任务。
- `postDebugTask` - 要在调试会话最后启动任务，请将此属性设置为 `tasks.json` 中指定的任务名称。
- `internalConsoleOptions` - 控制调试会话期间调试控制台面板的可见性。
- `debugServer` - 调试时的特殊设置
- `serverReadyAction` - 调试中的程序向调试控制台或集成终端输出特定消息时触发动作

大部分的调试器都支持下面的字段属性，
- `"program"` ：调试器启动时需要执行的程序或者文件。
- `"args"` - 调试时传递给程序的参数。
- `"env"` - 环境变量
- `"envFile"` - 保存环境变量的 `.env` 文件路径。
- `"cwd"` - 可以找到依赖项和文件的当前工作路径。
- `"port"` - 连接到正在运行的进程时的端口。
- `"stopOnEntry"` - 程序启动时立即中断。
- `"console"` - 使用哪种类型的控制台。


#### Remote Debugging

VS Code 本身不支持远程调试，但是对于 Node.js 调试器的远程调试是支持的。



### Tasks

有很多工具可以自动执行诸如检查、构建、打包、测试或部署软件系统等任务，这些工具大多从命令行运行，并自动执行内部软件开发循环内部和外部的作业（编辑、编译、测试和调试）。

在 VS Code 中可以通过配置**任务**（**Tasks**）为运行脚本或者启动进程，以便可以在 VS Code 中使用许多现有工具，而无需输入命令行或编写新代码。 

工作区或文件夹特定任务是从工作区的 `.vscode` 文件夹中的 `tasks.json` 文件配置的。

![](imgs/Pasted%20image%2020240112165444.png)



#### tasks.json

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "type": "typescript",
      "tsconfig": "tsconfig.json",
      "problemMatcher": ["$tsc"],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```









