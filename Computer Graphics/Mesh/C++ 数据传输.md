
# 通过 P/Invoke

在 C# 端解析模型并生成 Mesh 数据后，C# 和 C++ 可以通过 P/Invoke 将这些 Mesh 数据传递给 C++ 渲染引擎。

### 1. C++ 中定义 Mesh 数据结构

首先，你需要定义 C++ 和 C# 之间共享的 Mesh 数据结构，常见的 Mesh 数据可能包括以下内容：
- 顶点数组（float数组）
- 法线数组（float数组）
- UV数组（float数组）
- 索引数组（int数组）

在C++中可以定义类似以下的结构体：
```c++
struct MeshData {
    float* vertices;   // 顶点数组
    float* normals;    // 法线数组
    float* uvs;        // UV数组
    int* indices;      // 索引数组
    int vertexCount;   // 顶点数量
    int indexCount;    // 索引数量
};
```


### 2. 传递 Mesh 数据

接下来，你可以设计一个 C++ 函数，用于接收 C# 端传递过来的 Mesh 数据。为了实现从 C# 到 C++ 的数据传递，Mesh 数据中的数组可以通过指针传递。

```c++
extern "C" __declspec(dllexport) void SubmitMeshData(MeshData* mesh) {
    // 在此处将Mesh数据存储到渲染管线中
    // 例如拷贝顶点和索引数据
}
```


### 3. C# 中准备 Mesh 数据

在 C# 中，首先需要使用与 C++ 对应的结构体，将顶点、法线、UV、索引等数据准备好并转换为可以传递给 C++ 的格式。

```c#
[StructLayout(LayoutKind.Sequential)]
public struct MeshData
{
    public IntPtr vertices;    // 顶点数组
    public IntPtr normals;     // 法线数组
    public IntPtr uvs;         // UV数组
    public IntPtr indices;     // 索引数组
    public int vertexCount;    // 顶点数量
    public int indexCount;     // 索引数量
}
```

然后，C# 端生成的 Mesh 数据需要转换为非托管内存（比如使用`Marshal`分配内存），并将这些内存指针传递给 C++。

```c#
[DllImport("Renderer.dll", CallingConvention = CallingConvention.Cdecl)]
public static extern void SubmitMeshData(ref MeshData mesh);

public static void SendMeshToCpp(float[] vertices, float[] normals, float[] uvs, int[] indices)
{
    MeshData meshData = new MeshData();

    // 分配非托管内存并将数组数据复制进去
    meshData.vertices = Marshal.AllocHGlobal(vertices.Length * sizeof(float));
    Marshal.Copy(vertices, 0, meshData.vertices, vertices.Length);

    meshData.normals = Marshal.AllocHGlobal(normals.Length * sizeof(float));
    Marshal.Copy(normals, 0, meshData.normals, normals.Length);

    meshData.uvs = Marshal.AllocHGlobal(uvs.Length * sizeof(float));
    Marshal.Copy(uvs, 0, meshData.uvs, uvs.Length);

    meshData.indices = Marshal.AllocHGlobal(indices.Length * sizeof(int));
    Marshal.Copy(indices, 0, meshData.indices, indices.Length);

    meshData.vertexCount = vertices.Length / 3; // 每个顶点有3个分量
    meshData.indexCount = indices.Length;

    // 将Mesh数据传递给C++
    SubmitMeshData(ref meshData);

    // 释放非托管内存
    Marshal.FreeHGlobal(meshData.vertices);
    Marshal.FreeHGlobal(meshData.normals);
    Marshal.FreeHGlobal(meshData.uvs);
    Marshal.FreeHGlobal(meshData.indices);
}
```

### 4. 内存管理

确保 C# 端在分配了非托管内存后，传递完数据后及时释放内存，防止内存泄漏。可以在 C++ 端只存储数据的副本，避免与 C# 共享指针管理。

### 5. 优化传输性能

对于大规模 Mesh 数据，可以考虑通过批量传输或者优化内存布局来提高性能。如果数据量较大，还可以设计多线程或异步机制，以减少 C# 和 C++ 之间的传递延迟。

### 6. 扩展性

如果将来需要扩展 Mesh 数据结构（如加入骨骼动画或材质），可以通过修改 Mesh 结构体并在接口中加入新的字段，保持接口的灵活性。


# 通过 [ProtoBuf](../../../Language/Data%20Format/ProtoBuf.md) 

紧凑的二进制格式适合在渲染管线中使用，尤其是当需要传递较大或复杂的模型数据时，可以通过 .proto 文件定义数据结构，减少手动处理跨语言结构体的不便。

虽然 Protobuf 比较高效，但仍然需要额外的序列化和反序列化步骤，可能在高实时性要求下带来轻微的延迟。

# 通过 [FlatBuffers](../../../Language/Data%20Format/FlatBuffers.md) 

FlatBuffers 是由 Google 开发的高性能跨平台序列化库，和 Protobuf 类似，具有零拷贝特性。允许 C# 和 C++ 直接使用同一段内存，避免了冗余的内存拷贝操作，特别适合对性能有严格要求的场景。


# 通过共享内存

共享内存 是一种非常高效的跨进程通信（IPC）方式，C#和C++进程可以通过共享内存段直接读写相同的内存区域。


# 知名三维平台选用的技术路线

1. Unreal Engine
	- Memory Buffers（内存缓冲区）：在插件或自定义引擎模块中，Mesh 数据通常通过内存缓冲区传递，特别是在需要通过 C++ 与外部模块进行通信时。例如，顶点数据、法线、纹理坐标等都会以结构体的方式传递给渲染管线。
	- USD（Universal Scene Description）：对于更复杂的场景或跨平台实时渲染，Unreal 也集成了 USD 支持，通过 USD 格式传递包含 Mesh 数据的场景信息。

2. Unity
	- P/Invoke 与内存指针：在 C# 与 C++ 通信中，Mesh 数据常常通过 P/Invoke 或 DLL Import 的方式传递。C# 通过内存指针直接访问 C++ 的 Mesh 数据结构，如顶点缓冲区、索引缓冲区等。
	- Protobuf/FlatBuffers：一些高级项目中使用 Protobuf 或 FlatBuffers 将复杂的 Mesh 数据序列化，并在 C# 与 C++ 之间传递。

3. Blender
	- Python/C API：Blender 提供了 C 和 Python 的扩展接口，通过这些接口，开发者可以访问和传输 Blender 内部的 Mesh 数据。Python 常用于小规模的脚本化通信，而 C API 更适合大规模的高性能传输。
	- USD（Universal Scene Description）：Blender 集成了 USD 支持，特别是在需要与其他支持 USD 的平台（如 Unreal 和 Omniverse）传输复杂场景数据时非常有效。