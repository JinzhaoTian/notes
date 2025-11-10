
GLTF（全称：GL Transmission Format）是一种用于传输和加载3D模型的文件格式。由Khronos Group开发和推广，GLTF旨在提供一种高效的、跨平台的方式来描述3D模型及其相关的数据。

其主要包括：
- **JSON文件**：描述了模型的结构、材质、相机、动画等元数据。
- **二进制数据（.bin文件）**：包含几何数据（如顶点、法线、UV坐标）和动画数据。
- **纹理图像**：使用常见的图像格式（如PNG或JPEG）存储模型的纹理。

数据可以嵌入到 glTF 文件中，也可以从文件中引用。glTF 格式以其轻巧的重量和在应用程序之间快速传输 3D 资产的能力而闻名。此外，设计专业人员经常使用 glTF 格式在网页上嵌入交互式 3D 内容，因为它提供了快速的运行时速度和快速的上传/下载时间。

GLTF文件通常使用`.gltf`（文本格式）或`.glb`（二进制格式）作为文件扩展名。

[GLTF 编辑器 -NSDT](https://gltf.nsdt.cloud/) 是一款在线 GLB/GLTF 查看、预览工具，只需将 GLB/GLTF 拖入场景即可对 GLB/GLTF 进行预览；除此之外还对模型的材质进行编辑、修改。



## [基础结构](https://github.com/KhronosGroup/glTF-Tutorials/blob/main/gltfTutorial/README.md) 

![](imgs/Pasted%20image%2020240708153342.png)

```gltf
{
  "scene": 0,
  "scenes" : [
    {
      "nodes" : [ 0 ]
    }
  ],
  
  "nodes" : [
    {
      "mesh" : 0
    }
  ],
  
  "meshes" : [
    {
      "primitives" : [ {
        "attributes" : {
          "POSITION" : 1
        },
        "indices" : 0
      } ]
    }
  ],

  "buffers" : [
    {
      "uri" : "data:application/octet-stream;base64,AAABAAIAAAAAAAAAAAAAAAAAAAAAAIA/AAAAAAAAAAAAAAAAAACAPwAAAAA=",
      "byteLength" : 44
    }
  ],
  "bufferViews" : [
    {
      "buffer" : 0,
      "byteOffset" : 0,
      "byteLength" : 6,
      "target" : 34963
    },
    {
      "buffer" : 0,
      "byteOffset" : 8,
      "byteLength" : 36,
      "target" : 34962
    }
  ],
  "accessors" : [
    {
      "bufferView" : 0,
      "byteOffset" : 0,
      "componentType" : 5123,
      "count" : 3,
      "type" : "SCALAR",
      "max" : [ 2 ],
      "min" : [ 0 ]
    },
    {
      "bufferView" : 1,
      "byteOffset" : 0,
      "componentType" : 5126,
      "count" : 3,
      "type" : "VEC3",
      "max" : [ 1.0, 1.0, 0.0 ],
      "min" : [ 0.0, 0.0, 0.0 ]
    }
  ],
  
  "asset" : {
    "version" : "2.0"
  }
}
```

### scenes

scene 是一个对象数组，定义这个 gltf 中有多少场景，每个场景是一个对象。

### nodes

![](imgs/Pasted%20image%2020240709135723.png)

nodes 是一个对象数组，包含所有的节点数据，是构成 Scene Graph 的核心数据。

```
Structure:           local transform      global transform
root                 R                    R
 +- nodeA            A                    R*A
     +- nodeB        B                    R*A*B
     +- nodeC        C                    R*A*C
```


### meshes

meshes 是一个对象数组，mesh 对象实际的集合数据由 primitive 对象的 attributes 对象数据和 indices 对象数组通过引用 accessor 对象给出。


![](imgs/Pasted%20image%2020250701165102.png)

### materials




![](imgs/Pasted%20image%2020250701165150.png)


### textures






### accessors

用来描述并索引 bufferView 对象，如顶点位置数据，顶点索引数据，法向数据等。


### bufferView

一个 bufferView 对象引用了一个 buffer 对象的一部分数据。



### extensions



## 使用

### 记录边信息

glTF 虽然本身没有专门设计用于存储边信息的字段，但可以通过以下几种方法实现边信息的传输：

1. **使用顶点颜色或自定义属性**
```json
{
  "meshes": [
    {
      "primitives": [
        {
          "attributes": {
            "POSITION": 0,
            "EDGE_FLAG": 1
          }
        }
      ]
    }
  ],
  "accessors": [
    {
      "bufferView": 0,
      "componentType": 5126,
      "count": 36,
      "type": "VEC3"
    },
    {
      "bufferView": 1,
      "componentType": 5126,
      "count": 36,
      "type": "SCALAR",
      "max": [1.0],
      "min": [0.0]
    }
  ]
}
```

2. **使用扩展(Extensions)**：创建自定义扩展来存储边信息
```json
{
    "extensionsUsed": ["EDGE_info"],
    "extensions": {
        "EDGE_info": {
            "edgeIndices": [0, 1, 1, 2, 2, 0, ...],
            "edgeProperties": {
                "sharpness": [0.5, 1.0, 0.8, ...],
                "visibility": [true, false, true, ...]
            }
        }
    }
}
```

3. **使用变形目标(Morph Targets)**：将边信息编码为额外的变形目标
```json
{
    "meshes": [
        {
            "primitives": [
                {
                    "targets": [
                        {
                            "EDGE_FLAG": 2
                        }
                    ]
                }
            ]
        }
    ]
}
```


4. **使用额外纹理**：将边信息编码到纹理中，通过 UV 映射访问
```json
{
  "materials": [
    {
      "extensions": {
        "EDGE_texture": {
          "edgeMapTexture": {
            "index": 2
          }
        }
      }
    }
  ],
  "textures": [
    {
      "source": 1,
      "sampler": 0
    }
  ]
}
```







## 相关项目


[CesiumGS/obj2gltf: Convert OBJ assets to glTF (github.com)](https://github.com/CesiumGS/obj2gltf)


- [jonlo/ifcConverter-web (github.com)](https://github.com/jonlo/ifcConverter-web) ：提供了一个前后端的web示例，其中涉及到IFC转化成gltf是通过调用IfcCovert.exe完成的。
