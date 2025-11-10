在 Unreal Engine 中，Renderer 是引擎的核心子系统，负责将场景数据转化为最终的图像输出。它管理从几何处理、光照计算到后期效果的全流程渲染工作，其设计深度融合了现代图形 API（如 DirectX 12/ Vulkan ）和硬件特性（如硬件光追、异步计算）。

## 核心架构

1. **延迟渲染管线（Deferred Shading Pipeline，`FDeferredShadingRenderer`）**：UE 默认采用延迟渲染路径，通过多阶段分离几何与光照计算：
    - **BasePass**：将场景几何信息（位置、法线、材质属性）写入 GBuffer（多张渲染目标纹理），避免重复绘制
    - **LightingPass**：基于 GBuffer 计算直接光照（含阴影）和间接光照（如 Lumen ），结果累积到 SceneColor
    - **半透明与后处理**：半透明物体由远到近混合到离屏纹理，后处理阶段应用 Bloom、抗锯齿等效果

2. **`MobileSceneRenderer`**：

3. **Render Dependency Graph（RDG）**
    - **自动化资源管理**：自动裁剪无用渲染通道（Pass）和资源（如临时纹理），优化显存使用
    - **异步计算优化**：将计算密集型任务（如体素化、光线追踪）分发到 GPU 异步队列，提升并行效率
    - **调试工具支持**：通过`RDG Insight`可视化渲染流程，或使用`vis`命令实时查看中间渲染目标（如GBuffer）


## 光照与阴影系统

1. **Lumen全局光照**
    - **动态光追与有号距离场（SDF）**：结合硬件光追与 SDF 加速，实时计算直接光（`RenderDirectLightingForLumen`）和间接光漫反射（`ApplyLumenCardAlbedo`）
    - **体素化光照缓存**：将场景光照数据烘焙到稀疏体素网格（`HeterogeneousVolumesSparseVoxelPipeline`），支持动态更新。
2. **传统光照与阴影**
    - **阴影贴图（Shadow Maps）**：为动态光源生成深度贴图，通过遮挡剔除（如`CullDistanceFieldObjectsForLight`）优化性能
    - **阴影混合策略**：区分平行光（全场景覆盖）和点光源（平截头体裁剪），减少无效计算


## 特殊效果渲染

1. **体积渲染（Heterogeneous Volumes）**
    - 支持四条管线：实时步进（`LiveShading`）、预烘焙材质（`PreShading`）、硬件光追（DXR）、蒙特卡洛路径追踪，用于烟雾、云层等动态介质
    - 通过**光线步进**（Raymarching）与**自由路径追踪**（Free-Path Tracking）模拟光散射

2. **粒子系统（Niagara Renderer）**
    - **精灵渲染器（Sprite Renderer）**：支持子UV动画、速度对齐、摄像机距离混合等特效
    - **GPU粒子**：将模拟计算卸载到GPU，通过`Simulation Stage`迭代渲染目标（RenderTarget）实现流体模拟等复杂效果

3. **动态渲染目标（Render Target）**
    - 用途包括：后处理输入、动态反射/折射纹理、小地图渲染、实时光照数据存储等
    - 示例：Niagara可将粒子输出到RenderTarget，再作为材质输入生成动态表面效果


## 渲染输出与合成

1. **分层渲染（Stencil Layers）**
    - 通过模板缓冲区分离不同对象组（如角色/背景），独立应用后期效果（如景深），支持后期合成
    - 例如：`Movie Render Queue`可输出分离的反射层、无光照层，供影视后期调色

2. **多通道渲染（Render Passes）**
    - 输出路径追踪、仅光照、对象ID通道等，用于专业合成（如Nuke中对特定对象调色）
    - 对象ID通道需禁用多重采样（`Disable Multisample Effects`）以保证边缘精度