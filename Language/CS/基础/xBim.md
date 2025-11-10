xBIM（_eXtensible Building Information Modeling_）是一个开源的 BIM（建筑信息模型）工具包，主要用于处理 IFC（Industry Foundation Classes）文件格式，支持建筑、工程和施工（AEC）行业的数据交换与管理。

NuGet 地址：[NuGet Gallery | Xbim.Essentials 6.0.521](https://www.nuget.org/packages/Xbim.Essentials)

安装，
```bash
dotnet add package Xbim.Essentials --version 6.0.521
```

## 主要特点

1. **IFC 文件支持**
    - 支持 **IFC2x3、IFC4** 等主流版本，可用于读取、编辑和生成 IFC 文件。
    - 提供高效的 IFC 解析和几何处理能力。
2. **跨平台开发**
    - 基于 **.NET** 开发，可在 Windows、Linux 和 macOS（通过 .NET Core/.NET 5+）上运行。
    - 提供 **C#** 和 **VB.NET** 的 API，方便集成到 BIM 软件或自定义应用中。
3. **3D 可视化**
    - 内置 **WebGL** 和 **WPF** 渲染引擎，支持 BIM 模型的 3D 可视化。
    - 可用于开发 BIM 查看器或协作平台。
4. **BIM 数据操作**
    - 支持查询、修改和验证 BIM 模型数据（如构件属性、关系、分类等）。
    - 可用于自动化检查（如 clash detection）、数据提取和报表生成。
5. **开源与社区支持**
    - 采用 **MIT 开源协议**，可免费用于商业项目。
    - 由开发者社区维护，GitHub 上提供文档和示例代码。

