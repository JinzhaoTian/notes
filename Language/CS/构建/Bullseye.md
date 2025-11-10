Bullseye 是一个用于定义和运行构建目标的 .NET 库，常用于项目构建脚本中。它通常与简单的命令行工具结合使用，为 .NET 项目提供类似 Makefile 的功能。

## 主要特点

1. **目标定义**：可以定义多个构建目标及其依赖关系
2. **依赖管理**：自动处理目标之间的依赖关系
3. **并行执行**：支持并行运行独立的目标
4. **彩色输出**：提供清晰的可视化构建输出
5. **轻量级**：不依赖复杂的构建系统

## 典型用途

Bullseye 常用于：
- 替代复杂的构建脚本
- 定义开发工作流（构建、测试、打包等）
- 作为 CI/CD 流程的一部分

## 使用示例

```cs
using static Bullseye.Targets;

// 定义目标
Target("clean", () => 
{
    Console.WriteLine("清理构建输出");
    // 执行清理操作
});

Target("build", DependsOn("clean"), () => 
{
    Console.WriteLine("构建项目");
    // 执行构建操作
});

Target("test", DependsOn("build"), () => 
{
    Console.WriteLine("运行测试");
    // 执行测试
});

Target("default", DependsOn("test"));

// 运行目标
RunTargetsAndExit(args);
```