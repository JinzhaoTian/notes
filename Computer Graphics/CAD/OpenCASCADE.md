Open CASCADE Technology（OCCT）


## 核心模块

| 模块                               | 作用                   | 关键类                                                 |
| -------------------------------- | -------------------- | --------------------------------------------------- |
| **Foundation Classes**           | 基础工具：内存管理、集合、数学      | `Handle<T>`, `gp_Pnt`, `NCollection`                |
| **Modeling Data**                | 几何与拓扑结构定义            | `Geom_Surface`, `TopoDS_Shape`, `BRep_Builder`      |
| **Modeling Algorithms**          | 布尔、倒角、缝合等建模操作        | `BRepAlgoAPI_Fuse`, `BRepFilletAPI_MakeFillet`      |
| **Visualization**                | 3D 显示与交互             | `V3d_Viewer`, `AIS_InteractiveContext`, `AIS_Shape` |
| **Data Exchange**                | 导入导出 STEP/IGES/STL 等 | `STEPControl_Reader`, `IGESControl_Writer`          |
| **Application Framework (OCAF)** | 文档管理、撤销/重做、参数化       | `TDocStd_Document`, `XCAFDoc`                       |

这些模块相互协作：用 **Modeling** 创建 `TopoDS_Shape`，交由 **Visualization** 的 `AIS_Shape` 显示，通过 **Algorithms** 修改模型，最后用 **Data Exchange** 保存为通用格式。


