FlatBuffers 是由 Google 开发的高性能跨平台序列化库，和 [ProtoBuf](ProtoBuf.md) 类似，具有零拷贝特性，适用于游戏开发、嵌入式系统等需要高性能的场景。

## 特点

FlatBuffers 在反序列化时无需解析或解包数据，它直接在原始内存中读取，**避免了拷贝数据到新的内存空间**。这与传统的序列化库（如 Protobuf、JSON 等）相比，减少了数据的解码和映射步骤，因此反序列化非常快。通常在性能敏感的应用中，反序列化的速度是至关重要的。

FlatBuffers 在读取数据时无需将整个对象加载到内存中，它支持部分加载。这样可以处理非常大的数据集，而不需要为整个数据集预留足够的内存。


## 使用

1. **定义数据模式**

FlatBuffers 的数据结构是通过 `.fbs` 文件定义的，
```fbs
namespace MyGame;

table Person {
  name: string;
  age: int;
  position: Vec3;
}

table Vec3 {
  x: float;
  y: float;
  z: float;
}

root_type Person;
```

2. **生成代码**：使用 FlatBuffers 编译器生成指定语言代码。


3. 序列化与反序列化数据。