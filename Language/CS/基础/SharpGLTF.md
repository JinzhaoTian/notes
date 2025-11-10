SharpGLTF 是一个支持 [glTF](../../../Computer%20Graphics/Mesh/glTF.md) 文件格式的开源 .NET 库。

NuGet 地址：[NuGet Gallery | SharpGLTF.Toolkit 1.0.4](https://www.nuget.org/packages/SharpGLTF.Toolkit)
GitHub 地址：[vpenades/SharpGLTF: glTF reader and writer for .NET Standard](https://github.com/vpenades/SharpGLTF)

安装，
```bash
dotnet add package SharpGLTF.Toolkit --version 1.0.4
```



**核心功能**

1. **glTF 文件支持**：
    - 读写 **glTF 2.0** 标准格式（`.gltf`/`.glb`）。
    - 支持嵌入纹理、动画、蒙皮、材质（PBR 金属粗糙度工作流）等特性。
2. **跨平台**：
    - 基于 .NET Standard 2.0，可在 Windows、Linux、macOS 及 Unity 等环境中使用。
3. **工具链集成**：
    - 与 **Microsoft DirectX**、**Vulkan** 等图形 API 兼容。
    - 提供模型转换、优化、验证等功能（如合并网格、压缩纹理）。
4. **轻量高效**：
    - 直接操作 glTF 数据结构，避免不必要的转换开销。


## 使用方法

[glTF](../../../Computer%20Graphics/Mesh/glTF.md) 有一些关键概念如：Model、Scene、Node、Mesh 等。

![](../../../Computer%20Graphics/Mesh/imgs/Pasted%20image%2020240708153342.png)

#### 创建模型

模型（Model）与 glTF 文件一一对应，对应类型为 `SharpGLTF.Schema2.Model`。

1. 直接创建
```cs
var model = ModelRoot.CreateModel();
```

2. 从文件创建
```cs
var model = ModelRoot.Load("model.gltf");
```

##### 模型操作

1. 切换默认场景
```cs
// 设置默认场景
model.DefaultScene = model.LogicalScenes[0];
```





#### 创建场景

创建场景的方式有两种：
- 创建 glTF 场景实例（`SharpGLTF.Schema2.Scene`）
- 创建 glTF 场景构建器（`SharpGLTF.Scenes.SceneBuilder`）

`SceneBuilder` 最终会将其操作应用到 `SharpGLTF.Schema2.Scene` 上，`model.DefaultScene` 本质也是一个 `SharpGLTF.Schema2.Scene`，可通过 `UseScene` 获取。

构建器模式提供了更友好的 API 封装，但两者可以混合使用，选择取决于你的具体需求：简单编辑用 `UseScene`，复杂构造用 `SceneBuilder`。

##### 场景实例

1. 使用 `UseScene`
```cs
var scene = model.UseScene("default");     // 如果default存在, 使用该场景; 如果不存在, 创建一个名为default的新场景
scene.AddNode("node1");                    // 直接添加节点到 glTF 场景
```

2. `DefaultScene`
```cs
var scene = model.DefaultScene;
```


##### 场景构建器

1. 直接创建
```cs
var scene = new SceneBuilder();
```

2. 从 `SharpGLTF.Schema2.Scene` 转换
```cs
var scene = model.DefaultScene.ToSceneBuilder();       // 可能读写有问题
```


#### 创建节点

创建节点的格式有两种：
- 创建 glTF 节点 （`SharpGLTF.Schema2.Node`）
- 创建 glTF 节点构建器（`SharpGLTF.Scenes.NodeBuilder`）







## 示例代码

```cs
private static void ExportToglTF(Dictionary<string, ThBimMeshEntity> meshes, string objPath, string gltfPath)
{
	var scene = new SceneBuilder();

	foreach (var meshPair in meshes)
	{
		var meshEntity = meshPair.Value;

		// 为每个ThBimMeshEntity创建一个MeshBuilder
		var meshBuilder = new MeshBuilder<VertexPositionNormal>(meshEntity.MeshId);

		var material = new MaterialBuilder()
			.WithDoubleSide(true)
			.WithMetallicRoughnessShader()
			.WithChannelParam(KnownChannel.BaseColor, KnownProperty.RGBA, new Vector4(1, 1, 1, 1));

		var prim = meshBuilder.UsePrimitive(material);

		// 处理所有表面
		foreach (var surface in meshEntity.Surfaces)
		{
			// 处理所有三角形
			foreach (var triangle in surface.Triangles)
			{
				// 确保每个三角形有3个顶点
				if (triangle.Vertexes.Count == 3)
				{
					// 获取三个顶点的位置和法线
					var v0 = triangle.Vertexes[0];
					var v1 = triangle.Vertexes[1];
					var v2 = triangle.Vertexes[2];

					// 创建顶点
					var vertex0 = new VertexPositionNormal(
						new Vector3(v0.X, v0.Y, v0.Z),
						new Vector3(v0.Nx, v0.Ny, v0.Nz));

					var vertex1 = new VertexPositionNormal(
						new Vector3(v1.X, v1.Y, v1.Z),
						new Vector3(v1.Nx, v1.Ny, v1.Nz));

					var vertex2 = new VertexPositionNormal(
						new Vector3(v2.X, v2.Y, v2.Z),
						new Vector3(v2.Nx, v2.Ny, v2.Nz));

					// 添加三角形
					prim.AddTriangle(vertex0, vertex1, vertex2);
				}
			}
		}

		// 将网格添加到场景中
		if (meshBuilder.Primitives.Count > 0)
		{
			scene.AddRigidMesh(meshBuilder, Matrix4x4.Identity);
		}
	}

	// 保存glTF模型到文件
	scene.ToGltf2().SaveGLTF(gltfPath);
}
```